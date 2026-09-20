---
name: gnomad-variant-pass-filtering
description: Use when a workflow needs a cleaned gnomAD SNV set (e.g. gnomAD r2.0.2
  VCF) restricted to PASS-filtered synonymous, nonsense, and missense variants as
  input for domain-level or meta-domain variant aggregation.
when_to_use_negative:
- Input is not a gnomAD-style sites VCF (e.g. already-filtered variant tables, ClinVar-only
  VCFs, or VCFs where consequence annotation is absent), since the PASS/consequence
  filters cannot be evaluated
- Analysis requires all variant classes (indels, MNPs, non-PASS variants) or aims
  to characterize filtering artifacts rather than true SNV variation, since PASS-only
  SNV selection discards those records by design
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_2533
tools:
- name: MetaDome
  role: Consumes the PASS-filtered gnomAD SNV set at runtime and projects variants
    onto Pfam domain consensus positions in the PostgreSQL mapping database
  repo: https://github.com/cmbi/metadome
- name: Python
  role: MetaDome implementation language (Python v3.5.1) used for VCF filtering and
    downstream processing logic
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Relational database (v10) storing the genomic-to-protein mapping into which
    the filtered variants are intersected
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:4c9c7b6b6ee080c56a6e783e5d99e891693f9c1a80c5654e6f6a99bafb3b52e6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/laurensvdwiel/metadome
  repo_url: https://github.com/cmbi/metadome
  tools:
  - MetaDome
  - PostgreSQL
  - Python
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_2533
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# gnomad-variant-pass-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters a raw gnomAD VCF down to the variant set MetaDome uses for tolerance analysis: all synonymous, nonsense, and missense SNVs that meet the PASS filter criteria. Applying a defensible quality and consequence filter is a prerequisite for projecting population variation onto Pfam domain consensus positions.

## When to use

Use when a workflow needs a cleaned gnomAD SNV set (e.g. gnomAD r2.0.2 VCF) restricted to PASS-filtered synonymous, nonsense, and missense variants as input for domain-level or meta-domain variant aggregation.

## When NOT to use

- Input is not a gnomAD-style sites VCF (e.g. already-filtered variant tables, ClinVar-only VCFs, or VCFs where consequence annotation is absent), since the PASS/consequence filters cannot be evaluated
- Analysis requires all variant classes (indels, MNPs, non-PASS variants) or aims to characterize filtering artifacts rather than true SNV variation, since PASS-only SNV selection discards those records by design

## Inputs

- gnomAD sites VCF (gzipped, with .tbi index), e.g. gnomad.exomes.V2XXX.sites.vcf.gz (GRCh37) or gnomad.joint.v4.X.sites.exomes.vcf.gz (GRCh38)
- Genome build designation (GRCh37 or GRCh38) matching the mapping database

## Outputs

- Filtered set of PASS-only synonymous, nonsense, and missense SNVs (12,164,292 variants for the MetaDome gnomAD r2.0.2 configuration)
- Variant set ready for genomic-position-to-protein-residue mapping via the MetaDome PostgreSQL mapping database

## How to apply

Load the gnomAD sites VCF (e.g. gnomad.exomes.V2XXX.sites.vcf.gz with its .tbi index) and retain only SNVs — not indels or MNPs — whose consequence annotation is synonymous, nonsense, or missense. Apply the VCF PASS filter: variants meeting the PASS criteria are considered true variants and are the only ones counted; drop all others. In the MetaDome paper this step yields 12,164,292 gnomAD variants, which is the expected total to validate against. Filenames and release selection are configured in metadome/default_settings.py, and MetaDome was tested with gnomAD v2 for GRCh37 and v4.1 (joint exomes) for GRCh38. The filtered set is then fed into the PostgreSQL mapping database to translate genomic positions to UniProt residues and Pfam domain consensus positions.

## Related tools

- **MetaDome** (Consumes the PASS-filtered gnomAD SNV set at runtime and projects variants onto Pfam domain consensus positions in the PostgreSQL mapping database) — https://github.com/cmbi/metadome
- **Python** (MetaDome implementation language (Python v3.5.1) used for VCF filtering and downstream processing logic) — https://github.com/cmbi/metadome
- **PostgreSQL** (Relational database (v10) storing the genomic-to-protein mapping into which the filtered variants are intersected) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz  # filtered gnomAD SNVs are then read from $GNOMAD_DIR per default_settings.py
```

## Evaluation signals

- Variant count after filtering matches the published total of 12,164,292 gnomAD variants for the gnomAD r2.0.2 / synonymous-nonsense-missense SNV configuration
- Every retained record has FILTER=PASS and is an SNV with consequence annotated as synonymous, nonsense, or missense
- Filenames in default_settings.py resolve to existing VCF and .tbi files under $GNOMAD_DIR/<build>/ for both configured genome builds
- Downstream mapping query returns Pfam domain consensus positions for the expected fraction of filtered variants (consistent with the ~71-72% Pfam coverage reported for pathogenic missense variants)

## Limitations

- Filtering is release-dependent: only gnomAD v2 (GRCh37) and v4.1 joint exomes (GRCh38) were tested; later versions 'may work as well as they come available' but are unverified
- GENCODE, UniProt and Pfam are not read by the application at runtime — they are prebuild-pipeline inputs — so a correctly filtered VCF alone is insufficient without a populated mapping database
- Aggregating variation via meta-domains loses specific context such as haplotype information or protein-protein interactions, so the filtered set supports domain-level tolerance but not variant-context analysis

## Evidence

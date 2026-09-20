---
name: population-vs-pathogenic-variant-enrichment-analysis
description: Use when you need to quantify how many disease-causing missense variants
  fall within annotated Pfam protein domains versus outside them, or to project gnomAD/ClinVar
  SNVs onto domain consensus positions for tolerance comparison.
when_to_use_negative:
- Variants are non-SNV (indels, CNVs) or fail the PASS filter — the pipeline retains
  only PASS synonymous, nonsense, and missense SNVs
- The variant's genomic position has no entry in the mapping database (transcript
  not linked to a Swiss-Prot protein), so no domain projection is possible
- The goal is gene-level tolerance landscapes rather than domain-level enrichment
  of pathogenic versus population variation
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: MetaDome
  role: Provides the mapping database and pipeline that project variants onto Pfam
    domain consensus positions and aggregate homologous domain variation
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the genomic-to-protein mapping and domain annotations queried to translate
    SNV genomic positions to UniProt residues
- name: Python 3.5.1
  role: Implementation language for the MetaDome batch loading and variant projection
    logic
  repo: https://github.com/cmbi/metadome
- name: BLAST
  role: Protein–Protein BLAST aligns GENCODE Basic translations to Swiss-Prot sequences
    to build the mapping database used in the projection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:b9e0b74e135aab6dcd0fcb7d1014b3aec965559dece775ca338f09e79283d823
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
  - BLAST
  - MetaDome
  - PostgreSQL
  - PostgreSQL 10
  - Python
  - Python 3.5.1
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Population vs pathogenic variant enrichment analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps filtered gnomAD population SNVs and ClinVar/HGMD pathogenic missense variants onto Pfam domain consensus positions via the MetaDome PostgreSQL mapping database, then computes the fraction of pathogenic missense variants inside versus outside Pfam domains (~71–72% coverage). This establishes whether pathogenic variation is enriched within protein domains relative to population variation.

## When to use

Use when you need to quantify how many disease-causing missense variants fall within annotated Pfam protein domains versus outside them, or to project gnomAD/ClinVar SNVs onto domain consensus positions for tolerance comparison.

## When NOT to use

- Variants are non-SNV (indels, CNVs) or fail the PASS filter — the pipeline retains only PASS synonymous, nonsense, and missense SNVs
- The variant's genomic position has no entry in the mapping database (transcript not linked to a Swiss-Prot protein), so no domain projection is possible
- The goal is gene-level tolerance landscapes rather than domain-level enrichment of pathogenic versus population variation

## Inputs

- gnomAD sites VCF (e.g. gnomAD r2.0.2 / v2 exomes or v4.1 joint exomes, .vcf.gz with .tbi)
- ClinVar VCF (e.g. clinvar_YYYYMMDD.vcf.gz with .tbi) with pathogenicity annotations
- MetaDome prebuild mapping database CSV (genomic position → transcript → UniProt residue → Pfam consensus position) loaded into PostgreSQL
- Pfam-A domain annotations (interpro_domains table, from InterProScan)

## Outputs

- Per-dataset fraction of pathogenic missense variants located inside versus outside a Pfam domain (expected ~71–72% inside)
- Per-variant assignments of Pfam domain and consensus position
- Counts of variants tabulated per Pfam domain family

## How to apply

Load the gnomAD VCF and retain only synonymous, nonsense, and missense SNVs meeting the PASS filter criteria (12,164,292 variants expected); separately load a ClinVar VCF and retain SNVs annotated as Pathogenic (34,076 expected). Query the PostgreSQL mapping database — populated from GENCODE/BLAST/Swiss-Prot alignments — to translate each SNV's genomic position to its UniProt protein residue via the 'mappings' table. Intersect mapped residues with Pfam-A domain annotations (from InterProScan) to assign each variant a Pfam domain and consensus position. Compute, per dataset, the fraction of pathogenic missense variants inside a Pfam domain versus outside, and tabulate counts per domain family; the reference result is ~71–72% of pathogenic missense variants within Pfam domains.

## Related tools

- **MetaDome** (Provides the mapping database and pipeline that project variants onto Pfam domain consensus positions and aggregate homologous domain variation) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the genomic-to-protein mapping and domain annotations queried to translate SNV genomic positions to UniProt residues)
- **Python 3.5.1** (Implementation language for the MetaDome batch loading and variant projection logic) — https://github.com/cmbi/metadome
- **BLAST** (Protein–Protein BLAST aligns GENCODE Basic translations to Swiss-Prot sequences to build the mapping database used in the projection)

## Examples

```
./populate_database.sh --data-dir /path/to/csv_dir --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- Recomputed gnomAD variant count after PASS + synonymous/nonsense/missense filtering equals 12,164,292
- Recomputed ClinVar pathogenic variant count equals 34,076
- Number of Pfam domains linked to Swiss-Prot sequences equals 71,419 across 30,406 Swiss-Prot sequences
- Fraction of pathogenic missense variants inside a Pfam domain falls in the 71–72% range
- Mapping database contains 32,595,355 unique genomic positions linked to 19,226,961 Swiss-Prot residues

## Limitations

- Aggregating variation via meta-domains loses specific context such as haplotype information or protein-protein interactions
- Aggregation via meta-domains only encapsulates general biological or molecular functions attributed to the domain
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; unlinked transcripts (e.g. grayed in the web view) are excluded
- The mapping database holds tens of millions of rows per genome build, so database loading is long and needs disk headroom

## Evidence

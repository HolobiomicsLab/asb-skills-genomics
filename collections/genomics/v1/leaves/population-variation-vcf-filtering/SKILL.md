---
name: population-variation-vcf-filtering
description: Use when building gene-wise dN/dS or missense/synonymous tolerance inputs
  from a raw population VCF (e.g. gnomAD r2.0.2 sites VCF) and you need only confident,
  protein-coding SNVs.
when_to_use_negative:
- The downstream analysis needs indels, splice variants, or low-quality variants —
  this skill deliberately retains only PASS SNVs of three protein-coding classes
- Variants cannot be mapped to a Swiss-Prot protein via the MetaDome mapping database
  (transcripts not mapped to Swiss-Prot cannot be used in the analysis)
- You need genome-wide summary statistics rather than per-transcript tolerance computation
  — raw VCF filtering alone does not produce tolerance scores
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0622
tools:
- name: MetaDome
  role: Application whose variation-loading and filtering pipeline consumes the PASS-filtered
    gnomAD and ClinVar SNVs for tolerance landscape computation
  repo: https://github.com/cmbi/metadome
- name: Python 3.5.1
  role: Language used (with pandas/PyVCF per the workflow) to parse the gnomAD VCF
    and retain PASS synonymous/nonsense/missense SNVs
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL 10
  role: Stores the mapping database used to translate filtered genomic variant positions
    into Swiss-Prot protein residues
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:6071c70f85eef55aa2b73a59dda0a47de3a4e19adfa301630d2218e06be0660e
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
  - PostgreSQL 10
  - Python
  - Python 3.5.1
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# population-variation-vcf-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters large population-genetics VCF releases (gnomAD exomes/genomes, ClinVar) down to the variant classes MetaDome's tolerance statistics consume: synonymous, nonsense, and missense SNVs meeting the PASS filter, plus pathogenic SNVs from ClinVar.

## When to use

Use when building gene-wise dN/dS or missense/synonymous tolerance inputs from a raw population VCF (e.g. gnomAD r2.0.2 sites VCF) and you need only confident, protein-coding SNVs.

## When NOT to use

- The downstream analysis needs indels, splice variants, or low-quality variants — this skill deliberately retains only PASS SNVs of three protein-coding classes
- Variants cannot be mapped to a Swiss-Prot protein via the MetaDome mapping database (transcripts not mapped to Swiss-Prot cannot be used in the analysis)
- You need genome-wide summary statistics rather than per-transcript tolerance computation — raw VCF filtering alone does not produce tolerance scores

## Inputs

- gnomAD sites VCF (e.g. gnomad.exomes.V2XXX.sites.vcf.gz for GRCh37, gnomad.joint.v4.X.sites.exomes.vcf.gz for GRCh38) with .tbi index
- ClinVar release VCF (clinvar_YYYYMMDD.vcf.gz) with .tbi index
- PostgreSQL mapping database linking genomic positions to Swiss-Prot protein residues

## Outputs

- Filtered set of PASS synonymous, nonsense, and missense SNVs per genome build
- Set of pathogenic SNVs from the ClinVar release VCF
- Per-residue missense and synonymous counts feeding the sliding-window dN/dS tolerance landscape

## How to apply

Load the gnomAD VCF (e.g. gnomad.exomes.V2XXX.sites.vcf.gz) with pandas/PyVCF and retain only SNVs annotated as synonymous, nonsense, or missense that meet the PASS filter criteria; variants meeting the PASS criteria are treated as true variants, so everything failing site-level filters is discarded. In parallel, extract pathogenic SNVs from the ClinVar release VCF (clinvar_YYYYMMDD.vcf.gz) for the same genome build (GRCh37 or GRCh38). Map each retained variant's genomic position to protein residues via the PostgreSQL mapping database before computing per-residue missense and synonymous counts. Filenames of the VCFs to read are set in metadome/default_settings.py, so configure the release-specific file names there when switching gnomAD or ClinVar versions.

## Related tools

- **MetaDome** (Application whose variation-loading and filtering pipeline consumes the PASS-filtered gnomAD and ClinVar SNVs for tolerance landscape computation) — https://github.com/cmbi/metadome
- **Python 3.5.1** (Language used (with pandas/PyVCF per the workflow) to parse the gnomAD VCF and retain PASS synonymous/nonsense/missense SNVs) — https://github.com/cmbi/metadome
- **PostgreSQL 10** (Stores the mapping database used to translate filtered genomic variant positions into Swiss-Prot protein residues)

## Examples

```
Filenames are set in metadome/default_settings.py; change them there to use other releases. Only variants passing gnomAD's `PASS` filter are counted.
```

## Evaluation signals

- Every retained variant carries the PASS filter flag in the gnomAD VCF FILTER column; no filtered-in-place or non-SNV records remain
- Retained records are restricted to the three sequence-ontology classes: synonymous, nonsense, and missense SNVs
- ClinVar variants retained are pathogenic SNVs from the versioned release VCF (tested with version 20251006 for GRCh37 and GRCh38)
- Downstream tolerance track must reproduce codon for codon against the released prebuild-visualizations quality-control report for the corresponding genome build
- Filtered positions resolve to residues in the mapping database; the resulting mapping links 32,595,355 unique genomic positions to Swiss-Prot residues in the published build

## Limitations

- Only variants passing gnomAD's PASS filter are counted, so site-quality-flagged true variation is excluded by design
- Version coupling: MetaDome was tested with gnomAD v2 (GRCh37) and v4.1 joint exomes (GRCh38) and ClinVar 20251006; later versions 'may work as well' but are unverified and filenames must be updated in default_settings.py
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; variants on unmapped transcripts (displayed in gray) are unusable

## Evidence

---
name: dnds-sliding-window-tolerance-landscape-computation
description: Use when you need a per-residue genetic tolerance profile for a protein-coding
  transcript from population variation data, e.g. to locate regions depleted of missense
  variation before variant interpretation.
when_to_use_negative:
- The transcript of interest is not mapped to a Swiss-Prot protein — only transcripts
  mapped to a Swiss-Prot protein can be used in the analysis
- Variants are not single-nucleotide (indels/structural variants) — the landscape
  is computed only from synonymous, nonsense, and missense SNVs
- You need domain-level cross-gene tolerance rather than a within-gene landscape —
  use the meta-domain aggregation workflow instead
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: MetaDome
  role: Server/Python package that computes the tolerance landscape on demand or via
    prebuilding
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Stores the mapping database translating variant genomic positions into protein
    residues of the linked Swiss-Prot entry
- name: Python
  role: Implementation language (v3.5.1 in original paper) for loading VCFs, counting
    variants, and computing the sliding-window dN/dS
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:0e9b5def157ced827a6d526dc02eabaf9173cc4050d8f8db41a84634d45eb793
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# dN/dS sliding-window tolerance landscape computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes MetaDome's gene-wide tolerance landscape as a missense-over-synonymous (dN/dS) ratio over a sliding window of 21 residues (10 before and 10 after each residue of interest) across the full protein, using gnomAD SNVs with codon-table background correction.

## When to use

Use when you need a per-residue genetic tolerance profile for a protein-coding transcript from population variation data, e.g. to locate regions depleted of missense variation before variant interpretation.

## When NOT to use

- The transcript of interest is not mapped to a Swiss-Prot protein — only transcripts mapped to a Swiss-Prot protein can be used in the analysis
- Variants are not single-nucleotide (indels/structural variants) — the landscape is computed only from synonymous, nonsense, and missense SNVs
- You need domain-level cross-gene tolerance rather than a within-gene landscape — use the meta-domain aggregation workflow instead

## Inputs

- gnomAD sites VCF (V2 for GRCh37, v4.1 joint exomes for GRCh38) with .tbi index
- ClinVar release VCF (version 20251006 tested) with .tbi index
- MetaDome prebuild mapping database CSV (PostgreSQL-loaded) linking genomic positions to Swiss-Prot residues
- Swiss-Prot protein sequence linked to the GENCODE Basic transcript

## Outputs

- tolerance_landscape.csv with one row per protein position (per-position dN/dS score)
- Tolerance track annotation marking window start/end positions as noisy

## How to apply

Load gnomAD VCF (e.g. gnomad.exomes V2 for GRCh37 or gnomad.joint v4.1 exomes for GRCh38) and retain only synonymous, nonsense, and missense SNVs meeting the PASS filter, plus pathogenic SNVs from the ClinVar release VCF (e.g. version 20251006). Query the PostgreSQL mapping database (loaded from the Zenodo prebuild-mapping-database CSV via metadome.batch_load) to translate each variant's genomic position into the corresponding Swiss-Prot residue. Per residue, count observed missense and synonymous variants and normalize each by the number of possible missense and synonymous substitutions from the codon table (background correction). Compute the dN/dS score as (missense_obs/missense_bg)/(synonymous_obs/synonymous_bg) over a sliding window of size 21 (10 residues before and 10 after the residue of interest) across the entire protein; mark window edges (start/end) as noisy since they are not averaged over a full-length window. Emit one row per protein position as tolerance_landscape.csv, and validate that the track reproduces codon for codon against the released prebuild-visualizations quality-control report for the corresponding genome build.

## Related tools

- **MetaDome** (Server/Python package that computes the tolerance landscape on demand or via prebuilding) — https://github.com/cmbi/metadome
- **PostgreSQL** (Stores the mapping database translating variant genomic positions into protein residues of the linked Swiss-Prot entry)
- **Python** (Implementation language (v3.5.1 in original paper) for loading VCFs, counting variants, and computing the sliding-window dN/dS)

## Examples

```
./prebuild_all_after_populate_database.sh --workers 8 --genome-build GRCh38
```

## Evaluation signals

- The tolerance track must reproduce codon for codon against the released prebuild-visualizations quality-control report (quality_control_report_GRCh37.p13.md / GRCh38.p14.md) for the corresponding genome build
- Every reported position must have a full 21-residue window; start/end positions are explicitly flagged as noisy rather than silently truncated
- Only variants passing gnomAD's PASS filter are counted — verify filtered-out variants do not contribute to counts
- Per-residue observed counts are normalized by codon-table-derived possible missense/synonymous substitution counts before ratioing (background correction applied)
- Prebuilt landscapes match on-demand recomputation from the same mapping database and variant sources

## Limitations

- Window edges (protein start/end) are noisy because they are not averaged over a full-length 21-residue window
- Only transcripts mapped to a Swiss-Prot protein can be used in the analysis; unmapped transcripts are displayed in gray
- The first on-demand request for a transcript without prebuilding can take minutes; prebuilding is optional but recommended

## Evidence

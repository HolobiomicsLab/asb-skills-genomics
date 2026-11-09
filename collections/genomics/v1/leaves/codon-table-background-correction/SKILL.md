---
name: codon-table-background-correction
description: Use when computing per-residue missense or synonymous variation from
  a VCF where raw counts would be biased by codon degeneracy; required before calculating
  dN/dS tolerance scores over a gene.
when_to_use_negative:
- When computing meta-domain aggregated tolerance across homologous Pfam domains —
  that analysis projects variation onto Pfam consensus positions rather than using
  per-residue dN/dS with codon correction
- When variant counts per residue are already normalized or the analysis does not
  distinguish missense from synonymous variation (e.g., purely pathogenic-variant
  annotation via ClinVar)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0625
tools:
- name: MetaDome
  role: Web server and Python application that computes the tolerance landscape with
    codon-table background correction and sliding-window dN/dS
  repo: https://github.com/cmbi/metadome
- name: PostgreSQL
  role: Stores the mapping database that translates each variant's genomic position
    into the corresponding Swiss-Prot protein residue before correction
  repo: https://github.com/cmbi/metadome
- name: Python
  role: Implementation language (v3.5.1) in which the correction and dN/dS computation
    are implemented
  repo: https://github.com/cmbi/metadome
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/humu.23798
    title: 'MetaDome: Pathogenicity analysis of genetic variants through aggregation
      of homologous human protein domains'
  source_content_hash: sha256:f6c265bf0d4231e1132cbb42e08f266ab3e1909641e8ea9ce76b99bdccb5476c
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
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# codon-table-background-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes observed variant counts per residue by the number of possible amino-acid substitutions derivable from each residue's codon, before computing the missense/synonymous (dN/dS) tolerance ratio used in MetaDome's gene-wide tolerance landscape.

## When to use

Use when computing per-residue missense or synonymous variation from a VCF where raw counts would be biased by codon degeneracy; required before calculating dN/dS tolerance scores over a gene.

## When NOT to use

- When computing meta-domain aggregated tolerance across homologous Pfam domains — that analysis projects variation onto Pfam consensus positions rather than using per-residue dN/dS with codon correction
- When variant counts per residue are already normalized or the analysis does not distinguish missense from synonymous variation (e.g., purely pathogenic-variant annotation via ClinVar)

## Inputs

- PASS-filtered gnomAD SNVs (synonymous, nonsense, missense) from gnomAD r2.0.2 VCF (or v2/v4.1 per genome build)
- Genomic-to-protein-position mappings from the MetaDome PostgreSQL mapping database
- Swiss-Prot protein sequence linked to the GENCODE Basic transcript

## Outputs

- Per-residue corrected missense and synonymous ratios
- Per-position dN/dS tolerance scores (tolerance_landscape.csv, one row per protein position)

## How to apply

For each protein residue, count observed missense and synonymous variants (from PASS-filtered gnomAD SNVs mapped via the PostgreSQL mapping database to Swiss-Prot residues), then divide each count by the number of possible missense and synonymous substitutions obtainable from that residue's codon using the standard codon table. This background correction removes the intrinsic codon-degeneracy bias (e.g., leucine has six codons and more possible missense changes than tryptophan's single codon). The corrected missense ratio divided by the corrected synonymous ratio yields the per-position dN/dS score, which MetaDome then averages over a sliding window of size 21 (10 residues before and 10 after each residue of interest) across the full protein. Emit per-position scores as tolerance_landscape.csv, marking window edges as noisy since they are not averaged over a full-length window.

## Related tools

- **MetaDome** (Web server and Python application that computes the tolerance landscape with codon-table background correction and sliding-window dN/dS) — https://github.com/cmbi/metadome
- **PostgreSQL** (Stores the mapping database that translates each variant's genomic position into the corresponding Swiss-Prot protein residue before correction) — https://github.com/cmbi/metadome
- **Python** (Implementation language (v3.5.1) in which the correction and dN/dS computation are implemented) — https://github.com/cmbi/metadome

## Examples

```
./populate_database.sh --data-dir /path/to/csv --file MetaDome_v2.0_GRCh38.p14_GENCODE-v45_UniProt-2025-01_Pfam-37.4_prebuild-mapping-database.csv.gz
```

## Evaluation signals

- The tolerance track must reproduce codon for codon against the released prebuild-visualizations quality-control report for the corresponding genome build (GRCh37.p13 or GRCh38.p14)
- Corrected ratios must be bounded: observed missense/synonymous counts divided by codon-table substitution counts cannot exceed 1 per residue
- Window-edge positions (first and last 10 residues of the protein) must be flagged as noisy because they lack a full-length 21-residue window
- Only variants meeting the PASS filter criteria are included in the observed counts; filtered-out variants must not alter the ratios
- Non-mapped transcripts (not linked to a Swiss-Prot protein) must be excluded from landscape computation and displayed in gray

## Limitations

- Window edges of the tolerance landscape are noisy because they are not averaged over a full 21-residue window
- Only synonymous, nonsense, and missense SNVs meeting the PASS filter are used; other variant classes and indels are excluded
- Only transcripts mappable to a Swiss-Prot protein can be analyzed (42,116 of 56,319 GENCODE Basic transcripts in the published build)
- Meta-domain aggregation loses specific context such as haplotype information and protein-protein interactions, and only encapsulates general biological or molecular functions attributed to the domain

## Evidence

- [other] Per residue, count observed missense and synonymous variants and divide each by the number of possible missense and synonymous substitutions from the codon table (background correction).: [evidence span withheld — anchor status: not_found]
- [methods] Sliding window dN/dS: "The tolerance landscape computes this ratio as a sliding window of size 21 (i.e., 10 residues before and 10 after the residue of interest)"
- [methods] PASS filter: "selecting all synonymous, nonsense, and missense variants that meet the PASS filter criteria. Variants meeting the PASS criteria are considered to be true variants"
- [readme] QC reproduction: "confirms the tolerance track reproduces from it codon for codon, and compares the release against MetaDome v1.0.1 (2022)"

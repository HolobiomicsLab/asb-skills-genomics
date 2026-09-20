---
name: rna-editing-fraction-per-cluster-aggregation
description: Use when a long-read single-cell dataset (UMI consensus BAM from ScNaUmi-seq/Sicelore)
  must be interrogated for RNA editing site usage across cell populations or maturation
  states, quantified as edited/total molecules per cluster group.
when_to_use_negative:
- The input is raw Nanopore reads or an uncorrected read BAM rather than UMI consensus
  molecules — per-base errors would be indistinguishable from true editing events.
- Cells lack a cluster assignment (e.g., no t-SNE cluster groups), so no per-cluster
  aggregation can be performed.
- The goal is genome-wide SNV calling with default molecule filters rather than retaining
  all molecules (MINRN=0, MINQV=0) for a targeted editing-site quantification.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Sicelore (SNPMatrix pipeline)
  role: Generates the [Cell] x [SNP position - base] molecule-count matrix and per-molecule
    SNP info used to compute editing fractions
  repo: https://github.com/ucagenomix/sicelore-2.1
- name: samtools
  role: Required dependency of the Sicelore workflow for BAM manipulation of the molecule
    BAM
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:4de9b328fe9cb41ba433c0f7b35e1dfba454c83f8ff4d1f4b73e89eb085477ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore-2.1
  tools:
  - Java
  - Sicelore (SNPMatrix pipeline)
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# rna-editing-fraction-per-cluster-aggregation

## Summary

Computes per-cell-cluster A-to-I RNA editing fractions (e.g., the Gria2 R/G site) from Sicelore SNPMatrix molecule-count matrices, revealing editing increases during neuronal maturation in E18 mouse brain scRNA-seq.

## When to use

Use when a long-read single-cell dataset (UMI consensus BAM from ScNaUmi-seq/Sicelore) must be interrogated for RNA editing site usage across cell populations or maturation states, quantified as edited/total molecules per cluster group.

## When NOT to use

- The input is raw Nanopore reads or an uncorrected read BAM rather than UMI consensus molecules — per-base errors would be indistinguishable from true editing events.
- Cells lack a cluster assignment (e.g., no t-SNE cluster groups), so no per-cluster aggregation can be performed.
- The goal is genome-wide SNV calling with default molecule filters rather than retaining all molecules (MINRN=0, MINQV=0) for a targeted editing-site quantification.

## Inputs

- UMI consensus molecule BAM (molecules.GE.tags.bam produced by Sicelore step 4.b.5)
- SNP descriptor file listing the editing positions (e.g., Gria2 R/G site)
- ValidBarcodes.csv (valid cell barcode list)
- Sicelore PREFIX_matrix.txt [Cell] x [SNP position - base] molecule-count matrix
- Sicelore PREFIX_molinfos.txt per-molecule SNP information
- Cell barcode to t-SNE cluster assignment (cluster groups 1,2,5 / 3,6 / 4,7)

## Outputs

- editing_fraction_per_cluster.tsv (editing fraction per cluster group per editing position)
- Per-group edited vs. total molecule counts covering each SNP position

## How to apply

Run the Sicelore SNPMatrix pipeline (Step 5, Single Nucleotide Variant calling cell by cell) on the UMI consensus molecule BAM (molecules.GE.tags.bam from step 4.b.5) with the SNP descriptor file, the valid barcode list (ValidBarcodes.csv), setting MINRN=0 and MINQV=0 so all molecules are retained rather than filtered. Parse the PREFIX_matrix.txt [Cell] x [SNP position - base] molecule-count matrix together with PREFIX_molinfos.txt for per-molecule SNP information. Map each cell barcode to its t-SNE cluster group (progenitor clusters 1,2,5; immature clusters 3,6; mature neurons 4,7) and sum edited-base versus total molecule counts per position within each group. Compute the editing fraction as edited molecules divided by total molecules covering the editing position per cluster group, and compare against the reported maturation trend (9.3% progenitor, 14.3% immature GABAergic, 16.4% immature glutamatergic, 26.7% mature GABAergic, 26% mature glutamatergic). The rationale is that UMI-guided error correction produces consensus molecules accurate enough to distinguish true A-to-G mismatches from Nanopore sequencing errors.

## Related tools

- **Sicelore (SNPMatrix pipeline)** (Generates the [Cell] x [SNP position - base] molecule-count matrix and per-molecule SNP info used to compute editing fractions) — https://github.com/ucagenomix/sicelore-2.1
- **samtools** (Required dependency of the Sicelore workflow for BAM manipulation of the molecule BAM) — https://github.com/samtools/samtools

## Examples

```
java -jar Sicelore.jar -method 5 -b molecules.GE.tags.bam -f genome.fa -s snpDescriptor.txt -cb ValidBarcodes.csv -o PREFIX -minrn 0 -minqv 0
```

## Evaluation signals

- Recomputed editing fractions reproduce the reported maturation gradient: 9.3% (progenitors 1,2,5), 14.3% (immature GABAergic 6), 16.4% (immature glutamatergic 3), 26.7% (mature GABAergic 7), 26% (mature glutamatergic 4).
- Editing fraction per cluster equals edited molecules divided by total molecules covering the R/G position — verify denominator counts only molecules with coverage at the SNP position.
- Matrix dimensions of PREFIX_matrix.txt match the number of valid cell barcodes (from ValidBarcodes.csv) and SNP positions in the descriptor file.
- Every cell barcode in the aggregation maps to exactly one cluster group; no unassigned or duplicated barcodes remain.
- Per-position edited + unedited base counts are consistent between PREFIX_matrix.txt and PREFIX_molinfos.txt molecule tallies.

## Limitations

- MINRN and MINQV are set to 0 to retain all molecules, which sacrifices the quality filtering normally applied for SNP calling; low-support positions may be noisy.
- Editing fractions are measured only at SNP positions listed in the descriptor file; discovery of novel editing sites is out of scope.
- Aggregation depends on a prior cell-to-cluster mapping; errors in barcode-to-cluster assignment propagate directly into the per-group fractions.

## Evidence

- [other] Sicelore SNPMatrix on molecules.GE.tags.bam with MINRN=0 and MINQV=0: "MINRN (required): Minimum read number to keep the molecule for SNP calling"
- [abstract] ScNaUmi-seq enables defining SNVs/RNA editing at single cell level: "ScNaUmi-seq allows defining splicing and SNVs (RNA editing) at a single cell level"
- [readme] Sicelore step 5 is single nucleotide variant calling cell by cell: "Step 5 (optional) - [Single Nucleotide Variant calling cell by cell](#snp-calling)"
- [readme] Sicelore requires Java and samtools: "requires:

* Java >= 13
* [samtools](https://github.com/samtools/samtools)"

---
name: single-cell-count-matrix-comparison
description: Use when a long-read (Nanopore/PacBio) single-cell count matrix (e.g.
  Sicelore PREFIX_genematrix.txt) must be benchmarked against a same-sample Illumina
  Cell Ranger matrix to quantify agreement.
when_to_use_negative:
- The two count matrices come from different biological samples or incompatible barcode
  whitelists, since barcode intersection would be meaningless
- The long-read matrix was produced by a short-read-free workflow (e.g. Sicelore 2.1)
  without a same-sample Illumina counterpart to validate against
- The goal is isoform, SNP, or fusion-level comparison rather than gene/UMI count
  concordance, which these matrices do not directly capture
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: Sicelore
  role: Produces the Nanopore gene-level count matrix (PREFIX_genematrix.txt) via
    cell barcode/UMI assignment and isoform/gene matrix generation that is validated
    by this comparison
  repo: https://github.com/ucagenomix/sicelore
- name: Oxford Nanopore sequencing
  role: Long-read platform generating the Nanopore dataset whose per-cell gene/UMI
    counts are compared to Illumina
- name: Illumina
  role: Short-read platform generating the Cell Ranger reference matrix used as the
    benchmark for correlation
- name: 10x Genomics Chromium
  role: Single-cell isolation system providing the shared cellBC/UMI structure common
    to both count matrices
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17800-6
    title: High throughput error corrected Nanopore single cell transcriptome sequencing
  source_content_hash: sha256:07e95a207ec974dec355c2d7e475d27133262e5683558b53914dc2d7f7abed7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ucagenomix/sicelore
  tools:
  - 10x Genomics Chromium
  - Illumina
  - Oxford Nanopore sequencing
  - Sicelore
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-count-matrix-comparison

## Summary

Cross-platform validation of Nanopore single-cell gene/UMI count matrices against Illumina Cell Ranger matrices by computing per-cell gene/UMI totals and Pearson correlations, verifying that the long-read dataset faithfully represents the 10x Genomics transcriptome.

## When to use

Use when a long-read (Nanopore/PacBio) single-cell count matrix (e.g. Sicelore PREFIX_genematrix.txt) must be benchmarked against a same-sample Illumina Cell Ranger matrix to quantify agreement.

## When NOT to use

- The two count matrices come from different biological samples or incompatible barcode whitelists, since barcode intersection would be meaningless
- The long-read matrix was produced by a short-read-free workflow (e.g. Sicelore 2.1) without a same-sample Illumina counterpart to validate against
- The goal is isoform, SNP, or fusion-level comparison rather than gene/UMI count concordance, which these matrices do not directly capture

## Inputs

- Sicelore Nanopore gene-level count matrix (PREFIX_genematrix.txt, geneId x cellBC)
- Illumina Cell Ranger filtered gene/cell count matrix (10x Genomics format)
- 10x Genomics Cell Ranger cell-associated barcode list (barcodes.tsv)
- Gene identifier annotation for aligning the two feature spaces

## Outputs

- Per-cell gene counts and UMI totals for Nanopore and Illumina platforms
- Pearson correlation of gene counts and UMI counts across cells
- Mean per-cell per-gene expression Pearson correlation
- comparison_report.csv with computed vs reported benchmark metrics

## How to apply

Load the Sicelore Nanopore gene-level count matrix (PREFIX_genematrix.txt, genes x cellBC) and the Illumina Cell Ranger filtered matrix with pandas or R. Intersect the cell barcodes shared between the two matrices (e.g. the 1121-cell dataset) and align the gene identifier space (e.g. Gencode v38 geneIds). Compute per-cell detected gene counts and per-cell UMI totals for both platforms, then calculate Pearson correlations of gene counts and UMI counts across cells; additionally compute the per-gene Pearson correlation within each shared cell and take the mean across cells. Accept the comparison when gene-count and UMI-count Pearson r approximate 0.99, the mean per-cell expression correlation approximates 0.90, and median per-cell values (2427 genes, 6047 UMIs) match the reported benchmark, confirming that additional full-length PCR and Nanopore library preparation do not skew the long-read data.

## Related tools

- **Sicelore** (Produces the Nanopore gene-level count matrix (PREFIX_genematrix.txt) via cell barcode/UMI assignment and isoform/gene matrix generation that is validated by this comparison) — https://github.com/ucagenomix/sicelore
- **Oxford Nanopore sequencing** (Long-read platform generating the Nanopore dataset whose per-cell gene/UMI counts are compared to Illumina)
- **Illumina** (Short-read platform generating the Cell Ranger reference matrix used as the benchmark for correlation)
- **10x Genomics Chromium** (Single-cell isolation system providing the shared cellBC/UMI structure common to both count matrices)

## Examples

```
import pandas as pd; nano = pd.read_csv('PREFIX_genematrix.txt', sep='\t', index_col=0); illu = pd.read_csv('cellranger_matrix.csv', index_col=0); shared = nano.columns.intersection(illu.columns); r = nano[shared].sum(0).corr(illu[shared].sum(0))
```

## Evaluation signals

- Pearson correlation of per-cell gene counts between Nanopore and Illumina approximates 0.99
- Pearson correlation of per-cell UMI counts approximates 0.99
- Mean per-cell per-gene expression correlation approximates 0.90
- Median Nanopore per-cell values match the reported 2427 genes and 6047 UMIs
- Cell barcode intersection yields the expected 1121 shared cells before correlation is computed

## Limitations

- Agreement is only demonstrated for the shared barcode set; cells unique to one platform are excluded from the comparison
- High correlation at gene-count level does not guarantee isoform-level concordance, since Illumina short reads lack full-length splicing information
- Barcode/UMI assignment depends on Nanopore error correction; uncorrected sequencing errors can mis-assign UMIs and depress counts

## Evidence

- [results] Nanopore and Illumina gene count agreement: [evidence span withheld — anchor status: not_found]
- [methods] workflow rationale: [evidence span withheld — anchor status: not_found]
- [methods] validation criteria: [evidence span withheld — anchor status: not_found]
- [readme] Sicelore matrix generation: [evidence span withheld — anchor status: external_doc]

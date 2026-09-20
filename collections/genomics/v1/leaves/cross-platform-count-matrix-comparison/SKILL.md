---
name: cross-platform-count-matrix-comparison
description: Use when you have matched long-read and short-read (Cell Ranger) single-cell
  10x data and need to quantify how accurately a long-read pipeline recovers cell
  barcodes and gene counts relative to short reads or rival tools.
when_to_use_negative:
- No matched short-read (Cell Ranger) data exist for the sample, so no short-read
  ground-truth matrix is available for correlation
- The goal is cell type assignment or isoform discovery rather than quantifying cell-barcode/gene-count
  accuracy against another platform
- The pipelines being compared were run on different hardware or core counts, making
  runtime comparison invalid
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: scywalker
  role: Long-read single-cell pipeline under benchmark; produces emptydrops-filtered
    gene count files compared to Cell Ranger
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Default aligner used inside scywalker (minimap2_splice, splice preset) for
    read alignment prior to quantification
- name: IsoQuant
  role: Backend used by scywalker for isoform and gene reconstruction and quantification
    feeding the count matrices
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Framework in which scywalker is implemented; its cg tools support querying
    tsv count files used in the comparison
  repo: https://github.com/derijkp/genomecomb
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:623eeba1c5f5b943b72fa2b8b5cbe3250685348b38d475a2734b3cfe97ca0b07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-platform-count-matrix-comparison

## Summary

Benchmark a long-read single-cell workflow (e.g. scywalker) against short-read quantification by computing Pearson correlations of per-cell and gene-per-cell UMI count matrices, plus wall-clock runtime, against competing long-read pipelines such as BLAZE-FLAMES and ONT wf-single-cell.

## When to use

Use when you have matched long-read and short-read (Cell Ranger) single-cell 10x data and need to quantify how accurately a long-read pipeline recovers cell barcodes and gene counts relative to short reads or rival tools.

## When NOT to use

- No matched short-read (Cell Ranger) data exist for the sample, so no short-read ground-truth matrix is available for correlation
- The goal is cell type assignment or isoform discovery rather than quantifying cell-barcode/gene-count accuracy against another platform
- The pipelines being compared were run on different hardware or core counts, making runtime comparison invalid

## Inputs

- Long-read single-cell FASTQ files (Oxford Nanopore 10x data, e.g. scmixology2 GSE154870, 5 lung adenocarcinoma cell lines, 183 target cells)
- Matching Cell Ranger short-read UMI gene count matrix (emptydrops-filtered)
- Count matrices from each competing long-read pipeline (scywalker emptydrops-filtered files, BLAZE v1.1.0 + FLAMES v0.1, wf-single-cell v0.2.8)
- Fixed-core compute environment (24 cores in the benchmark) for runtime measurement

## Outputs

- Cell-level Pearson correlation (R) of per-cell UMI counts vs short-read Cell Ranger for each pipeline
- Gene-per-cell-level Pearson correlation (R) vs Cell Ranger for each pipeline
- Wall-clock runtimes per pipeline on the fixed-core system
- Benchmark agreement table/figures (e.g. Supplementary Figs S3–S4)

## How to apply

Run each pipeline end-to-end on the same FASTQ data on a fixed-core system (e.g. 24 cores, as in the scmixology2 benchmark with 183 expected cells and the ONT preset with minimap2_splice alignment). For the pipeline under test, use the emptydrops-filtered gene count files (the _filtered variants, which contain only emptydrops-approved cells) as the comparison matrix. Compute Pearson correlations of UMI counts against Cell Ranger at two levels: cell level (per-cell UMI totals) and gene-per-cell level, then record wall-clock runtimes for all pipelines on the identical hardware. Success is judged by reproducing the benchmark agreement pattern: scywalker cell-level R = 0.942 vs 0.734 for wf-single-cell and 0.65 for BLAZE-FLAMES, gene-per-cell R = 0.89 / 0.765 / 0.563, and runtimes of 3h37 / 3h19 / 5h41 (Supplementary Figs S3–S4, Section 3.3). Note that gene multimapping reads are weighed so a read mapping to N genes gives each gene a 1/N count, which affects the matrices being compared.

## Related tools

- **scywalker** (Long-read single-cell pipeline under benchmark; produces emptydrops-filtered gene count files compared to Cell Ranger) — https://github.com/derijkp/scywalker
- **minimap2** (Default aligner used inside scywalker (minimap2_splice, splice preset) for read alignment prior to quantification)
- **IsoQuant** (Backend used by scywalker for isoform and gene reconstruction and quantification feeding the count matrices) — https://github.com/ablab/IsoQuant
- **genomecomb** (Framework in which scywalker is implemented; its cg tools support querying tsv count files used in the comparison) — https://github.com/derijkp/genomecomb

## Examples

```
scywalker -v 1 -d 24 -refdir refdb_hg38 -sc_expectedcells 183 test10x  # then compute Pearson R of _filtered counts vs Cell Ranger at cell and gene-per-cell levels
```

## Evaluation signals

- Cell-level Pearson correlation against Cell Ranger reaches the expected range (scywalker R ≈ 0.942; wf-single-cell ≈ 0.734; BLAZE-FLAMES ≈ 0.65 on scmixology2)
- Gene-per-cell correlation matches reported values (R = 0.89 / 0.765 / 0.563 for scywalker / BLAZE-FLAMES / wf-single-cell)
- Comparison uses _filtered (emptydrops-approved cells) count files, not _raw files that include all detected cells
- Runtimes are recorded on the identical fixed-core system (24 cores) for all three pipelines (expected ~3h37 / 3h19 / 5h41)
- Gene multimapping handled consistently: reads mapping to N genes contribute 1/N to each gene

## Limitations

- The scmixology2 comparison is specific to 10x ONT data from 5 lung adenocarcinoma cell lines (~183 cells); correlations may differ for complex tissues
- Organelle chromosomes require special handling: the isoquant-based code often hangs or crashes on very different organelle data, so organelles must be specified in scywalker_makerefdir
- Gene-per-cell correlation is lower than cell-level correlation for all tools (e.g. 0.89 vs 0.942 for scywalker), so conclusions differ by granularity

## Evidence

- [results] scywalker achieves a higher cell identification correlation with short-read data (R = 0.942) than wf-single-cell (R = 0.734) and BLAZE-FLAMES (R = 0.65): [evidence span withheld — anchor status: not_found]
- [results] filtered counts contain only emptydrops-approved cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] multimapping reads are weighed: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [methods] The default aligner is minimap2_splice: "The default aligner is minimap2_splice (minimap2 with the splice preset)"
- [readme] organelle-specific algorithm avoids isoquant crashes: "If there are organelles in the genome sequence, it is important to specify them so the organelle specific algorithm can be used"
- [abstract] excellent correlation with short-read analyses: "Assessment of datasets from the human brain, Arabidopsis leaves, and previously benchmarked data from mixed cell lines demonstrate excellent correlation with short-read analyses at both the"

---
name: umi-count-correlation-analysis
description: Use when you have long-read single-cell (10x ONT) count matrices and
  a matching short-read Cell Ranger run of the same library and want to quantify cell
  barcode detection and quantification agreement via correlation.
when_to_use_negative:
- No matched short-read (Cell Ranger) data exist for the library — scywalker is designed
  to analyse 10x nanopore data without matching short reads, so this correlation is
  impossible and validation must use other signals
- The datasets come from different libraries or samples, so cell-level correlation
  would be meaningless
- Comparing bulk (not per-cell) long-read quantification only, where per-cell correlation
  of emptydrops-filtered cells does not apply
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: 'Long-read pipeline under test: end-to-end cell barcode finding, minimap2_splice
    alignment, isoform/gene reconstruction and per-cell UMI counts; benchmarked against
    Cell Ranger via correlation'
  repo: https://github.com/derijkp/scywalker
- name: genomecomb
  role: Package within which scywalker is implemented; provides emptydrops cell filtering
    and the _filtered/_raw count files used for correlation computation
  repo: https://github.com/derijkp/genomecomb
- name: minimap2
  role: Default aligner used inside scywalker (minimap2 with the splice preset) to
    produce the alignments underlying the quantified counts
- name: isoquant
  role: IsoQuant-based code used by scywalker for isoform and gene reconstruction
    and quantification (hangs/crashes on organelle data, hence the organelle-specific
    path)
  repo: https://github.com/ablab/IsoQuant
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:4e200e762c60a29c38287024e54bf77073c26b7b7789da7882bda1ec294f9d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - genomecomb
  - isoquant
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# umi-count-correlation-analysis

## Summary

Benchmark long-read single-cell pipelines by computing Pearson correlations of their UMI count matrices against a matching short-read (Cell Ranger) quantification, at the cell level and the gene-per-cell level, as done for scywalker vs BLAZE-FLAMES vs ONT wf-single-cell on scmixology2.

## When to use

Use when you have long-read single-cell (10x ONT) count matrices and a matching short-read Cell Ranger run of the same library and want to quantify cell barcode detection and quantification agreement via correlation.

## When NOT to use

- No matched short-read (Cell Ranger) data exist for the library — scywalker is designed to analyse 10x nanopore data without matching short reads, so this correlation is impossible and validation must use other signals
- The datasets come from different libraries or samples, so cell-level correlation would be meaningless
- Comparing bulk (not per-cell) long-read quantification only, where per-cell correlation of emptydrops-filtered cells does not apply

## Inputs

- Long-read (ONT) single-cell 10x FASTQ files from a benchmarked library (e.g. scmixology2, GSE154870, 5 lung adenocarcinoma cell lines, ~183 cells)
- Matching Cell Ranger short-read gene count / UMI quantification for the same library
- emptydrops-filtered gene count files (_filtered) and raw count files (_raw) from each long-read pipeline
- Reference directory (e.g. hg38 built with scywalker_makerefdir, with organelle chromosomes specified)
- Wall-clock runtimes per pipeline on a fixed core count

## Outputs

- Pearson correlation of per-cell UMI counts vs Cell Ranger for each pipeline (cell-level R)
- Pearson correlation of gene-per-cell counts vs Cell Ranger for each pipeline (gene-level R)
- Wall-clock runtimes per pipeline on the fixed hardware allocation
- Ranked comparison of cell identification accuracy across pipelines

## How to apply

Run each long-read pipeline (e.g. scywalker with -sc_expectedcells 183, the ONT preset, and minimap2_splice alignment; BLAZE v1.1.0 + FLAMES v0.1; ONT wf-single-cell v0.2.8) on the same FASTQ data on a fixed core allocation (e.g. -d 24) so runtimes are comparable. For scywalker, use the emptydrops-filtered gene count files (the _filtered files containing only emptydrops-approved cells) rather than the _raw files. Compute Pearson correlations at two levels: per-cell UMI totals against Cell Ranger (cell identification accuracy) and per-gene-per-cell counts against Cell Ranger (quantification accuracy). Record wall-clock runtimes under the identical resource setting as a secondary benchmark. In the scmixology2 benchmark this reproduced cell-level R = 0.942 / 0.734 / 0.65 and gene-per-cell R = 0.89 / 0.765 / 0.563 for scywalker / BLAZE-FLAMES / wf-single-cell, with runtimes of 3h37 / 3h19 / 5h41.

## Related tools

- **scywalker** (Long-read pipeline under test: end-to-end cell barcode finding, minimap2_splice alignment, isoform/gene reconstruction and per-cell UMI counts; benchmarked against Cell Ranger via correlation) — https://github.com/derijkp/scywalker
- **genomecomb** (Package within which scywalker is implemented; provides emptydrops cell filtering and the _filtered/_raw count files used for correlation computation) — https://github.com/derijkp/genomecomb
- **minimap2** (Default aligner used inside scywalker (minimap2 with the splice preset) to produce the alignments underlying the quantified counts)
- **isoquant** (IsoQuant-based code used by scywalker for isoform and gene reconstruction and quantification (hangs/crashes on organelle data, hence the organelle-specific path)) — https://github.com/ablab/IsoQuant

## Examples

```
scywalker -v 1 -d 24 -refdir hg38 -sc_expectedcells 183 testproject
```

## Evaluation signals

- Cell-level Pearson correlation against Cell Ranger falls near the reported benchmark range (scywalker R = 0.942 vs wf-single-cell 0.734 and BLAZE-FLAMES 0.65 on scmixology2)
- Gene-per-cell Pearson correlation against Cell Ranger reproduces the reported values (0.89 / 0.765 / 0.563 for scywalker / BLAZE-FLAMES / wf-single-cell)
- Only emptydrops-approved cells (is_cell = 1) are included when using the _filtered gene count files, not all detected 'cells' from _raw
- Number of cells analysed matches the expected target (183 cells for the scmixology2 benchmark) via -sc_expectedcells 183
- All pipelines ran on the same fixed 24-core system so recorded wall-clock runtimes (3h37 / 3h19 / 5h41) are directly comparable

## Limitations

- Gene multimapping reads are weighed (a read mapping to N genes gives each gene 1/N), which affects quantification-level correlations relative to unweighted schemes
- The isoquant-based code inside scywalker often hangs or crashes on very different organelle data, so organelle chromosomes must be specified in scywalker_makerefdir for the organelle-specific algorithm to be used
- Correlation validation requires matching short-read Cell Ranger data, which scywalker otherwise does not need; results are benchmark-specific (human brain, Arabidopsis leaves, mixed cell line datasets in the paper)

## Evidence

- [results] scywalker achieves a higher cell identification correlation with short-read data (R = 0.942) than wf-single-cell (R = 0.734) and BLAZE-FLAMES (R = 0.65): [evidence span withheld — anchor status: not_found]
- [results] In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells".: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [methods] The default aligner is minimap2_splice (minimap2 with the splice preset): "The default aligner is minimap2_splice (minimap2 with the splice preset)"
- [results] multimapping reads are weighed (if maps to N genes -> each gets 1/N count): "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [methods] The isoquant based code often hangs or crashes on the very different organelle data.: "The isoquant based code often hangs or crashes on the
very different organelle data."
- [readme] If there are organelles in the genome sequence, it is important to specify them so the organelle specific algorithm can be used: "If there are organelles in the genome sequence, it is important to specify them so the organelle
specific algorithm can be used"

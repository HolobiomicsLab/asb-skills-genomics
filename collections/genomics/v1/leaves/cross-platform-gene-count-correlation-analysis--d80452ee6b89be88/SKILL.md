---
name: cross-platform-gene-count-correlation-analysis
description: Use when you have long-read single-cell (10x) FASTQs processed with scywalker
  plus a matched short-read count matrix for the same sample, and you need to quantify
  cross-platform concordance at cell and per-cell gene-count level.
when_to_use_negative:
- The data are short-read-only single-cell counts with no long-read FASTQs, so there
  is no scywalker PacBio run to compare against.
- You need isoform-level (strict >= 90% transcript coverage) concordance rather than
  cell-level and per-cell gene-count concordance; this skill validates gene quantification
  only.
- The genome contains organelles and scywalker_makerefdir was run without '-organelles',
  since the IsoQuant-based code can hang or crash on organelle data, invalidating
  the comparison.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scywalker
  role: End-to-end long-read single-cell pipeline run with '-preset pacbio' to produce
    the long-read per-cell gene counts being compared
  repo: https://github.com/derijkp/scywalker
- name: scywalker_makerefdir
  role: Builds the hg38 reference directory, adding the PacBio minimap2 index via
    '-pacbioindex 1' (only the ONT index is built by default)
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Default read aligner (minimap2_splice splice preset) used for the PacBio long-read
    alignments underlying the counts
- name: IsoQuant
  role: Basis of the isoform/gene reconstruction and quantification step that generates
    the per-cell gene counts
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Framework in which scywalker is implemented; its cg tools query the output
    tsv count tables used for barcode matching
  repo: https://github.com/derijkp/genomecomb/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:aaf24c10f9be88662a7fa3e2e24622e34ad3a047b4cc739ed88ab086cb16383e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  - scywalker_makerefdir
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# cross-platform-gene-count-correlation-analysis

## Summary

Benchmark long-read single-cell scywalker quantification (e.g. PacBio HiFi with '-preset pacbio') against matched short-read quantification by matching cell barcodes and computing Pearson correlations of total counts per cell and gene counts per cell, following the HG002 10x lymphoblastoid validation (R = 0.986 cell level, R = 0.886 per-cell gene counts).

## When to use

Use when you have long-read single-cell (10x) FASTQs processed with scywalker plus a matched short-read count matrix for the same sample, and you need to quantify cross-platform concordance at cell and per-cell gene-count level.

## When NOT to use

- The data are short-read-only single-cell counts with no long-read FASTQs, so there is no scywalker PacBio run to compare against.
- You need isoform-level (strict >= 90% transcript coverage) concordance rather than cell-level and per-cell gene-count concordance; this skill validates gene quantification only.
- The genome contains organelles and scywalker_makerefdir was run without '-organelles', since the IsoQuant-based code can hang or crash on organelle data, invalidating the comparison.

## Inputs

- Long-read (PacBio HiFi) 10x single-cell FASTQ files from a matched sample (e.g. HG002 lymphoblastoid cell line)
- Matched short-read single-cell gene count matrix for the same sample
- Reference genome FASTA and GTF transcript annotation (hg38) for scywalker_makerefdir
- Expected cell number for emptydrops filtering (-sc_expectedcells)

## Outputs

- Filtered gene count matrix (emptydrops-approved cells only) from the long-read scywalker run (sc_gene_counts_filtered)
- Per-cell matched table of long-read vs short-read total and gene-level counts
- pacbio_vs_shortread_correlations.tsv with cell-level Pearson correlation (R = 0.986) and gene-count Pearson correlation (R = 0.886)

## How to apply

Build a reference directory with scywalker_makerefdir from the hg38 FASTA and GTF, adding the PacBio minimap2 index with '-pacbioindex 1' because only the ONT index is built by default. Assemble a project/sample directory (samples/fastq layout) for the PacBio 10x FASTQs and run scywalker with '-preset pacbio', a required '-refdir', '-sc_expectedcells' for emptydrops cell filtering, and parallel options (e.g. '-d' with '-dmaxmem' on a local machine), keeping the default minimap2_splice alignment and IsoQuant-based isoform/gene reconstruction. Extract the sc_gene_counts_filtered matrix (only emptydrops-approved cells) from the long-read output and match cell barcodes to the matched short-read quantification. Compute the Pearson correlation of total counts per cell (expected R = 0.986, Supplementary Fig. S14) and of gene counts per cell (expected R = 0.886, Supplementary Fig. S15), then write both values plus the per-cell table to pacbio_vs_shortread_correlations.tsv.

## Related tools

- **scywalker** (End-to-end long-read single-cell pipeline run with '-preset pacbio' to produce the long-read per-cell gene counts being compared) — https://github.com/derijkp/scywalker
- **scywalker_makerefdir** (Builds the hg38 reference directory, adding the PacBio minimap2 index via '-pacbioindex 1' (only the ONT index is built by default)) — https://github.com/derijkp/scywalker
- **minimap2** (Default read aligner (minimap2_splice splice preset) used for the PacBio long-read alignments underlying the counts)
- **IsoQuant** (Basis of the isoform/gene reconstruction and quantification step that generates the per-cell gene counts) — https://github.com/ablab/IsoQuant
- **genomecomb** (Framework in which scywalker is implemented; its cg tools query the output tsv count tables used for barcode matching) — https://github.com/derijkp/genomecomb/

## Examples

```
scywalker_makerefdir -pacbioindex 1 -organelles chrM chrPt refdir_hg38 hg38.fa genes.gtf
```

## Evaluation signals

- Cell-level Pearson correlation of total counts per matched cell barcode reaches R = 0.986 (Supplementary Fig. S14).
- Per-cell gene-count Pearson correlation reaches R = 0.886 (Supplementary Fig. S15).
- Only cells flagged by emptydrops (is_cell = 1) from the sc_gene_counts_filtered file are included in the correlation.
- Every correlation pair involves a cell barcode present in both the long-read and short-read quantifications (matched barcodes only).
- Both correlation values and the underlying per-cell table are written to pacbio_vs_shortread_correlations.tsv.

## Limitations

- Only the ONT minimap2 index is built by scywalker_makerefdir by default; forgetting '-pacbioindex 1' yields an unusable reference for PacBio data.
- The IsoQuant-based code often hangs or crashes on organelle data, so organelles must be flagged at reference construction.
- Correlations are reported at the cell and per-cell gene-count levels; this skill does not establish isoform-level equivalence, and gene multimapping reads are weighed (1/N per gene), which affects counts.

## Evidence

- [intro] scywalker supports PacBio and its short-read concordance benchmark: "scywalker is a package designed to analyse single cell (10x) Oxford nanopore long read data (without the need for matching short read data), but it can also analyse PacBio data"
- [readme] PacBio index must be added explicitly at reference construction: "By default scywalker_makerefdir will only create an ont minimap2 index for the genome.
If you want to analyze PacBio data, the appropriate index can be added"
- [results] Filtered count files contain only emptydrops-approved cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] emptydrops cell flag semantics used to select cells for the correlation: "is_cell : 1 if emptydrops categorized the cell as real, 0 if not"
- [supplementary] Expected PacBio vs short-read correlations from the HG002 benchmark: [evidence span withheld — anchor status: not_found]
- [methods] emptydrops cell filtering is a configured step of the scywalker run: "This information is required for filtering cells using emptydrops"

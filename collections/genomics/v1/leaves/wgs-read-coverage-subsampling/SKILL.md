---
name: wgs-read-coverage-subsampling
description: Use when you need to determine the sequencing depth required for a read-based
  prediction pipeline to reach a target performance level (e.g. >90% precision/recall)
  or its optimum.
when_to_use_negative:
- When the goal is per-base variant calling quality assessment rather than feature
  (exon/boundary) prediction accuracy versus depth
- When only a single fixed-coverage sample is available and no reference annotation
  exists to compute precision/recall
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Seqtk
  role: Subsamples the WGS read set in 10% increments to generate the coverage titration
    series
- name: EXFI
  role: Downstream pipeline (build_baited_bloom_filter + build_splice_graph, k=25)
    whose exon prediction is evaluated at each coverage
  repo: https://github.com/jlanga/exfi
- name: bedtools
  role: BEDTools intersect compares predicted exons to reference exons with a mutual
    overlap of at least 95%
- name: Pandas
  role: Tabulates precision and recall versus genome coverage for all subsamples into
    coverage_metrics.csv
- name: biobloomcategorizer
  role: Read filtering within the EXFI pipeline evaluated at each depth
  repo: https://github.com/bcgsc/biobloom
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:1ee98e00d00e7626e118120c3e4f8216f6a4c2acd15e281566f930ecf7737776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - EXFI
  - Pandas
  - Python
  - Seqtk
  - bedtools
  - biobloomcategorizer
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# wgs-read-coverage-subsampling

## Summary

Generate a WGS coverage titration series by subsampling reads in 10% increments with Seqtk, then measure how tool performance (e.g. EXFI exon precision/recall) scales with sequencing depth to find the minimum and optimal coverage.

## When to use

Use when you need to determine the sequencing depth required for a read-based prediction pipeline to reach a target performance level (e.g. >90% precision/recall) or its optimum.

## When NOT to use

- When the goal is per-base variant calling quality assessment rather than feature (exon/boundary) prediction accuracy versus depth
- When only a single fixed-coverage sample is available and no reference annotation exists to compute precision/recall

## Inputs

- Raw WGS sequencing reads (FASTQ)
- Reference genome, transcriptome, and GFF3 annotation (e.g. Ensembl release 91, D. rerio GRCz10)
- Reference exon coordinates in GFF/BED format

## Outputs

- Subsampled read sets spanning a coverage titration series (10% increments)
- coverage_metrics.csv tabulating precision, recall, and F1 versus genome coverage
- Precision/recall-vs-coverage curve identifying minimum adequate and optimal depth

## How to apply

Start from a simulated or real WGS read set (here, zebrafish D. rerio reads against Ensembl release 91 assembly GRCz10) and subsample it in 10% increments using Seqtk to produce a coverage titration series. Run the downstream pipeline (for EXFI: build_baited_bloom_filter followed by build_splice_graph with k-mer length 25 bp, testing both low 4 GB and high 60 GB Bloom filter memory settings) on each subsample. Compare predicted exons to reference exon coordinates converted from chromosome-based GFF/BED to transcript-based BED using BEDTools intersect with a mutual overlap threshold of at least 95%, computing precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1. Tabulate metrics versus genome coverage (e.g. coverage_metrics.csv with Pandas) and interpret the trajectory: the rationale is that low coverage starves k-mer evidence for intron-exon boundaries while excessive coverage accumulates sequencing errors, so performance rises to a plateau and then declines.

## Related tools

- **Seqtk** (Subsamples the WGS read set in 10% increments to generate the coverage titration series)
- **EXFI** (Downstream pipeline (build_baited_bloom_filter + build_splice_graph, k=25) whose exon prediction is evaluated at each coverage) — https://github.com/jlanga/exfi
- **bedtools** (BEDTools intersect compares predicted exons to reference exons with a mutual overlap of at least 95%)
- **Pandas** (Tabulates precision and recall versus genome coverage for all subsamples into coverage_metrics.csv)
- **biobloomcategorizer** (Read filtering within the EXFI pipeline evaluated at each depth) — https://github.com/bcgsc/biobloom

## Examples

```
seqtk sample wgs_reads.fastq 0.1 > subsample_10pct.fastq && exfi build_baited_bloom_filter --kmer 25 transcriptome.fa subsample_10pct.fastq
```

## Evaluation signals

- The precision/recall trajectory should exceed 90% for both metrics by approximately 16x coverage
- Maxima should plateau at approximately 96.7% precision and 94.7% recall between 26x and 37x coverage
- Both precision and recall should decline beyond ~37x due to accumulated sequencing errors
- Every subsample must produce parseable EXFI output and BEDTools comparisons (no missing points in the titration series)

## Limitations

- Beyond the optimum, both precision and recall decline due to sequencing errors, so more depth is not always better
- Results depend on transcriptome quality: lowly expressed transcripts may be fragmented and tissue-specific transcripts missing, affecting prediction metrics
- Bloom filter memory setting (low 4 GB vs high 60 GB) influences the trade-off between BF FPR and runtime

## Evidence

- [results] At 16× sequencing depth, EXFI's zebrafish precision and recall already exceed 90%: [evidence span withheld — anchor status: not_found]
- [other] Subsampling workflow: [evidence span withheld — anchor status: not_found]
- [methods] Bloom filter false positives: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] Read filtering benefit: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] Dataset provenance: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38"

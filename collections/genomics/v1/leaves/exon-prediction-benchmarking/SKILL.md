---
name: exon-prediction-benchmarking
description: Use when evaluating exon/IEB predictions from a transcriptome+WGS pipeline
  (EXFI, ChopStitch, GMAP) against Ensembl GFF3 annotations, or when measuring how
  Bloom filter memory, read filtering, or sequencing depth change BF FPR and precision/recall.
when_to_use_negative:
- You only need mappability or SNP/indel genotyping counts (e.g. the tench SNP discovery
  application) rather than exon-coordinate precision/recall
- No reference annotation exists — transcript-based BED ground truth cannot be built,
  so only mappability-style evaluation is possible (e.g. draft-genome species like
  sugar pine or axolotl)
- Input is a splice-aware alignment (GMAP) output where IEB coordinates are already
  extracted, not pipeline-predicted exons needing benchmarking
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: EXFI
  role: 'Pipeline under test: build_baited_bloom_filter and build_splice_graph predict
    exons and the splice graph; its compare_to_gff3 script computes precision, recall,
    and F1'
  repo: https://github.com/jlanga/exfi
- name: bedtools
  role: bedtools intersect compares predicted vs reference transcript-based BED coordinates
    with mutual overlap of at least 95%
  repo: https://github.com/jlanga/exfi
- name: biobloomtools
  role: biobloomcategorizer filters WGS reads not sharing a k-mer with the transcriptome,
    lowering BF FPR before benchmarking
  repo: https://github.com/bcgsc/biobloom
- name: abyss-bloom
  role: Used to build the cascading Bloom filter whose BF FPR is recorded as a benchmark
    metric
  repo: https://github.com/bcgsc/abyss
- name: wgsim
  role: Simulates zebrafish/human WGS reads for the synthetic benchmark datasets
- name: Seqtk
  role: Subsamples WGS reads in 10% increments to generate the sequencing-depth titration
    series
- name: Pandas
  role: Tabulates BF FPR and classification metrics into CSV benchmark tables
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_004
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:6bdcbdc4dd6e1d3c9efda616253760f2b82146f767d2585f2967bc13fbe01fb4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - ABySS
  - BioPython
  - EXFI
  - Pandas
  - Python
  - Seqtk
  - abyss-bloom
  - bedtools
  - biobloomtools
  - wgsim
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# exon-prediction-benchmarking

## Summary

Benchmarks exon and splice-graph predictions (e.g. from EXFI) against reference GFF3 annotations using transcript-based BED conversion, bedtools mutual-overlap matching, and precision/recall/F1 computed with compare_to_gff3, while tracking Bloom-filter false-positive rate (BF FPR).

## When to use

Use when evaluating exon/IEB predictions from a transcriptome+WGS pipeline (EXFI, ChopStitch, GMAP) against Ensembl GFF3 annotations, or when measuring how Bloom filter memory, read filtering, or sequencing depth change BF FPR and precision/recall.

## When NOT to use

- You only need mappability or SNP/indel genotyping counts (e.g. the tench SNP discovery application) rather than exon-coordinate precision/recall
- No reference annotation exists — transcript-based BED ground truth cannot be built, so only mappability-style evaluation is possible (e.g. draft-genome species like sugar pine or axolotl)
- Input is a splice-aware alignment (GMAP) output where IEB coordinates are already extracted, not pipeline-predicted exons needing benchmarking

## Inputs

- Reference genome, transcriptome, and GFF3 annotation (Ensembl release 91; GRCz10 for zebrafish, GRCh38 for human)
- Simulated whole-genome sequencing reads (wgsim) or real WGS reads
- Predicted exon coordinates / splice graph in GFA1 format from build_splice_graph
- Reference exons converted to transcript-based BED (strand- and order-aware)

## Outputs

- Precision, recall, and F1 per run (computed by compare_to_gff3)
- BF FPR per Bloom filter memory setting and filtering condition
- Benchmark tables such as exfi_memory_benchmark_metrics.csv or coverage_metrics.csv (metrics vs coverage)

## How to apply

Download the reference genome, transcriptome, and GFF3 annotation (e.g. Ensembl release 91, assemblies GRCz10 and GRCh38), and simulate WGS reads with wgsim if running the synthetic benchmark. Run the prediction pipeline (EXFI's build_baited_bloom_filter then build_splice_graph, with k-mer length fixed at 25 bp and Bloom filter sizes such as 4 GB vs 60 GB) and record the BF FPR for each setting. Convert reference exon coordinates from chromosome-based GFF3 into transcript-based BED, accounting for strand and exon order, then compare predicted vs reference exons with bedtools intersect requiring mutual overlap of at least 95%. Compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 with the compare_to_gff3 script, and tabulate metrics (e.g. BF FPR, precision, recall per species x memory setting) in a CSV such as exfi_memory_benchmark_metrics.csv. Expect filtered reads to cut BF FPR by 68%-75% and improve F1 (89.8% unfiltered to 94.8% at low memory in zebrafish).

## Related tools

- **EXFI** (Pipeline under test: build_baited_bloom_filter and build_splice_graph predict exons and the splice graph; its compare_to_gff3 script computes precision, recall, and F1) — https://github.com/jlanga/exfi
- **bedtools** (bedtools intersect compares predicted vs reference transcript-based BED coordinates with mutual overlap of at least 95%) — https://github.com/jlanga/exfi
- **biobloomtools** (biobloomcategorizer filters WGS reads not sharing a k-mer with the transcriptome, lowering BF FPR before benchmarking) — https://github.com/bcgsc/biobloom
- **abyss-bloom** (Used to build the cascading Bloom filter whose BF FPR is recorded as a benchmark metric) — https://github.com/bcgsc/abyss
- **wgsim** (Simulates zebrafish/human WGS reads for the synthetic benchmark datasets)
- **Seqtk** (Subsamples WGS reads in 10% increments to generate the sequencing-depth titration series)
- **Pandas** (Tabulates BF FPR and classification metrics into CSV benchmark tables)

## Examples

```
docker run --rm exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- BF FPR falls in the reported ranges: 32.6% to 8.1% at low memory without vs with read filtering, and 8.1% to 0.4% from 4 GB to 60 GB in zebrafish
- Zebrafish classification metrics near reported values: 95.8% precision / 93.8% recall at 4 GB and 96.6% / 94.6% at 60 GB
- Read-filtering run shows 68%-75% relative BF FPR reduction and F1 at least as high as the unfiltered run (94.8% vs 89.8% at low memory)
- Depth curve exceeds 90% precision/recall by about 16x coverage and plateaus near 96.7% precision / 94.7% recall between 26x and 37x
- All exon comparisons pass bedtools intersect with mutual overlap of at least 95% and are scored by compare_to_gff3 rather than raw coordinate listing

## Limitations

- Benchmark precision is only marginally affected by BF FPR: raising memory from 4 GB to 60 GB cut BF FPR from 8.1% to 0.4% but improved zebrafish precision/recall by under 1 point
- Mappability scores (e.g. EXFI's 99.5% on herring) are more optimistic than exon precision because they cover only interior exon sections, not full exon sequences
- Results degrade when the input transcriptome is de novo-assembled from a single tissue (Atlantic herring case: lowly expressed transcripts fragmented, other tissues' transcripts missing)
- Metrics beyond ~37x coverage decline due to sequencing errors; beyond the optimum the FPR-induced errors outweigh added coverage

## Evidence

- [results] Filtering the reads resulted in a 68%-75% reduction of the BF FPR while also slightly improving all the classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38"
- [methods] exons of length less than k + q (q by default five) are filtered out: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [methods] It does not return false negatives, but it can produce false positives with a tunable false-positive rate (BF FPR): "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [discussion] even with higher BF FPR rates, EXFI obtained better prediction metrics, except for the Atlantic herring (Table 3): "even with higher BF FPR rates, EXFI obtained better prediction metrics, except for the Atlantic herring (Table 3)"
- [readme] docker build --rm --tag exfi:1.5.6 github.com/jlanga/exfi-docker: [evidence span withheld — anchor status: not_found]

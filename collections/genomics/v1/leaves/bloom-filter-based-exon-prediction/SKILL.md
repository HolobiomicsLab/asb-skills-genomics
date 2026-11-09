---
name: bloom-filter-based-exon-prediction
description: Use when you have a de novo transcriptome assembly and raw whole-genome
  sequencing reads but no reliable reference genome, and need exon sequences, intron-exon
  boundaries, or a splice graph for downstream SNP genotyping or annotation.
when_to_use_negative:
- A reliable reference genome and annotation exist and a splice-aware aligner such
  as GMAP can align transcripts directly to extract IEB coordinates
- Input is RNA-Seq reads alone without an assembled transcriptome — EXFI requires
  a transcriptome assembly plus WGS reads
- The transcriptome derives from a single tissue with lowly expressed transcripts
  fragmented — the article notes this herring case degraded exon prediction
tools:
- name: EXFI
  role: 'Python pipeline performing the whole skill: build_baited_bloom_filter for
    read filtering and Bloom filter construction, build_splice_graph for exon/splice
    graph prediction, gfa1_to_fasta for exon extraction'
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: Builds the cascading Bloom filter over the retained WGS reads inside build_baited_bloom_filter
  repo: https://github.com/bcgsc/abyss/
- name: biobloomcategorizer
  role: BioBloom Tools read categorizer that discards WGS reads not sharing at least
    one k-mer with the transcriptome
  repo: https://github.com/bcgsc/biobloom.git
- name: bedtools
  role: Benchmarking exon predictions against reference coordinates via intersect
    with >=95% mutual overlap
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:4bfcdfbff28e9347ff1f3403c7c937d59a5eb96d4655b5ab5928cdb3ebd2b828
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BCFTools
  - BioPython
  - Bowtie2
  - EXFI
  - Pandas
  - Python
  - Seqtk
  - abyss-bloom
  - bedtools
  - biobloomcategorizer
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# bloom-filter-based-exon-prediction

## Summary

Predict exon sequences and a splice graph from an assembled transcriptome plus raw WGS reads without a reference genome, using EXFI's baited Bloom filter (abyss-bloom, biobloomcategorizer) followed by splice-graph construction emitted in GFA1 format.

## When to use

Use when you have a de novo transcriptome assembly and raw whole-genome sequencing reads but no reliable reference genome, and need exon sequences, intron-exon boundaries, or a splice graph for downstream SNP genotyping or annotation.

## When NOT to use

- A reliable reference genome and annotation exist and a splice-aware aligner such as GMAP can align transcripts directly to extract IEB coordinates
- Input is RNA-Seq reads alone without an assembled transcriptome — EXFI requires a transcriptome assembly plus WGS reads
- The transcriptome derives from a single tissue with lowly expressed transcripts fragmented — the article notes this herring case degraded exon prediction

## Inputs

- Assembled transcriptome (FASTA)
- Raw whole-genome sequencing (WGS) reads (FASTQ/FASTA, e.g. simulated wgsim reads or pooled genomic reads)
- Optional: reference GFF3/GFF annotation and genome for benchmarking

## Outputs

- Predicted exon sequences (FASTA via gfa1_to_fasta)
- Splice graph in GFA1 format encoding exons as nodes, IEBs as edges, transcripts as paths
- Optional benchmark table of precision, recall, F1 per condition (e.g. coverage_metrics.csv)

## How to apply

Run EXFI's build_baited_bloom_filter on the transcriptome and WGS reads: it first discards with biobloomcategorizer each WGS read that does not share at least one k-mer with the transcriptome (this read filtering cut BF FPR by 68%-75% and slightly improved classification metrics), then builds a cascading Bloom filter with ABySS. Next run build_splice_graph with a chosen k-mer length (k=25 bp default; F1 is stable for odd k in 23-35, validated up to 4 GB and 60 GB Bloom filter memory settings) to predict in one step the exon sequences and the splice graph in GFA1 format, then extract exons with gfa1_to_fasta. Internally the script filters exons shorter than k+q (q default 5), merges exons overlapping by more than 10 bases, and polishes long-overlap exon pairs by inspecting donor/acceptor GU/AG sites. Validate by converting reference GFF exon coordinates to transcript-based BED and comparing with BEDTools intersect requiring >=95% mutual overlap, computing precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1; expect >90% precision/recall at >=16x coverage and maxima near 96.7%/94.7% at 26-37x on zebrafish-like data.

## Related tools

- **EXFI** (Python pipeline performing the whole skill: build_baited_bloom_filter for read filtering and Bloom filter construction, build_splice_graph for exon/splice graph prediction, gfa1_to_fasta for exon extraction) — https://github.com/jlanga/exfi
- **abyss-bloom** (Builds the cascading Bloom filter over the retained WGS reads inside build_baited_bloom_filter) — https://github.com/bcgsc/abyss/
- **biobloomcategorizer** (BioBloom Tools read categorizer that discards WGS reads not sharing at least one k-mer with the transcriptome) — https://github.com/bcgsc/biobloom.git
- **bedtools** (Benchmarking exon predictions against reference coordinates via intersect with >=95% mutual overlap)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_baited_bloom_filter --help
```

## Evaluation signals

- F1 stays near-constant across odd k-mer lengths 23-35 at both 4 GB and 60 GB Bloom filter settings, justifying k=25 default
- Precision (TP/(TP+FP)) and recall (TP/(TP+FN)) exceed 90% at ~16x coverage and plateau around 96.7%/94.7% at 26-37x on the zebrafish simulation
- Read filtering reduces the BF FPR by 68%-75% relative to unfiltered input
- Output splice graph parses as valid GFA1 and exon FASTA sequences extracted by gfa1_to_fasta correspond to graph nodes
- In the tench retrospective test, all IEBs proximal to the 96 known transcriptomic SNPs are detected (92/96 genotyped, 100% precision over the SNP set)

## Limitations

- Bloom filters return no false negatives but can produce false positives with a tunable BF FPR, so raw unfiltered runs inflate false IEB calls
- Performance degrades with low-quality or single-tissue transcriptomes: in Atlantic herring, EXFI was less accurate predicting exons than comparators even though mappability was highest (99.5%)
- High mappability scores cover only interior exon sections, not full exon sequences, and are more optimistic than exon precision
- Optimal precision/recall requires ~26-37x WGS coverage; beyond that sequencing errors degrade both metrics

## Evidence

- [abstract] EXFI pipeline predicts splice graph and exons from transcriptome + WGS reads: "This paper presents EXFI, a Python pipeline that predicts the splice graph and exon sequences using an assembled transcriptome and raw whole-genome sequencing reads."
- [methods] Read filtering rule via biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [methods] Exon length filter threshold: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [methods] GU/AG polish step: "each pair of exons with a long‐enough overlap is inspected for the donor/acceptor sites (usually GU/AG; 2e) and correctly trimmed if possible"
- [results] Filtering improves FPR and metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [readme] README summary of toolchain: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"

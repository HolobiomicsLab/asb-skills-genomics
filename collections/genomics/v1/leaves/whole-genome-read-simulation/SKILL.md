---
name: whole-genome-read-simulation
description: Use when benchmarking exon/IEB prediction tools on species with a reference
  genome, and you need synthetic WGS reads plus a known GFF3 truth set to compute
  precision, recall, and F1.
when_to_use_negative:
- The study species has no reference genome or reliable annotation — EXFI's purpose
  is the reference-free case, and no truth set exists to score the simulation
- The goal is real variant discovery rather than method benchmarking — simulated reads
  with a known error model will not reflect the input transcriptome quality issues
  seen in real samples (e.g., the single-tissue herring transcriptome)
- You need mapping locations, not just classification — Bloom filter classification
  is faster than alignment but does not provide mapping positions
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: wgsim
  role: Simulates the synthetic WGS reads from the zebrafish and human reference genomes
    used as EXFI input
- name: EXFI
  role: Consumes the simulated WGS reads to build baited Bloom filters and predict
    exons/splice graphs via build_baited_bloom_filter and build_splice_graph
  repo: https://github.com/jlanga/exfi
- name: bedtools
  role: Compares predicted exons to the reference GFF3 (converted to transcript-based
    BED) with at least 95% mutual overlap to compute precision/recall
- name: Pandas
  role: Assembles the BF FPR and classification metrics from all runs (2 species x
    2 memory settings) into the benchmark CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:5bcb48d176bdb08829d6d9eb5d45229f059d7c68a94d1e6ad35ac3e5d755b622
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - EXFI
  - Pandas
  - Python
  - bedtools
  - wgsim
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# whole-genome-read-simulation

## Summary

Simulates WGS reads from reference genomes (zebrafish GRCz10, human GRCh38, Ensembl release 91) with wgsim to create synthetic benchmark inputs for EXFI, whose splice graph and exon predictions can then be scored against known reference annotations.

## When to use

Use when benchmarking exon/IEB prediction tools on species with a reference genome, and you need synthetic WGS reads plus a known GFF3 truth set to compute precision, recall, and F1.

## When NOT to use

- The study species has no reference genome or reliable annotation — EXFI's purpose is the reference-free case, and no truth set exists to score the simulation
- The goal is real variant discovery rather than method benchmarking — simulated reads with a known error model will not reflect the input transcriptome quality issues seen in real samples (e.g., the single-tissue herring transcriptome)
- You need mapping locations, not just classification — Bloom filter classification is faster than alignment but does not provide mapping positions

## Inputs

- Reference genome FASTA (e.g., GRCz10, GRCh38 from Ensembl release 91)
- Reference GFF3 exon annotations (truth set)
- Transcriptome FASTA from the same release
- wgsim simulation parameters (read length, coverage, error rate)

## Outputs

- Simulated paired-end WGS FASTA/FASTQ reads
- Baited Bloom filter (.bf) at 4 GB and 60 GB settings with recorded BF FPR
- Predicted exon/splice graph output in GFA1 format
- Benchmark metrics CSV (exfi_memory_benchmark_metrics.csv: BF FPR, precision, recall, F1)

## How to apply

Download reference genomes, transcriptomes, and GFF3 annotations from Ensembl release 91 (assemblies GRCz10 for D. rerio and GRCh38 for H. sapiens) and simulate WGS reads with wgsim as described in the methods. Feed the simulated reads plus the transcriptome to EXFI's build_baited_bloom_filter with k-mer length fixed at 25 bp, varying Bloom filter sizes (4 GB and 60 GB) to measure how memory affects the tunable BF FPR. Then run build_splice_graph with k = 25 on each transcriptome plus its baited Bloom filter to predict exons and splice graphs. Rationale: because only 4.24% (human) and 5.68% (zebrafish) of the bases are exons, the simulation lets classification metrics be computed against a complete truth set that real WGS data cannot provide.

## Related tools

- **wgsim** (Simulates the synthetic WGS reads from the zebrafish and human reference genomes used as EXFI input)
- **EXFI** (Consumes the simulated WGS reads to build baited Bloom filters and predict exons/splice graphs via build_baited_bloom_filter and build_splice_graph) — https://github.com/jlanga/exfi
- **bedtools** (Compares predicted exons to the reference GFF3 (converted to transcript-based BED) with at least 95% mutual overlap to compute precision/recall)
- **Pandas** (Assembles the BF FPR and classification metrics from all runs (2 species x 2 memory settings) into the benchmark CSV)

## Evaluation signals

- Reproduced BF FPR falls in the reported 8.1%–0.4% range when Bloom filter memory increases from 4 GB to 60 GB
- Zebrafish exon precision/recall are close to the reported values: 95.8%/93.8% at 4 GB and 96.6%/94.6% at 60 GB
- Read filtering reduces BF FPR by 68%–75% relative to unfiltered runs while slightly improving classification metrics
- Precision, recall, and F1 are computed only over exon pairs with mutual overlap of at least 95% via BEDTools intersect
- All four run combinations (2 species x 2 memory settings) are present in exfi_memory_benchmark_metrics.csv

## Limitations

- Bloom filters return no false negatives but produce false positives with a tunable BF FPR; low memory (4 GB) yields ~8.1% FPR, yet exon precision/recall improve only marginally at high memory (60 GB)
- Human/zebrafish exonic content is tiny (4.24% and 5.68% of bases), so benchmark results are optimistic relative to real transcriptome quality cases
- Full WGS read accession numbers for real datasets were deferred to Table S1 and are not in the main text

## Evidence

- [methods] Ensembl datasets used for simulation: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38"
- [methods] Bloom filter FPR property: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] Effect of read filtering on BF FPR: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] Exonic fraction motivating synthetic benchmarks: "In the human and zebrafish genomes, only 4.24% and 5.68% of the bases are exons, respectively (Table 1)"
- [intro] Core pipeline components consuming the simulated reads: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"

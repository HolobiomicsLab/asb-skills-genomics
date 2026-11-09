---
name: short-read-quality-filtering
description: Use when preparing Illumina/short-read data as input to yak k-mer counting
  for NextPolish2 polishing or Merqury/yak QV evaluation, and reads need adapter removal,
  quality trimming, and filtering.
when_to_use_negative:
- Input is already a polished assembly or long-read HiFi FASTQ — fastp QC here targets
  short reads only.
- Short-read coverage is well below the recommended >=60X, since NextPolish2 performance
  relies heavily on short reads and singletons would be discarded by yak -b 37.
- The goal is polishing without k-mer evaluation (e.g., only Racon-based long-read
  polishing), where short-read QC adds no value.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: 'Quality control and filtering of short reads: adapter removal, quality trimming,
    read filtering before yak k-mer dataset construction'
- name: yak
  role: Builds k-mer datasets (k=21, k=31, -b 37) from filtered short reads for polishing
    and QV evaluation
  repo: https://github.com/lh3/yak
- name: NextPolish2
  role: Consumes the filtered reads (via yak datasets) to correct HiFi assemblies
    without introducing switch errors
  repo: https://github.com/Nextomics/NextPolish2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:d3561bbc90d76340bd46e2c408fbcd02d7d5fe5fff67d20bc06ccd2a30283dfb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/yak
  tools:
  - Merqury
  - NextPolish2
  - Racon
  - fastp
  - hifiasm
  - meryl
  - samtools
  - winnowmap
  - yak
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# short-read-quality-filtering

## Summary

Quality-control short-read data with fastp prior to k-mer dataset construction (yak) for genome polishing and QV evaluation, applying adapter removal, quality trimming, and read filtering so that polishing and Merqury-style metrics rely on reliable k-mers.

## When to use

Use when preparing Illumina/short-read data as input to yak k-mer counting for NextPolish2 polishing or Merqury/yak QV evaluation, and reads need adapter removal, quality trimming, and filtering.

## When NOT to use

- Input is already a polished assembly or long-read HiFi FASTQ — fastp QC here targets short reads only.
- Short-read coverage is well below the recommended >=60X, since NextPolish2 performance relies heavily on short reads and singletons would be discarded by yak -b 37.
- The goal is polishing without k-mer evaluation (e.g., only Racon-based long-read polishing), where short-read QC adds no value.

## Inputs

- Raw paired-end short-read FASTQ files (recommended >=60X coverage)
- fastp parameters: -q 20, adapter removal, -5 -3 -f 5 -F 5 -t 5 -T 5

## Outputs

- Quality-filtered short-read FASTQ files
- Cleaned reads ready for yak k-mer counting (k21.yak, k31.yak datasets)

## How to apply

Run fastp on the raw short reads with quality threshold -q 20, adapter removal enabled, 5' and 3' trimming of 5 bases (-5 -3 -f 5 -F 5 -t 5 -T 5), and no length-cutoff on N content (-n 0). The rationale is that NextPolish2 relies heavily on high-coverage short reads to judge whether a k-mer contains errors, so residual adapters and low-quality bases would corrupt the k-mer spectra. Feed the filtered reads into yak k-mer dataset construction with k=21 and k=31 using -b 37 to discard singletons, which are then used both for polishing (asm.np2.fa output) and downstream yak/Merqury QV and switch-error evaluation.

## Related tools

- **fastp** (Quality control and filtering of short reads: adapter removal, quality trimming, read filtering before yak k-mer dataset construction)
- **yak** (Builds k-mer datasets (k=21, k=31, -b 37) from filtered short reads for polishing and QV evaluation) — https://github.com/lh3/yak
- **NextPolish2** (Consumes the filtered reads (via yak datasets) to correct HiFi assemblies without introducing switch errors) — https://github.com/Nextomics/NextPolish2

## Examples

```
fastp -5 -3 -n 0 -f 5 -F 5 -t 5 -T 5 -q 20 -i sr_1.fq.gz -I sr_2.fq.gz -o sr_1.clean.fq.gz -O sr_2.clean.fq.gz
```

## Evaluation signals

- fastp report shows adapters removed and Q20 filtering applied (-q 20) with reasonable post-filter read counts retained
- yak count with -b 37 completes on the filtered reads producing k21.yak and k31.yak datasets without errors
- Downstream NextPolish2 run produces asm.np2.fa and improves QV (e.g., Col-XJTU 58.03 -> 64.26)
- Merqury/yak QV and switch-error evaluation on polished assembly matches or exceeds reported benchmark values (e.g., simulated A. thaliana switch error 1.99 -> 0.35 ‱)

## Limitations

- Performance of NextPolish2 relies heavily on short reads; biases inherent to short-read sequencing can lead to uncorrected errors, so high coverage is recommended
- No specific coverage threshold was stated beyond the recommended >=60X short reads for yak dataset preparation
- Singleton k-mers are discarded (-b 37), so very low-coverage datasets lose information

## Evidence

- [methods] fastp quality filtering: "# Quality control and filtering.
# fastp -5 -3 -n 0 -f 5 -F 5 -t 5 -T 5 -q 20"
- [discussion] short-read dependence: "The performance of NextPolish2 relies heavily on short reads to check whether a K-mer contains errors."
- [methods] k-mer dataset preparation: "2. Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads)."
- [methods] filter step details: "quality filtering steps (fastp) such as adapter removal, global or quality trimming, and read filtering are essential for short reads."

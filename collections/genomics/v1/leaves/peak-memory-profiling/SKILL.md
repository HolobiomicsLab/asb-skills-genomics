---
name: peak-memory-profiling
description: Use when reproducing or validating computational runtime and memory benchmarks
  of Z-Calling base calling or read classification on CCS BAM datasets with kinetic
  tags.
when_to_use_negative:
- The goal is A/Z calling accuracy (AUC/F1) rather than computational benchmarking
  — use the base-calling evaluation skill instead
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp), so they cannot pass filter_bam.py
  and timings would not reflect the benchmarked pipeline
- Profiling GPU or multi-node performance — Z-Calling is CPU-only (no GPU acceleration)
  on a standard multi-core CPU
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: Z-Calling
  role: Tool whose base-calling (z-calling-base) and read-classification (z-calling-read
    predict) modules are being profiled for runtime and peak memory
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source sequencing data format (BAM with --keep-kinetics PW/IPD signals) used
    as the benchmark input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:965062a70453ad5cb3f7bd9a713baddadbd81845098a14e6ddb105669237be64
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# peak-memory-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profile wall-clock time and peak RSS memory of the Z-Calling A/Z base-calling pipeline (BAM filtering + k11-mixed-AZ MLP, and k21-full-ZA + SVM read classification) on PacBio HiFi BAMs, normalizing throughput per 10k reads and per Gb against the paper's benchmark (~31 GB peak memory).

## When to use

Use when reproducing or validating computational runtime and memory benchmarks of Z-Calling base calling or read classification on CCS BAM datasets with kinetic tags.

## When NOT to use

- The goal is A/Z calling accuracy (AUC/F1) rather than computational benchmarking — use the base-calling evaluation skill instead
- Input reads lack PacBio kinetic tags (fi, ri, fp, rp), so they cannot pass filter_bam.py and timings would not reflect the benchmarked pipeline
- Profiling GPU or multi-node performance — Z-Calling is CPU-only (no GPU acceleration) on a standard multi-core CPU

## Inputs

- PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp) from benchmark datasets
- Pre-trained Z-Calling models: model/k11-mixed-AZ/scripted_m11.pth and model/k21-full-ZA/scripted_m21.pth
- SVM read classifier model/model/svm/k21ReadClassifier

## Outputs

- Wall-clock runtime per 10k reads and per Gb for each pipeline variant and thread configuration (-t 1, -t 16)
- Peak RSS memory measurements (expected ~30.92 GB during base calling)
- Filtered BAMs and Z-call output BAMs from the timed pipeline runs

## How to apply

First run py/filter_bam.py on each benchmark BAM so reads carry the kinetic tags (fi, ri, fp, rp), with Median_PW/Median_IPD within [0.3, 99th percentile] and fn >= 3, rn >= 3 passes. Then time build/z-calling-base with the k11-mixed-AZ model (scripted_m11.pth, -k 11) on the filtered BAMs under -t 1 and -t 16 thread configurations, capturing wall-clock time and peak RSS (e.g. with /usr/bin/time -v). Repeat the timing for z-calling-base with the k21-full-ZA model (scripted_m21.pth, -k 21) followed by build/z-calling-read predict (min_len 500, intervals 100) under both thread configurations. Normalize measured times to per-10k-reads and per-Gb throughput for each pipeline variant and thread configuration, then compare against the reported 37.48 ± 24.58 s per 10k reads (313.20 ± 98.72 s per Gb) and ~31 GB peak memory. The rationale is that benchmarks are only reproducible when the same filtering, model, and thread settings are used, since runtime depends heavily on the k-mer size (11 vs 21) and thread count.

## Related tools

- **Z-Calling** (Tool whose base-calling (z-calling-base) and read-classification (z-calling-read predict) modules are being profiled for runtime and peak memory) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source sequencing data format (BAM with --keep-kinetics PW/IPD signals) used as the benchmark input)

## Examples

```
/usr/bin/time -v build/z-calling-base -k 11 -t 16 filtered.bam model/k11-mixed-AZ/scripted_m11.pth output.bam
```

## Evaluation signals

- Measured peak RSS during z-calling-base falls within tolerance of ~30.92 GB (README: approx. 31 GB during base calling, ≥ 32 GB RAM recommended)
- Wall-clock time per 10k reads falls within tolerance of 37.48 ± 24.58 s for the k11-mixed-AZ base-calling module
- Per-Gb throughput is comparable to the reported 313.20 ± 98.72 s per Gb
- Filtering applied before timing retains only reads with fi/ri/fp/rp tags, Median_PW/Median_IPD in [0.3, 99th percentile], and fn >= 3, rn >= 3
- Both -t 1 and -t 16 configurations are measured and throughput differences are consistent with thread scaling

## Limitations

- Peak memory (~31 GB) means at least 32 GB RAM is required; smaller machines cannot run the base-calling benchmark
- Runtime figures are specific to the benchmark hardware (AMD EPYC 7402 24-Core Processor) and may differ on other CPUs
- SVM read classifier requires min_len >= 500 and intervals must be exactly 100, so timing of read classification depends on read-length distribution of the dataset

## Evidence

- [other] the A/Z base-calling step (BAM filtering + k11-mixed-AZ MLP) averaged 37.48 ± 24.58 s per 10k reads (313.20 ± 98.72 s per Gb data), with peak memory usage of 30.92 Gb: [evidence span withheld — anchor status: not_found]

---
name: ml-basecalling-inference-execution
description: Use when PacBio CCS/HiFi BAMs with kinetic tags (fi, ri, fp, rp) need
  single-nucleotide A/Z base calling or whole-read dZ-DNA classification with pre-trained
  MLP/SVM models, or when profiling the compute cost of those inference steps.
when_to_use_negative:
- The sequencing data is not PacBio CCS/HiFi with preserved kinetics (e.g. ONT or
  Illumina reads lack the PW/IPD signals the models require)
- The BAM has not been filtered for kinetic tags and pass counts — running inference
  on raw unfiltered reads violates the documented pre-processing requirements
- Reads are shorter than 500 bp or have fewer than 100 A+Z bases, which the read-classification
  step excludes by design
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Provides the z-calling-base MLP inference (k11-mixed-AZ and k21-full-ZA models),
    z-calling-read SVM classifier, and py/filter_bam.py pre-filtering used to execute
    base calling and read classification
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Produces the kinetic-carrying HiFi reads (--keep-kinetics) that are the direct
    input to the inference models
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:6c8a70eb7613d1ce709004faa18ecda1fed2ec96701bb8765fbcc0f86e901a72
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# ml-basecalling-inference-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes Z-Calling's machine-learning inference modules (MLP base calling and SVM read classification) on PacBio HiFi/CCS BAMs to call 2,6-diaminopurine (Z) bases and classify dZ-DNA reads, including benchmarking runtime and memory of the inference step.

## When to use

Use when PacBio CCS/HiFi BAMs with kinetic tags (fi, ri, fp, rp) need single-nucleotide A/Z base calling or whole-read dZ-DNA classification with pre-trained MLP/SVM models, or when profiling the compute cost of those inference steps.

## When NOT to use

- The sequencing data is not PacBio CCS/HiFi with preserved kinetics (e.g. ONT or Illumina reads lack the PW/IPD signals the models require)
- The BAM has not been filtered for kinetic tags and pass counts — running inference on raw unfiltered reads violates the documented pre-processing requirements
- Reads are shorter than 500 bp or have fewer than 100 A+Z bases, which the read-classification step excludes by design

## Inputs

- PacBio CCS/HiFi BAM files with kinetic tags (fi, ri, fp, rp; Median_PW, Median_IPD, fn, rn)
- Filtered BAM produced by py/filter_bam.py (mapped or unmapped)
- Pre-trained model files: model/k11-mixed-AZ/scripted_m11.pth, model/k21-full-ZA/scripted_m21.pth, model/svm/k21ReadClassifier
- Optional reference FASTA for alignment and downstream z-bam2txt/zfreq steps

## Outputs

- Output BAM annotated with MM/ML modification tags and ZP tags recording per-base Z probability
- Tab-separated read classification TSV (read name; + = dZ-DNA, - = canonical, otherwise unclassified)
- Benchmark timings (wall-clock per 10k reads and per Gb) and peak RSS measurements for each model/thread configuration

## How to apply

First run py/filter_bam.py on the raw BAM so only reads with kinetic tags (fi, ri, fp, rp), a Median_PW/Median_IPD ratio within [0.3, 99th percentile], and at least 3 forward and 3 reverse passes (fn >= 3, rn >= 3) are retained — the MLP models depend on these kinetics. Build the CPU-only PyTorch 2.2.2 environment via conda/mamba and CMake, then run build/z-calling-base with -k 11 against model/k11-mixed-AZ/scripted_m11.pth for site-level Z-probabilities (ZP tag added to A/T bases), or -k 21 against model/k21-full-ZA/scripted_m21.pth to score reads for read-level dZ detection; the -t flag controls threads (default 16). For molecule classification, feed the k21 output to build/z-calling-read predict with min_len >= 500 and exactly 100 intervals using model/svm/k21ReadClassifier, producing a TSV of read names and +/− classifications. To benchmark, time z-calling-base under -t 1 and -t 16 on filtered BAMs with /usr/bin/time, recording wall-clock time and peak RSS, and normalize to per-10k-reads and per-Gb throughput (reference: 37.48 ± 24.58 s per 10k reads, ~30.92 Gb peak memory).

## Related tools

- **Z-Calling** (Provides the z-calling-base MLP inference (k11-mixed-AZ and k21-full-ZA models), z-calling-read SVM classifier, and py/filter_bam.py pre-filtering used to execute base calling and read classification) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Produces the kinetic-carrying HiFi reads (--keep-kinetics) that are the direct input to the inference models)

## Examples

```
build/z-calling-base -k 11 -t 16 FilteredBAM.bam model/k11-mixed-AZ/scripted_m11.pth Output_BAM.bam
```

## Evaluation signals

- Output BAM contains ZP tags with Z probabilities recorded for each A/T base when the k11-mixed-AZ model is used
- Read classification TSV contains only +, -, or unclassified labels, with reads shorter than min_len=500 excluded
- Wall-clock time per 10k reads falls near the reported 37.48 ± 24.58 s (313.20 ± 98.72 s per Gb) and peak memory near ~30.92 Gb, within agreed tolerance
- Base-calling module performance is consistent with reported AUCs of 0.942–0.952 and F1 scores of 0.85–0.92 when evaluated on labeled benchmark datasets
- Environment check passes: python -c "import torch; print(torch.__version__)" prints 2.2.2+cpu

## Limitations

- Peak memory during base calling is ~31 GB, so ≥ 32 GB RAM is recommended; the tool is CPU-only (no GPU acceleration)
- The z-calling-read intervals parameter must be exactly 100 and min_len >= 500 for the current SVM model, and the SVM is optimized for fully substituted dZ-genomes, missing hybrid reads with diluted kinetic signatures
- The subtle Z-induced kinetic shift (0.5–1.5-fold PW change) overlaps substantially with canonical Adenine variance, limiting quantification in hybrid samples

## Evidence

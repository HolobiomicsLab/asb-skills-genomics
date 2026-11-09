---
name: quantization-distribution-analysis
description: Use when reimplementing, testing, or validating RawHash2-style adaptive
  quantization of raw nanopore signals (FAST5/POD5/SLOW5), or when verifying that
  quantized-value buckets are dynamically sized from the normalized signal distribution
  rather than fixed-width.
when_to_use_negative:
- The goal is uniform fixed-width quantization (RawHash-style); this skill specifically
  validates the adaptive, distribution-dependent bucket layout, and a uniform layout
  fails the check
- Input signals are already sequenced basecalls or basecalled reads (FASTQ/FASTA)
  rather than raw current-level signal data
- The task concerns later RawHash2 stages (hash-based seeding, chaining, DTW signal
  alignment) rather than the quantization step itself
edam_topics:
- http://edamontology.org/topic_3678
tools:
- name: RawHash2
  role: Reference implementation of the adaptive quantization mechanism (Equation
    1) whose bucket layout is reconstructed and validated
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Source of the R9.4 segmentation parameters used to normalize raw signal values
    before quantization
  repo: https://github.com/haowenz/sigmap
- name: RawHash
  role: Baseline mapper whose (non-adaptive) quantization serves as the comparison
    point for the adaptive bucket layout
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:3ac82b3173eaa486c1f78ed663bf54626c3d3ed8b65b2f6902a1bb6b7b5e701a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3678
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# quantization-distribution-analysis

## Summary

Reconstructs and validates RawHash2's adaptive quantization of normalized nanopore signal values, where bucket sizes are dynamically arranged from the normalized signal distribution (fine range bounded by fmin/fmax with resolution fr, coarser buckets outside it with cr=(1−fr)×0.5, per Equation 1). This quantization underpins hash-based seeding and drove RawHash2's accuracy and performance gains over RawHash.

## When to use

Use when reimplementing, testing, or validating RawHash2-style adaptive quantization of raw nanopore signals (FAST5/POD5/SLOW5), or when verifying that quantized-value buckets are dynamically sized from the normalized signal distribution rather than fixed-width.

## When NOT to use

- The goal is uniform fixed-width quantization (RawHash-style); this skill specifically validates the adaptive, distribution-dependent bucket layout, and a uniform layout fails the check
- Input signals are already sequenced basecalls or basecalled reads (FASTQ/FASTA) rather than raw current-level signal data
- The task concerns later RawHash2 stages (hash-based seeding, chaining, DTW signal alignment) rather than the quantization step itself

## Inputs

- Raw nanopore signal values from FAST5 files (e.g. test/data/d1_sars-cov-2_r94/fast5_files), also readable from POD5/SLOW5/BLOW5
- Fine-range parameters: fmin, fmax, and fine resolution fr (coarse resolution cr = (1−fr)×0.5)
- R9.4 segmentation/normalization parameters from Sigmap

## Outputs

- quantized_values.csv containing (s, q(s)) pairs of normalized signal values and their quantized bucket values
- Histogram of quantized-value occupancy across buckets revealing the bucket-size layout

## How to apply

Read raw nanopore signal values from packaged FAST5 files (e.g. test/data/d1_sars-cov-2_r94/fast5_files) with h5py and normalize them using the R9.4 segmentation parameters RawHash2 adopts from Sigmap. Implement a standalone quantize() function taking the normalized signal values, fine-range boundaries fmin and fmax, and fine resolution fr; within [fmin, fmax] assign high-resolution buckets of width fr, and outside that range arrange coarser bucket sizes with cr=(1−fr)×0.5 according to the normalized distribution of the signal values, implementing Equation (1). Map every normalized signal value s to its quantized value q(s), write the (s, q(s)) pairs to quantized_values.csv, and compute the histogram of quantized-value occupancy across buckets to expose the bucket-size layout. Rationale: RawHash2's new quantization mechanism dynamically arranges bucket sizes from the normalized signal distribution, which the paper reports provides significant improvements in both accuracy and performance over RawHash.

## Related tools

- **RawHash2** (Reference implementation of the adaptive quantization mechanism (Equation 1) whose bucket layout is reconstructed and validated) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Source of the R9.4 segmentation parameters used to normalize raw signal values before quantization) — https://github.com/haowenz/sigmap
- **RawHash** (Baseline mapper whose (non-adaptive) quantization serves as the comparison point for the adaptive bucket layout) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
python quantize.py --fast5 test/data/d1_sars-cov-2_r94/fast5_files --fmin -1.0 --fmax 1.0 --fr 0.8 --out quantized_values.csv
```

## Evaluation signals

- The quantized-value distribution across buckets is nonuniform: dense signal regions receive smaller (finer) buckets and sparse regions receive larger (coarser) buckets
- Within [fmin, fmax], buckets have width fr; outside the fine range, coarse bucket widths equal cr = (1−fr)×0.5 as per Equation (1)
- Every normalized signal value s maps to exactly one quantized value q(s), and the (s, q(s)) pairs in quantized_values.csv cover the full normalized signal range
- The bucket-occupancy histogram matches the normalized signal distribution layout expected by RawHash2, and a uniform fixed-width bucket layout fails this check
- Normalization reproduces the R9.4 segmentation parameters used by RawHash2 (from Sigmap) on the SARS-CoV-2 R9.4 test FAST5 data

## Limitations

- Bucket-size arrangement depends on the normalized distribution of each signal, so results are read-specific and cannot be compared across reads without re-quantization
- The fine/coarse split quality depends on correct fmin/fmax/fr settings; incorrect fine-range boundaries will misplace high- and low-resolution buckets
- Quantization quality directly affects RawHash2 accuracy and performance, so errors here propagate to hash-based seeding and mapping

## Evidence

- [methods] Dynamic bucket sizes from signal distribution: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [methods] Significant accuracy and performance improvements: "**This provides significant improvements in both accuracy and performance.**"
- [methods] R9.4 normalization parameters: "RawHash2 uses the R9.4 segmentation parameters from [Sigmap](https://github.com/haowenz/sigmap)."
- [readme] Hash-based real-time signal mapping context: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."

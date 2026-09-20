---
name: fine-coarse-range-bucket-partitioning
description: Use when implementing or verifying RawHash2-style adaptive quantization
  of normalized nanopore signal values into quantized buckets (q(s)) before hash-based
  seeding and indexing.
when_to_use_negative:
- Input signals are basecalled sequences (FASTQ/FASTA) rather than raw signal values
  in FAST5/POD5/SLOW5/BLOW5 format
- A uniform fixed-width bucket layout is required (e.g. replicating original RawHash
  quantization rather than RawHash2's adaptive scheme)
- The goal is R10.4-specific signal processing without normalization parameters adapted
  for that flow cell version
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash2
  role: Source implementation of the adaptive quantization mechanism with dynamically
    arranged bucket sizes; the quantize() function reproduces its Equation (1)
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Predecessor tool whose quantization scheme provides the baseline against which
    the improved fine/coarse bucket layout is compared
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Provides the R9.4 segmentation parameters used to normalize the signal values
    before quantization
  repo: https://github.com/haowenz/sigmap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:35e7f93d1faf99eaf9f9ce3810709918f772da9d4eccc5a6b0b09716c601af1c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  - Sigmap
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# fine-coarse-range-bucket-partitioning

## Summary

Reconstructs RawHash2's adaptive quantization mechanism, which partitions the normalized nanopore signal range into a bifurcated fine range [fmin, fmax] with high resolution fr and coarse ranges with resolution cr=(1−fr)×0.5, so that dense signal regions receive finer buckets per Equation (1).

## When to use

Use when implementing or verifying RawHash2-style adaptive quantization of normalized nanopore signal values into quantized buckets (q(s)) before hash-based seeding and indexing.

## When NOT to use

- Input signals are basecalled sequences (FASTQ/FASTA) rather than raw signal values in FAST5/POD5/SLOW5/BLOW5 format
- A uniform fixed-width bucket layout is required (e.g. replicating original RawHash quantization rather than RawHash2's adaptive scheme)
- The goal is R10.4-specific signal processing without normalization parameters adapted for that flow cell version

## Inputs

- Raw nanopore signal values from FAST5 files (e.g. test/data/d1_sars-cov-2_r94/fast5_files)
- Normalized signal values using R9.4 segmentation parameters from Sigmap
- Fine-range boundaries fmin and fmax and fine resolution fr

## Outputs

- quantized_values.csv containing (s, q(s)) pairs
- Histogram of quantized-value occupancy across buckets showing the bucket-size layout

## How to apply

Read raw nanopore signal values from packaged test FAST5 files (e.g. test/data/d1_sars-cov-2_r94/fast5_files) with h5py and normalize them using the R9.4 segmentation parameters RawHash2 adopts from Sigmap. Implement a standalone quantize() function taking the normalized signal, fine-range boundaries fmin and fmax, and fine resolution fr; inside [fmin, fmax] assign buckets of width fr, while outside the fine range use the coarse resolution cr=(1−fr)×0.5, assigning q(s) per Equation (1) so bucket sizes are dynamically arranged according to the normalized distribution of the signal values. Map every normalized value s to q(s), write the (s, q(s)) pairs to quantized_values.csv, and compute a histogram of quantized-value occupancy across buckets to expose the bucket-size layout. The rationale is that dynamically sized buckets—finer where signals are dense, coarser where sparse—improve both accuracy and performance of hash-based seed generation compared to fixed-width quantization.

## Related tools

- **RawHash2** (Source implementation of the adaptive quantization mechanism with dynamically arranged bucket sizes; the quantize() function reproduces its Equation (1)) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Predecessor tool whose quantization scheme provides the baseline against which the improved fine/coarse bucket layout is compared) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Provides the R9.4 segmentation parameters used to normalize the signal values before quantization) — https://github.com/haowenz/sigmap

## Examples

```
python quantize.py --fast5 test/data/d1_sars-cov-2_r94/fast5_files --fmin 0.2 --fmax 0.8 --fr 0.1 --out quantized_values.csv
```

## Evaluation signals

- The quantized-value distribution across buckets is nonuniform: dense signal regions receive smaller (finer) buckets and sparse regions receive larger (coarser) buckets
- A uniform fixed-width bucket layout fails the validation check — bucket sizes must be dynamically arranged from the normalized signal distribution
- The coarse resolution equals cr=(1−fr)×0.5 as specified by Equation (1) of RawHash2
- Every normalized signal value s maps to exactly one quantized value q(s), recorded in quantized_values.csv
- Normalization uses the R9.4 segmentation parameters from Sigmap, matching RawHash2's preprocessing

## Limitations

- Fine-range boundaries fmin, fmax and resolution fr must be chosen appropriately; the workflow does not prescribe values for all datasets
- The dynamic bucket layout is derived from the normalized distribution of the signal values, so it is sensitive to the normalization step (R9.4 parameters from Sigmap)
- Fixed-width quantization cannot capture the adaptive fine/coarse structure and will fail validation

## Evidence

- [methods] The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values.: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [methods] RawHash2 uses the R9.4 segmentation parameters from Sigmap.: "RawHash2 uses the R9.4 segmentation parameters from [Sigmap](https://github.com/haowenz/sigmap)."
- [intro] RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time.: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
- [methods] RawHash can be used to map reads from FAST5, POD5, SLOW5, or BLOW5 files to a reference genome in sequence format.: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."

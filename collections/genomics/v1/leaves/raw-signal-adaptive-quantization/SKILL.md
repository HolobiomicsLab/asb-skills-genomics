---
name: raw-signal-adaptive-quantization
description: 'Use when building or reimplementing the signal-representation step of
  RawHash2/Rawsamble: normalized raw nanopore signal values (FAST5/POD5/SLOW5) must
  be assigned to quantized buckets before hash-based seeding, and bucket sizes should
  follow the signal''s normalized distribution.'
when_to_use_negative:
- Input signals are basecalled sequences (FASTQ/FASTA) rather than raw electrical
  current values — quantization operates on raw signals.
- A uniform (fixed-width) bucket layout is required, since the validation explicitly
  rejects uniform layouts as inconsistent with RawHash2's adaptive mechanism.
- Signal values have not been normalized first — Equation (1) operates on normalized
  values using R9.4 segmentation parameters.
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash2
  role: Reference implementation whose adaptive quantization mechanism (Equation 1,
    fine/coarse ranges) is being reconstructed and validated
  repo: https://github.com/CMU-SAFARI/RawHash
- name: RawHash
  role: Prior-version baseline whose quantization lacked dynamic bucket sizing; used
    for comparison of quantized-value distributions
  repo: https://github.com/CMU-SAFARI/RawHash
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae478
    title: 'RawHash2: mapping raw nanopore signals using hash-based seeding and adaptive
      quantization'
  source_content_hash: sha256:61586acedf0fed1fa15a08b0e66cb6566eba8dcca887dd607921a6987eb650d3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - RawHash2
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# raw-signal-adaptive-quantization

## Summary

Implements RawHash2's adaptive quantization, which maps normalized nanopore signal values to quantized buckets using a bifurcated fine/coarse range scheme (fine range bounded by fmin/fmax at resolution fr, coarse buckets sized by cr=(1−fr)×0.5), so denser signal regions receive finer buckets. This improves hashing accuracy and throughput over RawHash's fixed quantization.

## When to use

Use when building or reimplementing the signal-representation step of RawHash2/Rawsamble: normalized raw nanopore signal values (FAST5/POD5/SLOW5) must be assigned to quantized buckets before hash-based seeding, and bucket sizes should follow the signal's normalized distribution.

## When NOT to use

- Input signals are basecalled sequences (FASTQ/FASTA) rather than raw electrical current values — quantization operates on raw signals.
- A uniform (fixed-width) bucket layout is required, since the validation explicitly rejects uniform layouts as inconsistent with RawHash2's adaptive mechanism.
- Signal values have not been normalized first — Equation (1) operates on normalized values using R9.4 segmentation parameters.

## Inputs

- Raw nanopore signal values from FAST5 files (e.g. RawHash2's test/data/d1_sars-cov-2_r94/fast5_files)
- Normalized signal values (R9.4 segmentation parameters from Sigmap)
- Quantization parameters: fine-range boundaries fmin and fmax, fine resolution fr

## Outputs

- quantized_values.csv with (s, q(s)) pairs mapping each normalized signal value to its bucket
- Histogram of quantized-value occupancy across buckets showing the bucket-size layout

## How to apply

Read raw signal values from packaged test FAST5 files (e.g. test/data/d1_sars-cov-2_r94/fast5_files) with h5py and normalize them using the R9.4 segmentation parameters from Sigmap. Implement a standalone quantize() function taking the normalized values plus fine-range boundaries fmin, fmax and fine resolution fr. Inside [fmin, fmax], assign buckets of width fr; outside that range, use coarse intervals with cr=(1−fr)×0.5, so the bucket layout is dynamically arranged from the normalized distribution of the signal values per Equation (1) — this dynamic arrangement is what RawHash2's new quantization mechanism adds and what 'provides significant improvements in both accuracy and performance'. Map every normalized signal value s to its quantized value q(s), write (s, q(s)) pairs to quantized_values.csv, and compute a histogram of quantized-value occupancy to expose the bucket layout.

## Related tools

- **RawHash2** (Reference implementation whose adaptive quantization mechanism (Equation 1, fine/coarse ranges) is being reconstructed and validated) — https://github.com/CMU-SAFARI/RawHash
- **RawHash** (Prior-version baseline whose quantization lacked dynamic bucket sizing; used for comparison of quantized-value distributions) — https://github.com/CMU-SAFARI/RawHash

## Examples

```
import h5py; f = h5py.File('test/data/d1_sars-cov-2_r94/fast5_files/read1.fast5','r'); raw = f['Raw/Reads/Read_000/Signal'][:]; q = quantize(normalize(raw), fmin, fmax, fr)
```

## Evaluation signals

- The quantized-value distribution across buckets is nonuniform: dense signal regions receive smaller buckets and sparse regions receive larger buckets.
- A uniform (fixed-width) bucket layout fails the validation check — it indicates the dynamic arrangement from the normalized distribution was not implemented.
- Coarse-range resolution equals cr=(1−fr)×0.5 and fine buckets within [fmin, fmax] have width fr.
- Every normalized signal value s has a corresponding q(s) written to quantized_values.csv with no unassigned values.
- The bucket-occupancy histogram reflects the bifurcated fine/coarse bucket-size layout expected from Equation (1).

## Limitations

- The scheme is tied to the R9.4 segmentation parameters from Sigmap; other chemistries (e.g. R10.4, which RawHash2 adds support for) may require different normalization parameters.
- Quantization quality depends on correct normalization; unnormalized or mis-segmented raw FAST5 signal will distort the fine/coarse bucket layout.
- FAST5 files may be VBZ-compressed, requiring the ONT VBZ HDF5 plugin for h5py to read signal values.

## Evidence

- [methods] quantization of signal values: "The new quantization mechanism dynamically arranges the bucket sizes that each signal value is quantized depending on the normalized distribution of the signal values."
- [methods] quantization accuracy/performance: "**This provides significant improvements in both accuracy and performance.**"
- [methods] normalization parameters: "RawHash2 uses the R9.4 segmentation parameters from [Sigmap](https://github.com/haowenz/sigmap)."
- [methods] input file formats: "RawHash can be used to map reads from **FAST5, POD5, SLOW5, or BLOW5** files to a reference genome in sequence format."

---
name: numerical-precision-error-analysis
description: Use when porting a floating-point algorithm to fixed-point/low-precision
  arithmetic and you need to quantify how a scaling-factor or quantization parameter
  affects output agreement with a float32 baseline.
when_to_use_negative:
- The algorithm is still running in floating point and no quantization parameter needs
  tuning
- Output agreement cannot be reduced to an exact-match or tolerance comparison against
  a float32 baseline (e.g., stochastic or non-deterministic outputs)
- The numeric range of the data exceeds 16-bit fixed-point representation even at
  moderate scaling factors (overflow dominates, as above factor 128 in HARU's sDTW)
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Sigfish-HARU
  role: Runs the float32 sDTW squiggle-mapping baseline and hosts the fixed-point
    variants whose scaling factor is swept
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: FPGA sDTW accelerator using 16-bit fixed-point arithmetic; the deployment
    target whose scaling factor is selected
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:0435dc0c8b76056fbde7808cc5f7a56754093560f8c95a3103d0207aa1c6ad36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Numerical precision error analysis

## Summary

Sweeps a numerical-precision parameter (fixed-point scaling factor) against float32 ground truth to characterize accuracy degradation, used by HARU to select a 16-bit fixed-point scaling factor of 32 for sDTW squiggle mapping (~99% accuracy).

## When to use

Use when porting a floating-point algorithm to fixed-point/low-precision arithmetic and you need to quantify how a scaling-factor or quantization parameter affects output agreement with a float32 baseline.

## When NOT to use

- The algorithm is still running in floating point and no quantization parameter needs tuning
- Output agreement cannot be reduced to an exact-match or tolerance comparison against a float32 baseline (e.g., stochastic or non-deterministic outputs)
- The numeric range of the data exceeds 16-bit fixed-point representation even at moderate scaling factors (overflow dominates, as above factor 128 in HARU's sDTW)

## Inputs

- 32-bit floating-point sDTW baseline mapping positions (from sigfish dtw against the reference genome)
- Nanopore raw signal reads in BLOW5 format (e.g., reads_0_0.blow5)
- Reference genome FASTA (nCoV-2019.reference.fasta)
- Integer scaling factor candidates for 16-bit fixed-point arithmetic

## Outputs

- accuracy_vs_scaling_factor.csv with rows of scaling factor, mapping accuracy, and overflow/degraded-accuracy flag

## How to apply

First run the float32 reference implementation — here sigfish dtw mapping nanopore BLOW5 reads (reads_0_0.blow5) against the SARS-CoV-2 reference nCoV-2019.reference.fasta — to capture baseline mapping positions for every read. Then recompute outputs with the fixed-point variant at a geometric sweep of integer scaling factors (2, 4, 8, 16, 32, 64, 128 for 16-bit arithmetic). Score each factor by the fraction of reads whose mapping positions exactly match the float32 baseline. Expect low scaling factors to under-resolve normalized signal values (~80% at factor 2), convergence toward 100% as the factor grows, and overflow-driven degradation above 128, justifying factor 32 (~99%) as the operating point.

## Related tools

- **Sigfish-HARU** (Runs the float32 sDTW squiggle-mapping baseline and hosts the fixed-point variants whose scaling factor is swept) — https://github.com/beebdev/sigfish-haru
- **HARU** (FPGA sDTW accelerator using 16-bit fixed-point arithmetic; the deployment target whose scaling factor is selected) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- accuracy_vs_scaling_factor.csv contains exactly 7 rows (factors 2, 4, 8, 16, 32, 64, 128)
- Accuracy at scaling factor 2 falls in the 75-85% range (reported ~80%)
- Accuracy at scaling factor 32 falls in the 97-100% range (reported ~99%)
- An overflow/degraded-accuracy flag is set for factors above 128
- Monotone increase of accuracy from factor 2 toward 100% before overflow degradation

## Limitations

- 16-bit fixed-point with scaling factor 2 loses accuracy (~80%); too-small factors under-resolve normalized signal values
- Scaling factors above 128 cause data overflow during sDTW, degrading accuracy, so only 2-128 is a valid sweep range
- Accuracy is defined as identical mapping positions versus the float32 sigfish baseline, so it inherits baseline errors

## Evidence

- [other] scaling factor 32 for ~99% accuracy: [evidence span withheld — anchor status: not_found]
- [other] Fixed-point scaling factor accuracy curve: [evidence span withheld — anchor status: not_found]
- [readme] Run sigfish-haru dtw on example data: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"

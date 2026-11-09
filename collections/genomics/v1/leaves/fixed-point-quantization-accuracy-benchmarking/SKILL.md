---
name: fixed-point-quantization-accuracy-benchmarking
description: Use when porting a dynamic-programming signal-alignment (e.g., sDTW)
  pipeline from floating-point to fixed-point FPGA arithmetic and you must quantify
  the accuracy penalty of quantizing normalized signal values across candidate scaling
  factors.
when_to_use_negative:
- The pipeline already runs in 32-bit floating point and no FPGA resource/precision
  constraint motivates fixed-point conversion — the sweep would add cost with no decision
  to inform
- The alignment algorithm is not sDTW-based signal mapping (e.g., base-level alignment
  with Minimap2), where quantizing normalized signal values is not the relevant accuracy
  knob
- Signal values are unnormalized or in different units, since the scaling factor's
  meaning (quantization step size on normalized values) and the overflow threshold
  above 128 would not transfer
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: HARU
  role: Provides the sDTW hardware accelerator whose 16-bit fixed-point precision
    and scaling factor are being benchmarked
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Proof-of-concept application used to run sDTW squiggle mapping (dtw subcommand)
    against the reference to produce baseline and fixed-point mapping positions
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:fcdaff871a19763a17b3930d165d661f8c48b1cb8479922f3e4173bdfd51a06d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# fixed-point-quantization-accuracy-benchmarking

## Summary

Benchmarks how 16-bit fixed-point quantization of normalized nanopore signal values (controlled by an integer scaling factor) affects sDTW mapping accuracy in the HARU accelerator, sweeping scaling factors 2–128 against a 32-bit floating-point sigfish baseline to select the factor that maximizes accuracy without data overflow.

## When to use

Use when porting a dynamic-programming signal-alignment (e.g., sDTW) pipeline from floating-point to fixed-point FPGA arithmetic and you must quantify the accuracy penalty of quantizing normalized signal values across candidate scaling factors.

## When NOT to use

- The pipeline already runs in 32-bit floating point and no FPGA resource/precision constraint motivates fixed-point conversion — the sweep would add cost with no decision to inform
- The alignment algorithm is not sDTW-based signal mapping (e.g., base-level alignment with Minimap2), where quantizing normalized signal values is not the relevant accuracy knob
- Signal values are unnormalized or in different units, since the scaling factor's meaning (quantization step size on normalized values) and the overflow threshold above 128 would not transfer

## Inputs

- Nanopore raw signal reads in BLOW5 format (e.g., reads_0_0.blow5)
- Reference genome in FASTA format (e.g., nCoV-2019.reference.fasta)
- Floating-point sDTW baseline mapping positions (sigfish output.paf)
- Memory-optimized sDTW implementation from task_003 supporting 16-bit fixed-point arithmetic
- List of integer scaling factors to sweep (2, 4, 8, 16, 32, 64, 128)

## Outputs

- accuracy_vs_scaling_factor.csv with columns: scaling factor, accuracy (fraction of reads with identical mapping positions vs baseline), overflow/degraded-accuracy flag
- Selected operating scaling factor (32) for the HARU sDTW accelerator

## How to apply

First run the 32-bit floating-point sDTW mapping (sigfish dtw -g nCoV-2019.reference.fasta -s reads_0_0.blow5) to obtain baseline mapping positions for every read. Then, using the memory-optimized sDTW implementation, recompute mapping positions with 16-bit fixed-point arithmetic at integer scaling factors 2, 4, 8, 16, 32, 64, and 128 applied to the normalized signal values. For each factor, compare fixed-point mapping positions against the floating-point baseline and record the fraction of reads with identical positions as accuracy. The expected shape of the curve is low accuracy at small factors (~80% at factor 2, i.e., coarser quantization steps), convergence toward 100% as the factor grows, and degraded accuracy above 128 due to data overflow during sDTW. HARU settles on a scaling factor of 32, which gives ~99% accuracy while leaving headroom below the overflow regime.

## Related tools

- **HARU** (Provides the sDTW hardware accelerator whose 16-bit fixed-point precision and scaling factor are being benchmarked) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept application used to run sDTW squiggle mapping (dtw subcommand) against the reference to produce baseline and fixed-point mapping positions) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- accuracy_vs_scaling_factor.csv contains exactly 7 rows, one per scaling factor (2, 4, 8, 16, 32, 64, 128)
- Accuracy at scaling factor 2 falls in the 75–85% range (reported ~80%)
- Accuracy at scaling factor 32 falls in the 97–100% range (reported ~99%)
- Accuracy monotonically increases from factor 2 up to mid-range factors before overflow degrades it above 128
- Rows with factor > 128 (if extended) carry an overflow/degraded-accuracy flag set to true

## Limitations

- Above scaling factor 128, data overflow during sDTW degrades accuracy, so larger factors are not necessarily better
- Fixed-point accuracy is measured as identity of mapping positions against a 32-bit floating-point baseline, so it inherits any baseline mapping errors
- The validated sweep uses SARS-CoV-2 data (nCoV-2019.reference.fasta, reads_0_0.blow5); the optimal factor may differ for other genomes, chemistries, or read lengths

## Evidence

- [results] above 128, data overflow during sDTW degrades accuracy, so HARU uses a scaling factor of 32 for ~99% accuracy: [evidence span withheld — anchor status: not_found]
- [methods] Example sigfish-haru invocation on SARS-CoV-2 data: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [intro] HARU uses a memory-efficient sDTW hardware accelerator: "a software–hardware codesign system for raw signal-alignment Read Until that uses the memory-efficient sDTW hardware accelerator"
- [readme] The accelerator is loaded onto the FPGA before running the mapping: "xmutil unloadapp
    # load our HARU sDTW accelerator
    xmutil loadapp haru-dtw-firmware"

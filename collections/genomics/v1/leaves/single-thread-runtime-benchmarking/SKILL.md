---
name: single-thread-runtime-benchmarking
description: Use when you need to characterize how software alignment runtime scales
  with input size — e.g., fitting a power law runtime = c * N^k over reference lengths
  from 1 kb to ~295 kb to establish the complexity exponent of sDTW on CPU.
when_to_use_negative:
- When measuring hardware-accelerated (FPGA/HARU) alignment performance — this skill
  benchmarks the pure-CPU path only and must not run with fpga=1 or the haru-dtw-firmware
  loaded
- When throughput of the full Read Until pipeline (all processing stages) is the target,
  rather than isolated per-alignment sDTW query time
- When no reference sequence of sufficient length is available to construct a range
  of increasing-length subregions
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Sigfish-HARU
  role: Source repository for building the sigfish-cpu binary used to time single-thread
    CPU sDTW alignments
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Reference hardware-accelerated implementation whose CPU counterpart (sigfish-cpu)
    is benchmarked; defines the sDTW workload being scaled
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:34909db33f385dce98a17b2a14312ab390ce7a85f129f546f2d6f3e8ca241751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-thread-runtime-benchmarking

## Summary

Benchmarks single-thread CPU execution time of the sDTW squiggle-mapping alignment (via sigfish-cpu) as reference sequence length increases, to quantify the empirical superlinear/quadratic O(MN) scaling that motivates hardware acceleration.

## When to use

Use when you need to characterize how software alignment runtime scales with input size — e.g., fitting a power law runtime = c * N^k over reference lengths from 1 kb to ~295 kb to establish the complexity exponent of sDTW on CPU.

## When NOT to use

- When measuring hardware-accelerated (FPGA/HARU) alignment performance — this skill benchmarks the pure-CPU path only and must not run with fpga=1 or the haru-dtw-firmware loaded
- When throughput of the full Read Until pipeline (all processing stages) is the target, rather than isolated per-alignment sDTW query time
- When no reference sequence of sufficient length is available to construct a range of increasing-length subregions

## Inputs

- Reference FASTA (e.g., nCoV-2019.reference.fasta or hg38 RFC1 region) from which subregions of increasing length (1 kb–295 kb) are extracted
- Fixed query signal reads in BLOW5 format (reads_0_0.blow5)
- sigfish-cpu binary built from the sigfish-haru repository with hardware acceleration disabled (make PROCESSOR=aarch64)

## Outputs

- Runtime-vs-reference-length CSV with measured single-thread execution times per alignment
- Fitted power-law model runtime = c * N^k with empirical complexity exponent k

## How to apply

Build the sigfish-cpu binary from the sigfish-haru repository on aarch64 with hardware acceleration disabled (make PROCESSOR=aarch64, without fpga=1) so alignment runs purely on CPU. Extract subregions of increasing length (1 kb to 295 kb) from the reference FASTA (nCoV-2019.reference.fasta or an hg38 RFC1 region) to serve as progressively longer alignment targets. For each reference length, run ./sigfish-cpu dtw with the subregion as -g and the fixed query reads_0_0.blow5 as -s, timing single-thread execution per alignment. Record runtime alongside reference length in a CSV, then fit the power-law model runtime = c * N^k to estimate the empirical complexity exponent. Accept the benchmark if runtimes increase monotonically with reference length and the fitted exponent falls within [1.8, 2.2], confirming quadratic O(MN) software behavior.

## Related tools

- **Sigfish-HARU** (Source repository for building the sigfish-cpu binary used to time single-thread CPU sDTW alignments) — https://github.com/beebdev/sigfish-haru
- **HARU** (Reference hardware-accelerated implementation whose CPU counterpart (sigfish-cpu) is benchmarked; defines the sDTW workload being scaled) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The runtime-vs-reference-length CSV exists and contains measured runtimes for each reference subregion length
- Measured runtimes increase monotonically with reference length
- Fitted power-law exponent k falls within [1.8, 2.2], confirming quadratic O(MN) behavior
- The CPU curve diverges increasingly from the FPGA curve as reference length grows, so HARU's speedup rises with reference size

## Limitations

- Results apply to single-thread CPU execution of the software sDTW; the optimized multithreaded software version on a 36-core Xeon shows different absolute times (HARU is ~2.5x faster than that, ~85x faster than unoptimized software)
- Building HARU components requires proprietary AMD Xilinx tools (Vivado/Vitis 2021.1); build steps were tested only with 2021.1 tooling
- The benchmark was validated on aarch64 (Kria KV260 / Zynq Ultrascale+ MPSoC) with example data (nCoV-2019.reference.fasta, reads_0_0.blow5); other platforms or chemistries may behave differently

## Evidence

- [methods] Single-thread CPU sDTW query time grows superlinearly with reference length: [evidence span withheld — anchor status: not_found]
- [methods] CPU build and runtime fitting procedure: [evidence span withheld — anchor status: not_found]
- [methods] Validation thresholds for the benchmark: [evidence span withheld — anchor status: not_found]
- [readme] Example dtw invocation: "./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"

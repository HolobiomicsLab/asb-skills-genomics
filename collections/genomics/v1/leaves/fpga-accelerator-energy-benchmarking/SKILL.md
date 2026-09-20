---
name: fpga-accelerator-energy-benchmarking
description: Use when you must quantify hardware-accelerator efficiency against CPU
  baselines from per-read time (ms) and per-read energy (mJ) measurements, and report
  energy-delay product ratios such as HARU's ~650x EDP advantage over a 36-core Xeon
  Gold server.
when_to_use_negative:
- When only aggregate total-runtime measurements are available and per-read latency/energy
  pairs cannot be obtained, since EDP cannot be recomputed or validated
- When comparing basecalling-dependent base-domain Read Until pipelines rather than
  raw signal-domain sDTW alignment, as the energy profile and bottlenecks differ fundamentally
- When the workload is not matched across platforms (e.g., different datasets or read
  counts), invalidating the per-read normalization
tools:
- name: HARU
  role: FPGA-accelerated sDTW Read Until implementation being benchmarked; its per-read
    time and energy are the reference measurements against software baselines
  repo: https://github.com/beebdev/HARU
- name: sigfish-haru
  role: Example application used to execute the accelerated sDTW mapping workload
    whose measurements feed the benchmark
  repo: https://github.com/beebdev/sigfish-haru
- name: MinION
  role: Oxford Nanopore sequencer whose selective sequencing (Read Until) workload
    defines the execution context being benchmarked
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:bd4ba6b2b1350ac4d35dcd1193cd7d280b7b5f6ad5c7142e85fff0a69d57e2fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - MinION
  - sigfish-haru
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# fpga-accelerator-energy-benchmarking

## Summary

A benchmarking procedure for comparing an FPGA-accelerated Read Until implementation (HARU sDTW on Xilinx Kria) against optimized software-only baselines using reported per-read latency and energy measurements, combined into energy-delay products (EDP) to expose efficiency gains beyond wall-clock speedup.

## When to use

Use when you must quantify hardware-accelerator efficiency against CPU baselines from per-read time (ms) and per-read energy (mJ) measurements, and report energy-delay product ratios such as HARU's ~650x EDP advantage over a 36-core Xeon Gold server.

## When NOT to use

- When only aggregate total-runtime measurements are available and per-read latency/energy pairs cannot be obtained, since EDP cannot be recomputed or validated
- When comparing basecalling-dependent base-domain Read Until pipelines rather than raw signal-domain sDTW alignment, as the energy profile and bottlenecks differ fundamentally
- When the workload is not matched across platforms (e.g., different datasets or read counts), invalidating the per-read normalization

## Inputs

- Per-read execution time measurements in milliseconds for each platform (HARU FPGA, ARM software, Core-i9, 36-core Xeon Gold)
- Per-read energy consumption measurements in millijoules for each platform
- Platform specifications (e.g., Xilinx Kria KV260 Zynq Ultrascale+ MPSoC with 4-core ARM, 36-core Intel Xeon Gold server)
- Workload identifier (SARS-CoV-2 dataset)

## Outputs

- edp_comparison JSON table keyed by platform, containing per-read time, per-read energy, recomputed EDP, EDP ratio, and energy reduction factor
- Derived ratio figures (~650x EDP advantage, 341.7x energy reduction, 2.5x and 85x speedups)

## How to apply

Load the reported per-read latency and energy measurements for each platform: HARU on Xilinx FPGA with 4-core ARM (0.94 ms, 1.05 mJ), 4-core ARM software (90.2 ms, 217.9 mJ), Intel Core-i9 (5.9 ms, 740.9 mJ), and 36-core Intel Xeon Gold server (1.8 ms, 358.3 mJ), all evaluated on a SARS-CoV-2 dataset. Compute EDP per platform as time x energy, yielding approximately 0.987 for HARU and 644.94 for the Xeon server. Derive the EDP ratio of each baseline relative to HARU (approximately 650x for the server) and the energy reduction factor (341.7x), validating that recomputed EDPs match reported values within ~1% tolerance. Record all measurements and derived ratios in a single keyed JSON table for reproducible comparison. The rationale is that wall-clock speedup alone (2.5x over optimized multithreaded software, 85x over unoptimized) understates the FPGA's advantage, since energy consumption is two orders of magnitude lower than the server.

## Related tools

- **HARU** (FPGA-accelerated sDTW Read Until implementation being benchmarked; its per-read time and energy are the reference measurements against software baselines) — https://github.com/beebdev/HARU
- **sigfish-haru** (Example application used to execute the accelerated sDTW mapping workload whose measurements feed the benchmark) — https://github.com/beebdev/sigfish-haru
- **MinION** (Oxford Nanopore sequencer whose selective sequencing (Read Until) workload defines the execution context being benchmarked)

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Recomputed EDP for HARU (0.94 ms x 1.05 mJ = 0.987) must match the reported value within ~1% tolerance
- Recomputed EDP for the 36-core Xeon Gold server (1.8 ms x 358.3 mJ = 644.94) must match the reported value within ~1% tolerance
- Server/HARU EDP ratio must reproduce approximately 650x and the energy reduction must reproduce 341.7x within the same tolerance
- Speedup ratios must be consistent with reported figures: HARU ~2.5x faster than optimized multithreaded software and ~85x faster than unoptimized software
- Output JSON must be keyed by platform and include time, energy, EDP, and derived ratio fields for all four platforms

## Limitations

- Figures are from a proof-of-concept HARU implementation on a Xilinx Kria KV260 (Zynq Ultrascale+ MPSoC) for a SARS-CoV-2 dataset; results may not transfer to other workloads or hardware
- Measurements are based on reported per-read numbers from the Results > Energy Comparison section; raw measurement provenance or SARS-CoV-2 dataset accession details are not stated
- Building the core accelerator requires proprietary AMD Xilinx software (Vivado 2021.1/2022.2, PetaLinux 2021.1), limiting independent reproduction of the hardware baseline

## Evidence

- [results] HARU achieves 0.94 ms/read at 1.05 mJ/read, giving an energy-delay product of 0.987 versus 644.94 for the 36-core Xeon server: [evidence span withheld — anchor status: not_found]
- [abstract] HARU on a Xilinx FPGA with 4-core ARM processor is around 2.5x faster than a highly optimized multithreaded software version running on a 36-core Intel Xeon server for a SARS-CoV-2 dataset: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [abstract] The energy consumption of HARU is 2 orders of magnitudes lower than the same application on the 36-core server: "The energy consumption of HARU is 2 orders of magnitudes lower than the same application executing on the 36-core server."
- [readme] If you wish, you can run the sigfish-cpu binary that does not use the hardware accelerator and see how slow it is.: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."

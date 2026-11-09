---
name: energy-delay-product-computation
description: Use when a benchmarking source reports per-read (or per-operation) execution
  time and energy consumption separately for multiple platforms and the analysis goal
  is a normalized efficiency comparison via EDP and derived ratio fields.
when_to_use_negative:
- The source reports only throughput or total-job timings without paired per-read
  energy measurements — EDP cannot be recomputed without both latency and energy
- The comparison is between runs on the same platform (e.g., sigfish-haru vs. sigfish-cpu
  on the Kria KV260) where a raw speed ratio alone is sufficient
- Measurements are total-workflow rather than per-read, unless they are first normalized
  to a per-read basis
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: HARU
  role: Hardware Accelerated Read Until system whose per-read latency/energy measurements
    on a Xilinx FPGA (Kria KV260) serve as the low-power baseline in the EDP comparison
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:c99e11a267ebb6277ccf7da37386df682aa7789b4f7dcaeb60bfdf9ed213c912
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# energy-delay-product-computation

## Summary

Recompute energy-delay product (EDP) as per-read latency (ms) × per-read energy (mJ) from reported benchmark measurements, and derive cross-platform ratios (e.g., HARU on Xilinx FPGA EDP ≈0.987 vs. 36-core Intel Xeon Gold server ≈644.94, a ~650x EDP gap). This turns scattered per-platform numbers into a single normalized efficiency metric for hardware-software codesign comparisons.

## When to use

Use when a benchmarking source reports per-read (or per-operation) execution time and energy consumption separately for multiple platforms and the analysis goal is a normalized efficiency comparison via EDP and derived ratio fields.

## When NOT to use

- The source reports only throughput or total-job timings without paired per-read energy measurements — EDP cannot be recomputed without both latency and energy
- The comparison is between runs on the same platform (e.g., sigfish-haru vs. sigfish-cpu on the Kria KV260) where a raw speed ratio alone is sufficient
- Measurements are total-workflow rather than per-read, unless they are first normalized to a per-read basis

## Inputs

- Per-read latency measurements in milliseconds per platform (e.g., HARU 0.94 ms, ARM 90.2 ms, Core-i9 5.9 ms, Xeon Gold 1.8 ms)
- Per-read energy measurements in millijoules per platform (e.g., HARU 1.05 mJ, ARM 217.9 mJ, Core-i9 740.9 mJ, Xeon Gold 358.3 mJ)
- Platform descriptors (HARU on Xilinx FPGA with 4-core ARM; 4-core ARM software; Intel Core-i9; 36-core Intel Xeon Gold server)

## Outputs

- JSON table 'edp_comparison' keyed by platform with recomputed EDP values (ms·mJ)
- Derived ratio fields: server-to-HARU EDP ratio (~650x) and HARU-to-server energy reduction (341.7x)

## How to apply

Extract the reported per-read latency and per-read energy for each platform from the Results > Energy Comparison section (HARU on Xilinx FPGA: 0.94 ms, 1.05 mJ; 4-core ARM software: 90.2 ms, 217.9 mJ; Intel Core-i9: 5.9 ms, 740.9 mJ; 36-core Xeon Gold: 1.8 ms, 358.3 mJ). Compute EDP = per-read time (ms) × per-read energy (mJ) for each platform, yielding ≈0.987 for HARU and ≈644.94 for the 36-core Xeon Gold server. Derive the server-to-HARU EDP ratio (≈650x) and the HARU-to-server energy reduction (341.7x), then write all measurements, recomputed EDPs, and ratios into a single JSON table (edp_comparison) keyed by platform. Validate by checking each recomputed EDP and ratio reproduces the reported values within ~1% tolerance.

## Related tools

- **HARU** (Hardware Accelerated Read Until system whose per-read latency/energy measurements on a Xilinx FPGA (Kria KV260) serve as the low-power baseline in the EDP comparison) — https://github.com/beebdev/HARU

## Examples

```
edp = {'HARU_FPGA': 0.94*1.05, 'ARM_4core': 90.2*217.9, 'Core-i9': 5.9*740.9, 'Xeon36': 1.8*358.3}; ratio = edp['Xeon36']/edp['HARU_FPGA']
```

## Evaluation signals

- Recomputed EDP for HARU equals 0.987 (0.94 × 1.05) within ~1% tolerance
- Recomputed EDP for the 36-core Xeon Gold server equals 644.94 (1.8 × 358.3) within ~1% tolerance
- Server-to-HARU EDP ratio reproduces ~650x within ~1% tolerance
- HARU energy reduction factor reproduces 341.7x (358.3 / 1.05) within ~1% tolerance
- edp_comparison JSON table contains entries for all four platforms (HARU, 4-core ARM, Core-i9, 36-core Xeon Gold) with latency, energy, EDP, and ratio fields

## Limitations

- EDP values are derived from reported per-read measurements on a SARS-CoV-2 dataset, not independently measured; accuracy inherits any errors in the source numbers
- The comparison is specific to the sDTW squiggle-mapping workload for Read Until on R9.4 nanopore chemistry; results may not generalize to other workloads
- Energy consumption is 2 orders of magnitudes lower for HARU than the 36-core server, so platform power-estimation methodology differences could affect the ratios

## Evidence

- [abstract] HARU energy advantage over server: "The energy consumption of HARU is 2 orders of magnitudes lower than the same application executing on the 36-core server."
- [abstract] HARU speed advantage over optimized server software: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [readme] Unoptimized CPU baseline exists for contrast: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."

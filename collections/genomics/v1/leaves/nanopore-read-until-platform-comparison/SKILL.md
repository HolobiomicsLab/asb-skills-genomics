---
name: nanopore-read-until-platform-comparison
description: Use when reported per-read execution time (ms) and energy (mJ) measurements
  exist for HARU (Xilinx FPGA + 4-core ARM) and one or more software baselines, and
  the goal is to derive or verify EDP and platform-level efficiency ratios.
when_to_use_negative:
- The sources report only throughput or wall-clock totals without per-read energy
  measurements — EDP cannot be reconstructed and a latency-only comparison should
  be used instead
- The task is to actually run the HARU pipeline (flash PetaLinux image, load haru-dtw-firmware,
  run sigfish-haru dtw) rather than to analyze or verify its reported benchmark numbers
- The comparison targets a different workload (e.g. non-SARS-CoV-2 or non-Read-Until
  benchmark) whose absolute EDP values would not transfer
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: Accelerated sDTW Read Until implementation on Xilinx FPGA + 4-core ARM; the
    low-EDP platform being benchmarked
  repo: https://github.com/beebdev/HARU
- name: sigfish-haru
  role: Proof-of-concept application integrating the HARU accelerator for squiggle
    mapping; the application-level execution measured in the comparison
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:e4862bb448b2643317b860a3bc00428c05ea8a83becdb4ee9babcfd344d52e9b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - sigfish-haru
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nanopore-read-until-platform-comparison

## Summary

Reconstructs the energy-delay product (EDP = per-read time × per-read energy) comparison of HARU's FPGA-accelerated sDTW Read Until against optimized software-only baselines (4-core ARM, Intel Core-i9, 36-core Xeon Gold), quantifying how much latency and energy an edge accelerator saves over server-class implementations.

## When to use

Use when reported per-read execution time (ms) and energy (mJ) measurements exist for HARU (Xilinx FPGA + 4-core ARM) and one or more software baselines, and the goal is to derive or verify EDP and platform-level efficiency ratios.

## When NOT to use

- The sources report only throughput or wall-clock totals without per-read energy measurements — EDP cannot be reconstructed and a latency-only comparison should be used instead
- The task is to actually run the HARU pipeline (flash PetaLinux image, load haru-dtw-firmware, run sigfish-haru dtw) rather than to analyze or verify its reported benchmark numbers
- The comparison targets a different workload (e.g. non-SARS-CoV-2 or non-Read-Until benchmark) whose absolute EDP values would not transfer

## Inputs

- Reported per-read latency (ms) and per-read energy (mJ) per platform (HARU FPGA, 4-core ARM, Intel Core-i9, 36-core Xeon Gold)
- SARS-CoV-2 dataset as the common workload for all platforms

## Outputs

- JSON-style edp_comparison table keyed by platform, containing per-read time, per-read energy, recomputed EDP, EDP ratio vs. HARU, and energy-reduction ratio

## How to apply

Load the reported per-read latency and energy pairs for each platform: HARU on Xilinx FPGA (0.94 ms, 1.05 mJ), 4-core ARM software (90.2 ms, 217.9 mJ), Intel Core-i9 (5.9 ms, 740.9 mJ), and 36-core Xeon Gold server (1.8 ms, 358.3 mJ), all measured on a SARS-CoV-2 dataset. Compute EDP = per-read time (ms) × per-read energy (mJ) for each platform, yielding ~0.987 for HARU and ~644.94 for the 36-core Xeon Gold server. Derive the server-to-HARU EDP ratio (~650x) and HARU's energy reduction relative to the server (~341.7x, i.e. two orders of magnitude lower energy). Also cross-check the speed ratios reported for the same dataset: HARU is ~2.5x faster than the optimized multithreaded 36-core Xeon version and ~85x faster than the unoptimized multithreaded software. Assemble all measurements, recomputed EDPs, and derived ratios into a single table (e.g. edp_comparison) keyed by platform.

## Related tools

- **HARU** (Accelerated sDTW Read Until implementation on Xilinx FPGA + 4-core ARM; the low-EDP platform being benchmarked) — https://github.com/beebdev/HARU
- **sigfish-haru** (Proof-of-concept application integrating the HARU accelerator for squiggle mapping; the application-level execution measured in the comparison) — https://github.com/beebdev/sigfish-haru

## Examples

```
python -c "p={'HARU':(0.94,1.05),'ARM4':(90.2,217.9),'i9':(5.9,740.9),'Xeon36':(1.8,358.3)}; print({k:{'edp':t*e} for k,(t,e) in p.items()})"
```

## Evaluation signals

- Recomputed EDP for HARU equals ~0.987 (0.94 ms × 1.05 mJ) and for the 36-core Xeon Gold equals ~644.94 (1.8 ms × 358.3 mJ), each within ~1% tolerance
- Server-to-HARU EDP ratio reproduces ~650x within tolerance
- HARU-to-server energy ratio reproduces ~341.7x (two orders of magnitude) within tolerance
- Speed ratio check: HARU ~2.5x faster than optimized multithreaded 36-core Xeon software on the SARS-CoV-2 dataset
- Speed ratio check: HARU ~85x faster than existing unoptimized multithreaded software

## Limitations

- Absolute EDP values depend on the SARS-CoV-2 dataset and the specific platform configurations (Kria KV260 with Zynq Ultrascale+ MPSoC vs. Core-i9 vs. Xeon Gold); they are not portable to other datasets or hardware
- Comparison relies on the reported software baselines (4-core ARM, Core-i9, 36-core Xeon Gold); the HARU build itself requires Xilinx Vivado/PetaLinux toolchains (e.g. Vivado 2022.2, PetaLinux 2021.1), so reproduction is toolchain-bound
- HARU's proof-of-concept uses a custom sDTW accelerator targeted specifically at the Kria Kria AI Starter Kit, so results do not generalize to arbitrary FPGA boards without rebuild

## Evidence

- [abstract] HARU energy advantage over server: "The energy consumption of HARU is 2 orders of magnitudes lower than the same application executing on the 36-core server."
- [abstract] HARU speed vs optimized multithreaded baseline: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [intro] Scope: platforms selected for portability/power trade-off: "Existing alignment methods for selective sequencing use high-performance computing systems to meet the real-time processing requirement, compromising portability, cost-effectiveness, and power"
- [readme] CPU-only baseline binary available for comparison: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."

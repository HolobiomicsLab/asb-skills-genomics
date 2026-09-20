---
name: reported-metrics-recomputation-from-publication
description: Use when a paper reports per-read execution time and energy per platform
  (e.g., HARU 0.94 ms/1.05 mJ vs Xeon 1.8 ms/358.3 mJ) and a derived metric such as
  EDP or speedup/energy ratios must be reproduced or verified.
when_to_use_negative:
- Raw latency/energy measurements must be collected experimentally rather than extracted
  from a publication's reported values
- The reported values are given only as aggregated or normalized quantities from which
  per-read time and per-read energy cannot be recovered
- The analysis target is a different metric (e.g., throughput, accuracy) that cannot
  be derived from time x energy per read
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3399
tools:
- name: HARU
  role: The FPGA-accelerated sDTW Read Until system whose reported per-read latency
    (0.94 ms) and energy (1.05 mJ) on a Xilinx Kria FPGA serve as the comparison baseline
    in the EDP recomputation
  repo: https://github.com/beebdev/HARU
- name: MinION
  role: Oxford Nanopore sequencer whose selective sequencing (Read Until) workload
    provides the per-read measurements that the EDP comparison is computed over
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:8de87a8a2507ba96c737fc774879875a29596b2c9551d02c6a43978a3e371367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - MinION
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3399
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# reported-metrics-recomputation-from-publication

## Summary

Recompute derived performance metrics (e.g., energy-delay product) from per-read latency and energy measurements reported in a publication, and validate that recomputed values and ratios match the reported figures. This turns scattered reported numbers into a machine-checkable table for cross-platform comparisons such as HARU vs. CPU servers.

## When to use

Use when a paper reports per-read execution time and energy per platform (e.g., HARU 0.94 ms/1.05 mJ vs Xeon 1.8 ms/358.3 mJ) and a derived metric such as EDP or speedup/energy ratios must be reproduced or verified.

## When NOT to use

- Raw latency/energy measurements must be collected experimentally rather than extracted from a publication's reported values
- The reported values are given only as aggregated or normalized quantities from which per-read time and per-read energy cannot be recovered
- The analysis target is a different metric (e.g., throughput, accuracy) that cannot be derived from time x energy per read

## Inputs

- Reported per-read execution time (ms) per platform from the Results > Energy Comparison section
- Reported per-read energy consumption (mJ) per platform
- Reported derived metric values (e.g., EDP = 0.987 for HARU, 644.94 for 36-core Xeon Gold) for validation
- Platform definitions (HARU on Xilinx FPGA, 4-core ARM, Intel Core-i9, 36-core Intel Xeon Gold server)

## Outputs

- JSON table (edp_comparison) keyed by platform, containing reported time, reported energy, recomputed EDP, and derived ratio fields (EDP ratio, energy reduction)
- Validation report confirming recomputed EDP and ratio values match reported values within ~1% tolerance

## How to apply

Load the reported per-read measurements from the Results > Energy Comparison section: HARU on Xilinx FPGA (0.94 ms, 1.05 mJ), 4-core ARM software (90.2 ms, 217.9 mJ), Intel Core-i9 (5.9 ms, 740.9 mJ), and 36-core Intel Xeon Gold server (1.8 ms, 358.3 mJ). Compute the energy-delay product for each platform as EDP = per-read time (ms) x per-read energy (mJ), yielding approximately 0.987 for HARU and 644.94 for the 36-core Xeon Gold server. Derive the baseline-to-HARU ratios: EDP ratio ~650x and energy reduction ~341.7x for the server relative to HARU. Write all measurements, recomputed EDP values, and derived ratios into a single JSON table (e.g., edp_comparison) keyed by platform. Validate that each recomputed EDP matches the reported value within ~1% tolerance and that the ratio fields reproduce the reported ~650x EDP ratio and 341.7x energy reduction within the same tolerance.

## Related tools

- **HARU** (The FPGA-accelerated sDTW Read Until system whose reported per-read latency (0.94 ms) and energy (1.05 mJ) on a Xilinx Kria FPGA serve as the comparison baseline in the EDP recomputation) — https://github.com/beebdev/HARU
- **MinION** (Oxford Nanopore sequencer whose selective sequencing (Read Until) workload provides the per-read measurements that the EDP comparison is computed over)

## Evaluation signals

- Recomputed EDP for HARU equals the reported 0.987 within ~1% tolerance
- Recomputed EDP for the 36-core Xeon Gold server equals the reported 644.94 within ~1% tolerance
- Derived server-to-HARU EDP ratio reproduces the reported ~650x within ~1% tolerance
- Derived HARU-to-server energy reduction reproduces the reported 341.7x within ~1% tolerance
- Output JSON table contains an entry for every reported platform (HARU, 4-core ARM, Core-i9, 36-core Xeon Gold) with both time and energy fields populated

## Limitations

- Recomputation is only possible for platforms where the paper reports both per-read time and per-read energy explicitly; missing values block EDP derivation
- Values are reported for a specific workload (SARS-CoV-2 dataset) and chemistry (R9.4); they do not generalize to other datasets or nanopore chemistries
- The comparison reflects a proof-of-concept HARU implementation targeted at a Xilinx Kria KV260 (Zynq Ultrascale+ MPSoC); different FPGA targets may yield different measurements

## Evidence

- [results] EDP recomputation from reported per-read measurements: "The energy consumption of HARU is 2 orders of magnitudes lower than the same application executing on the 36-core server."
- [abstract] Platform measurement scope: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [discussion] Workload definition: "for a complete SARS-CoV-2 dataset"
- [readme] HARU hardware target: "Our current proof-of-concept implementation of HARU utilises a custom subsequence DTW accelerator primarily targeted for a [Xilinx's Kria AI Starter Kit]"

---
name: benchmark-performance-ratio-analysis
description: Use when an article reports per-read execution time and per-read energy
  for multiple hardware platforms (e.g., HARU FPGA vs. ARM, Core-i9, Xeon Gold) and
  you must recompute EDP, speedup, and energy ratios into a machine-readable comparison
  table.
when_to_use_negative:
- The article reports only aggregate runtimes or throughput without per-read latency
  and per-read energy pairs, so EDP cannot be recomputed per platform
- The comparison targets basecalling-based (base-domain) Read Until pipelines rather
  than signal-alignment (raw signal domain) benchmarks, where the sDTW accelerator
  metrics do not apply
- Platforms were benchmarked on different datasets or read sets, making cross-platform
  ratio normalization invalid
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: HARU
  role: Hardware-accelerated sDTW Read Until implementation benchmarked on a Xilinx
    Kria FPGA (KV260, Zynq Ultrascale+ MPSoC with 4-core ARM); provides the reference
    per-read latency and energy measurements for the ratio analysis
  repo: https://github.com/beebdev/HARU
- name: sigfish-haru
  role: Proof-of-concept squiggle-mapping application using the HARU accelerator;
    the runnable software whose dtw benchmark results feed the per-read measurements
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:deb81b1f5f1b905cd6d2f2256056292418a1a58aa8353d4d941fb7a1209174d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - sigfish-haru
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-performance-ratio-analysis

## Summary

Reconstructs platform-level benchmark comparisons by recomputing derived metrics (energy-delay product, speedup, energy-reduction ratios) from reported per-read latency and energy measurements, and validates them against the reported values. Applied to HARU's cross-platform comparison of FPGA hardware acceleration versus multithreaded software on CPU platforms for signal-domain Read Until.

## When to use

Use when an article reports per-read execution time and per-read energy for multiple hardware platforms (e.g., HARU FPGA vs. ARM, Core-i9, Xeon Gold) and you must recompute EDP, speedup, and energy ratios into a machine-readable comparison table.

## When NOT to use

- The article reports only aggregate runtimes or throughput without per-read latency and per-read energy pairs, so EDP cannot be recomputed per platform
- The comparison targets basecalling-based (base-domain) Read Until pipelines rather than signal-alignment (raw signal domain) benchmarks, where the sDTW accelerator metrics do not apply
- Platforms were benchmarked on different datasets or read sets, making cross-platform ratio normalization invalid

## Inputs

- Reported per-read execution time (ms) per platform: HARU FPGA 0.94, ARM 4-core 90.2, Core-i9 5.9, Xeon Gold 36-core 1.8
- Reported per-read energy consumption (mJ) per platform: HARU FPGA 1.05, ARM 4-core 217.9, Core-i9 740.9, Xeon Gold 36-core 358.3
- SARS-CoV-2 dataset used as the common benchmark workload across platforms

## Outputs

- JSON table (edp_comparison) keyed by platform, containing per-read time, per-read energy, recomputed EDP, EDP ratio vs. HARU, and energy-reduction ratio
- Validated EDP values (HARU ~0.987, 36-core Xeon Gold ~644.94) and derived ratios (~650x EDP improvement, ~341.7x energy reduction)

## How to apply

Load the reported per-read latency and energy measurements for each platform: HARU on the Xilinx Kria FPGA (0.94 ms/read, 1.05 mJ/read), 4-core ARM software (90.2 ms, 217.9 mJ), Intel Core-i9 (5.9 ms, 740.9 mJ), and 36-core Xeon Gold server (1.8 ms, 358.3 mJ). Compute the energy-delay product per platform as EDP = per-read time (ms) × per-read energy (mJ), yielding approximately 0.987 for HARU and 644.94 for the 36-core Xeon Gold server. Derive the EDP ratio of the server baseline relative to HARU (~650x) and the energy reduction of HARU relative to the server (~341.7x, i.e., two orders of magnitude). Validate by requiring each recomputed EDP to match the reported value (HARU 0.987, server 644.94) within ~1% tolerance and the ratio fields to reproduce the ~650x EDP ratio and 341.7x energy reduction within the same tolerance. Write all measurements, recomputed EDPs, and derived ratios into a single JSON table keyed by platform.

## Related tools

- **HARU** (Hardware-accelerated sDTW Read Until implementation benchmarked on a Xilinx Kria FPGA (KV260, Zynq Ultrascale+ MPSoC with 4-core ARM); provides the reference per-read latency and energy measurements for the ratio analysis) — https://github.com/beebdev/HARU
- **sigfish-haru** (Proof-of-concept squiggle-mapping application using the HARU accelerator; the runnable software whose dtw benchmark results feed the per-read measurements) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Recomputed EDP for HARU equals 0.987 (0.94 ms × 1.05 mJ) within ~1% tolerance
- Recomputed EDP for the 36-core Xeon Gold equals 644.94 (1.8 ms × 358.3 mJ) within ~1% tolerance
- EDP ratio (server / HARU) reproduces approximately 650x within the same tolerance
- Energy ratio (358.3 mJ / 1.05 mJ) reproduces the reported 341.7x reduction, consistent with the article's '2 orders of magnitudes lower' claim
- Output is a single JSON table (edp_comparison) keyed by platform with all measurements, recomputed EDPs, and ratios present

## Limitations

- Measurements are for a SARS-CoV-2 dataset; ratios may not generalize to other references or chemistries
- The SARS-CoV-2 dataset lacks a stated accession or hosting location in the article, hindering exact reproduction of the benchmark input
- The comparison relies on reported per-read figures from the Results > Energy Comparison section; a baseline software version was noted as unoptimized and ~85x slower, so choice of software baseline strongly affects ratios

## Evidence

- [results] HARU achieves 0.94 ms/read at 1.05 mJ/read, giving an energy-delay product of 0.987 versus 644.94 for the 36-core Xeon server: [evidence span withheld — anchor status: not_found]
- [abstract] The energy consumption of HARU is 2 orders of magnitudes lower than the same application on the 36-core server: "The energy consumption of HARU is 2 orders of magnitudes lower than the same application executing on the 36-core server."
- [abstract] HARU on a Xilinx FPGA with 4-core ARM processor is around 2.5x faster than a highly optimized multithreaded software version: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [abstract] HARU is around 85x faster than the existing unoptimized multithreaded software: "around 85× faster than the existing unoptimized multithreaded software"
- [abstract] running on a sophisticated server with a 36-core Intel Xeon processor for a SARS-CoV-2 dataset: "running on a sophisticated server with a 36-core Intel Xeon processor for a SARS-CoV-2 dataset"

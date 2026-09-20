---
name: execution-time-percentage-analysis
description: Use when you need to quantify which pipeline stage dominates runtime
  (e.g., sDTW vs preprocessing vs I/O) to justify or evaluate hardware acceleration
  on a specific dataset such as SARS-CoV-2 BLOW5 nanopore reads.
when_to_use_negative:
- The goal is end-to-end throughput benchmarking (e.g., reads/second or the reported
  2.5x/85x speedups) rather than a per-stage percentage breakdown of where time is
  spent
- The pipeline being profiled is not a staged squiggle-mapping workflow (sDTW/preprocessing/I-O)
  — a single monolithic step has no meaningful per-stage share to attribute
- The dataset is not representative (timing shares like the >98% sDTW figure were
  reported for the complete SARS-CoV-2 dataset; shares may differ substantially on
  other read sets)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: Sigfish-HARU
  role: Fork of sigfish demonstrating HARU integration; its dtw command is run with
    wall-clock timing instrumentation around each pipeline stage to produce the per-stage
    breakdown
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: FPGA sDTW accelerator; running its accelerated build alongside the software-only
    build lets the analyst contrast the sDTW share (<64%) against the >98% software
    baseline
  repo: https://github.com/beebdev/HARU
- name: MinION
  role: Oxford Nanopore sequencer whose raw signal reads, processed as Read Until
    adaptive sampling, constitute the workload being profiled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:482859ae9c552289bdd8db97ed2133730c0966e184494f9a24e6d0e413a179d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - MinION
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# execution-time-percentage-analysis

## Summary

Profiles a nanopore Read Until pipeline (e.g., sigfish/sigfish-haru sDTW squiggle mapping) by instrumenting wall-clock time around each stage — sDTW alignment, signal preprocessing, and file I/O — and reports each stage's share of total runtime as a percentage. This reveals computational bottlenecks such as sDTW dominating >98% of software-only runtime versus <64% with the HARU FPGA accelerator.

## When to use

Use when you need to quantify which pipeline stage dominates runtime (e.g., sDTW vs preprocessing vs I/O) to justify or evaluate hardware acceleration on a specific dataset such as SARS-CoV-2 BLOW5 nanopore reads.

## When NOT to use

- The goal is end-to-end throughput benchmarking (e.g., reads/second or the reported 2.5x/85x speedups) rather than a per-stage percentage breakdown of where time is spent
- The pipeline being profiled is not a staged squiggle-mapping workflow (sDTW/preprocessing/I-O) — a single monolithic step has no meaningful per-stage share to attribute
- The dataset is not representative (timing shares like the >98% sDTW figure were reported for the complete SARS-CoV-2 dataset; shares may differ substantially on other read sets)

## Inputs

- Nanopore read dataset in BLOW5 format (e.g., SARS-CoV-2 dataset, Zenodo DOI 10.5281/zenodo.7314838, reads_0_0.blow5)
- Reference genome FASTA (e.g., nCoV-2019.reference.fasta)
- Built sigfish or sigfish-haru binary (software-only sigfish-cpu and/or FPGA-accelerated variant)

## Outputs

- Per-stage execution times (sDTW alignment, signal preprocessing, file I/O)
- Per-stage percentage shares of total runtime, e.g., written to runtime_breakdown.csv
- Comparison of sDTW runtime share between software-only and hardware-accelerated builds

## How to apply

Build the pipeline variant to profile (e.g., sigfish-cpu from sigfish-haru with `make PROCESSOR=aarch64`, without the `fpga=1` hardware-acceleration flag) and run the sDTW squiggle-mapping command `./sigfish dtw -g <reference.fasta> -s <reads.blow5>` on the target dataset (e.g., SARS-CoV-2 reads_0_0.blow5 against nCoV-2019.reference.fasta). Instrument wall-clock timing around each pipeline stage: sDTW alignment, signal preprocessing, and file I/O. Aggregate each stage's execution time, compute its percentage of total runtime, and write the per-stage breakdown to a table such as runtime_breakdown.csv. Judge correctness by comparing against reported shares: the sDTW stage should account for more than 98% of total software runtime (Fig. 6 of the HARU paper), while the hardware-accelerated HARU build drops the sDTW share to less than 64% for the SARS-CoV-2 dataset.

## Related tools

- **Sigfish-HARU** (Fork of sigfish demonstrating HARU integration; its dtw command is run with wall-clock timing instrumentation around each pipeline stage to produce the per-stage breakdown) — https://github.com/beebdev/sigfish-haru
- **HARU** (FPGA sDTW accelerator; running its accelerated build alongside the software-only build lets the analyst contrast the sDTW share (<64%) against the >98% software baseline) — https://github.com/beebdev/HARU
- **MinION** (Oxford Nanopore sequencer whose raw signal reads, processed as Read Until adaptive sampling, constitute the workload being profiled)

## Examples

```
./sigfish dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf  # with wall-clock timers around sDTW, preprocessing, and I/O stages
```

## Evaluation signals

- The sDTW alignment stage accounts for more than 98% of total software-only runtime on the SARS-CoV-2 dataset, consistent with the >98% share reported in Fig. 6
- In the HARU (FPGA-accelerated) build, the sDTW share drops below 64% for the SARS-CoV-2 dataset
- Stage times aggregate exactly to total wall-clock runtime (no unaccounted time or double counting between preprocessing, sDTW, and I/O)
- All stage percentages sum to ~100% and the breakdown table (runtime_breakdown.csv) is produced without the profiling script erroring
- The same command (`./sigfish dtw -g nCoV-2019.reference.fasta -s reads.blow5 > output.paf`) produces a valid PAF output while timing instrumentation is enabled

## Limitations

- Percentage shares are dataset- and platform-specific: the >98% sDTW figure was reported for a complete SARS-CoV-2 dataset on the software-only implementation and may not transfer to other read sets or hardware
- Comparing software-only and HARU builds requires correctly compiled variants — the CPU-only build must omit the `fpga=1` hardware-acceleration flag, and building HARU itself requires proprietary AMD Xilinx tooling (Vivado/Vitis 2021.1) targeted at the Kria KV260
- The article lacks a changelog and the SARS-CoV-2 dataset hosting/accession is not stated in all sections, which can complicate reproducing the exact profiling inputs

## Evidence

- [results] In the software-only implementations, over 98% of the runtime is spent on sDTW computation, whereas in HARU the sDTW share drops to less than 64% for the SARS-CoV-2 dataset.: [evidence span withheld — anchor status: not_found]
- [methods] Run the sDTW squiggle-mapping command with wall-clock timing instrumentation enabled around each pipeline stage.: [evidence span withheld — anchor status: not_found]
- [methods] Build the CPU-only software version of sigfish from the sigfish-haru repository using `make PROCESSOR=aarch64`.: [evidence span withheld — anchor status: not_found]
- [readme] ./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is.: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."

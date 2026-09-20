---
name: software-runtime-profiling-per-stage-decomposition
description: Use when you need to quantify the per-stage runtime share of a nanopore
  Read Until / squiggle-mapping pipeline (e.g. to verify that sDTW dominates software
  execution before investing in FPGA acceleration).
when_to_use_negative:
- When profiling the hardware-accelerated (HARU/FPGA) pipeline — the goal there is
  to show the reduced sDTW share (<64%), not the software bottleneck
- When the input reads are not raw nanopore signal in BLOW5 format or lack a matched
  reference FASTA for squiggle mapping
- When a coarse end-to-end timing comparison against the 36-core Intel Xeon server
  baseline is sufficient and per-stage decomposition is unnecessary
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: Sigfish-HARU
  role: Fork of sigfish whose CPU-only build is instrumented to time the sDTW, preprocessing,
    and I/O stages of squiggle mapping
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Hardware-accelerated reference whose reported sDTW share (<64%) serves as
    the contrast baseline for the >98% software-only sDTW share
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:72b3df7117d2944dfc17de5ea1dda4da62bc2f45667ae39a046f364b91f27dc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# software-runtime-profiling-per-stage-decomposition

## Summary

Profiles the software-only Read Until pipeline by timing each stage (sDTW alignment, signal preprocessing, file I/O) on the SARS-CoV-2 nanopore dataset to quantify what fraction of total runtime the sDTW computation occupies, establishing the >98% bottleneck baseline that motivates hardware acceleration with HARU.

## When to use

Use when you need to quantify the per-stage runtime share of a nanopore Read Until / squiggle-mapping pipeline (e.g. to verify that sDTW dominates software execution before investing in FPGA acceleration).

## When NOT to use

- When profiling the hardware-accelerated (HARU/FPGA) pipeline — the goal there is to show the reduced sDTW share (<64%), not the software bottleneck
- When the input reads are not raw nanopore signal in BLOW5 format or lack a matched reference FASTA for squiggle mapping
- When a coarse end-to-end timing comparison against the 36-core Intel Xeon server baseline is sufficient and per-stage decomposition is unnecessary

## Inputs

- SARS-CoV-2 nanopore read dataset in BLOW5 format (Zenodo DOI 10.5281/zenodo.7314838)
- nCoV-2019 reference genome in FASTA format (e.g. nCoV-2019.reference.fasta)
- CPU-only (software-only) sigfish binary built without the fpga=1 flag

## Outputs

- Per-stage execution times (seconds) for sDTW alignment, signal preprocessing, and file I/O
- Per-stage runtime percentage table written to runtime_breakdown.csv
- Validated finding that sDTW accounts for >98% of total software-only runtime

## How to apply

Build the CPU-only version of sigfish from the sigfish-haru repository with `make PROCESSOR=aarch64`, omitting the `fpga=1` hardware-acceleration flag, so the pipeline runs entirely in software. Obtain the SARS-CoV-2 nanopore reads in BLOW5 format (Zenodo DOI 10.5281/zenodo.7314838) and the nCoV-2019 reference FASTA, then run the sDTW squiggle-mapping command `./sigfish dtw -g <reference.fasta> -s <reads.blow5>` with wall-clock timing instrumentation wrapped around each pipeline stage. Aggregate the measured execution times for the sDTW alignment stage, the signal preprocessing stage, and the file I/O stage, and compute each stage's percentage of total runtime. The rationale is that per-stage decomposition isolates the computational bottleneck: the article reports over 98% of software-only runtime is spent on sDTW, whereas HARU reduces the sDTW share to less than 64% for the SARS-CoV-2 dataset. Write the per-stage percentages to a table (e.g. runtime_breakdown.csv) for comparison against the reported Fig. 6 breakdown.

## Related tools

- **Sigfish-HARU** (Fork of sigfish whose CPU-only build is instrumented to time the sDTW, preprocessing, and I/O stages of squiggle mapping) — https://github.com/beebdev/sigfish-haru
- **HARU** (Hardware-accelerated reference whose reported sDTW share (<64%) serves as the contrast baseline for the >98% software-only sDTW share) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-cpu dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The sDTW stage must account for more than 98% of total software-only runtime, consistent with the >98% share reported in Fig. 6
- Per-stage percentages for sDTW, preprocessing, and file I/O must sum to ~100% of wall-clock time
- The profiling script must run without error on the BLOW5 SARS-CoV-2 dataset and nCoV-2019 reference FASTA
- The output.paf mapping file from the timed dtw run must be produced normally, confirming the profiled run is functionally valid
- The measured runtime breakdown should differ markedly from HARU's reported <64% sDTW share, confirming the software-only configuration was actually profiled

## Limitations

- Results are specific to the SARS-CoV-2 dataset and R9.4 nanopore chemistry; shares may differ for other datasets or chemistries
- Building the accelerator components requires proprietary AMD Xilinx tools (Vivado/Vitis 2021.1), though the CPU-only profiling build itself avoids the fpga=1 flag
- Wall-clock timing on an embedded ARM platform may be affected by I/O (micro SD card) jitter, so I/O-stage measurements are approximate

## Evidence

- [results] Software-only sDTW runtime dominance: [evidence span withheld — anchor status: not_found]
- [discussion] Sigfish-HARU proof-of-concept integration: "Project name: Sigfish-HARUDescription: Source code that demonstrates the proof-of-concept integration of HARU accelerator for squiggle mapping"
- [methods] Example dtw command from methods: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] CPU-only binary for comparison from README: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."
- [intro] sDTW as the computational bottleneck: "HARU tackles the computational bottleneck by accelerating the sDTW algorithm with field-programmable gate arrays (FPGAs)"

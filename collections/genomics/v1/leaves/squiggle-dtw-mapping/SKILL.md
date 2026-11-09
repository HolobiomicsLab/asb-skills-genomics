---
name: squiggle-dtw-mapping
description: Use when nanopore raw signal reads (BLOW5/FAST5) must be mapped to a
  reference genome FASTA in the signal domain without basecalling, e.g. for Read Until
  adaptive sampling or benchmarking sDTW throughput against software baselines.
when_to_use_negative:
- The input is already basecalled sequence data that can be mapped with sequence aligners
  such as Minimap2
- No Xilinx Kria/MPSOC hardware is available and only the HARU hardware-accelerated
  path is desired (CPU-only sigfish builds are far slower)
- The reads are not ONT raw signals (R9.4 chemistry BLOW5/FAST5); RUscripts-R9/sigfish-haru
  target ONT nanopore squiggles
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: Hardware sDTW accelerator (Verilog HDL core + user-space driver) that speeds
    up squiggle mapping on the Kria MPSoC FPGA
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Example application (sigfish-haru / sigfish-cpu binaries) that invokes the
    sDTW accelerator or CPU fallback for squiggle mapping and emits PAF output
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Pure-Python software baseline for signal mapping (supports Python 3.6+, BLOW5,
    R9.4 chemistry) used as the ~12.52 reads/s comparison point
  repo: https://github.com/beebdev/RUscripts-R9
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:6e8811dfbbda9355ab67a64b6240e2d4e55fc31a2d5ec9310fa5cbcba7fb66c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# squiggle-dtw-mapping

## Summary

Maps nanopore raw signal (squiggles) to a reference genome using subsequence dynamic time warping (sDTW), optionally accelerated by the HARU FPGA accelerator on a Xilinx Kria KV260, to enable real-time signal-domain Read Until selective sequencing.

## When to use

Use when nanopore raw signal reads (BLOW5/FAST5) must be mapped to a reference genome FASTA in the signal domain without basecalling, e.g. for Read Until adaptive sampling or benchmarking sDTW throughput against software baselines.

## When NOT to use

- The input is already basecalled sequence data that can be mapped with sequence aligners such as Minimap2
- No Xilinx Kria/MPSOC hardware is available and only the HARU hardware-accelerated path is desired (CPU-only sigfish builds are far slower)
- The reads are not ONT raw signals (R9.4 chemistry BLOW5/FAST5); RUscripts-R9/sigfish-haru target ONT nanopore squiggles

## Inputs

- Raw nanopore signal reads in BLOW5 format (e.g. reads_0_0.blow5; SARS-CoV-2 dataset of 1.382M reads, Zenodo DOI 10.5281/zenodo.7314838)
- Reference genome FASTA (e.g. nCoV-2019.reference.fasta / MN908947.3)
- HARU accelerator firmware and sigfish-haru/sigfish-cpu binaries (prebuilt release or built from source)

## Outputs

- PAF-format signal mapping alignments (output.paf)
- Measured mapping throughput in reads/s per configuration (software vs. HARU-accelerated)

## How to apply

Build sigfish-haru (git clone --recursive) with or without FPGA acceleration, and obtain the reference genome FASTA (e.g. nCoV-2019.reference.fasta or MN908947.3) and raw signal reads in BLOW5 format (e.g. the 1.382M-read SARS-CoV-2 dataset from Zenodo DOI 10.5281/zenodo.7314838). For the hardware path, install the HARU accelerator on the Kria board with haru_install.sh, load the sDTW firmware via 'xmutil loadapp haru-dtw-firmware', then run './sigfish-haru dtw -g <reference.fasta> -s <reads.blow5> > output.paf'. For a CPU-only baseline, run the same dtw subcommand with the sigfish-cpu binary built via 'make' without fpga=1, and time the run to compute reads/s throughput. The rationale is that signal-domain mapping avoids the compute-intensive basecalling prerequisite of base-domain Read Until; sDTW is the computational bottleneck, hence the FPGA accelerator. Compare throughput against reported baselines (RUscripts-R9 Python ~12.52 reads/s; optimized C ~162 reads/s on a 10-core i9 desktop; ~432 reads/s on a 36-core Xeon server) and inspect the resulting PAF alignments for correctness.

## Related tools

- **HARU** (Hardware sDTW accelerator (Verilog HDL core + user-space driver) that speeds up squiggle mapping on the Kria MPSoC FPGA) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Example application (sigfish-haru / sigfish-cpu binaries) that invokes the sDTW accelerator or CPU fallback for squiggle mapping and emits PAF output) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Pure-Python software baseline for signal mapping (supports Python 3.6+, BLOW5, R9.4 chemistry) used as the ~12.52 reads/s comparison point) — https://github.com/beebdev/RUscripts-R9

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- output.paf exists and contains mapped read records for the input BLOW5 file
- Measured throughput is order-of-magnitude consistent with reported baselines (~12.52 reads/s Python; ~162 reads/s optimized C on 10-core desktop; ~432 reads/s on 36-core server; HARU ~2.5x faster than the optimized 36-core software)
- xmutil listapps confirms haru-dtw-firmware loaded successfully before running sigfish-haru
- sigfish-cpu dtw on the same example data completes but is markedly slower than sigfish-haru, confirming the accelerator is engaged
- Positive throughput values recorded in throughput_baselines.csv, with hardware-dependent deviations flagged for expert review

## Limitations

- Building the core accelerator requires proprietary AMD/Xilinx tools (Vivado 2021.1/2022.2, PetaLinux 2021.1) and is 'not intuitive'
- Throughput baselines are hardware-dependent (10-core i9 desktop vs. 36-core Xeon server), so absolute numbers vary by platform
- Software-only (sigfish-cpu) execution is slow, making full-dataset CPU mapping impractical on resource-constrained devices
- Implementation is a proof-of-concept targeting R9.4 chemistry and the Kria KV260 Vision AI Starter Kit

## Evidence

- [methods] HARU sDTW accelerator usage and output: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [methods] Accelerator loading on the FPGA: "xmutil unloadapp
# load our HARU sDTW accelerator
xmutil loadapp haru-dtw-firmware"
- [abstract] Headline performance result: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [full_text] Software baseline throughput for RUscripts-R9 C implementation: [evidence span withheld — anchor status: not_found]
- [readme] CPU-only fallback comparison from README: "If you wish, you can run the *sigfish-cpu* binary that does not use the hardware accelerator and see how slow it is."

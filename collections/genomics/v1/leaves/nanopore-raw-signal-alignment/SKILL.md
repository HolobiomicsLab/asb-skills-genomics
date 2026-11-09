---
name: nanopore-raw-signal-alignment
description: Use when performing signal-domain mapping of raw nanopore reads (BLOW5/FAST5)
  against a reference FASTA for selective sequencing (Read Until) or when benchmarking
  sDTW alignment throughput on CPU vs FPGA hardware.
when_to_use_negative:
- Input is already basecalled sequence (FASTQ) intended for sequence-domain aligners
  such as Minimap2
- No FPGA/MPSoC hardware is available and real-time Read Until latency is required
  — the CPU-only sigfish-cpu path (~12.52 reads/s Python, ~162 reads/s optimized C
  on a 10-core desktop) may be too slow
- Signal values are scaled above 128 in fixed-point mode, where sDTW data overflow
  degrades mapping accuracy
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: FPGA sDTW accelerator source (Verilog HDL core and user-space device driver)
    that performs the signal alignment on the Kria KV260 MPSoC
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Proof-of-concept application exposing the dtw subcommand that drives the HARU
    accelerator for squiggle mapping; sigfish-cpu is the CPU-only fallback for benchmarking
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Modified RUscripts Python pipeline supporting Python 3.6+, BLOW5 format, and
    R9.4 chemistry, used as the unoptimized software baseline for signal mapping throughput
  repo: https://github.com/beebdev/RUscripts-R9
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:c0b36c773005ef1623908ecf6d4274ed0d6635aef2d2751247d206062841bd31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nanopore-raw-signal-alignment

## Summary

Maps ONT nanopore raw signal (squiggles) directly to a reference genome using subsequence DTW (sDTW), bypassing basecalling, optionally accelerated by the HARU FPGA accelerator on a Xilinx Kria KV260 MPSoC for real-time Read Until.

## When to use

Use when performing signal-domain mapping of raw nanopore reads (BLOW5/FAST5) against a reference FASTA for selective sequencing (Read Until) or when benchmarking sDTW alignment throughput on CPU vs FPGA hardware.

## When NOT to use

- Input is already basecalled sequence (FASTQ) intended for sequence-domain aligners such as Minimap2
- No FPGA/MPSoC hardware is available and real-time Read Until latency is required — the CPU-only sigfish-cpu path (~12.52 reads/s Python, ~162 reads/s optimized C on a 10-core desktop) may be too slow
- Signal values are scaled above 128 in fixed-point mode, where sDTW data overflow degrades mapping accuracy

## Inputs

- Raw nanopore signal reads in BLOW5 format (e.g., reads_0_0.blow5)
- Reference genome FASTA (e.g., nCoV-2019.reference.fasta, MN908947.3)
- 1.382-million-read SARS-CoV-2 dataset (Zenodo DOI 10.5281/zenodo.7314838) for throughput benchmarks

## Outputs

- PAF mapping file (output.paf) of signal-to-reference mapping positions
- Throughput measurements (reads/s) for CPU and FPGA-accelerated baselines
- Runtime-vs-reference-length and accuracy-vs-scaling-factor tables

## How to apply

Build or download the sigfish-haru binary (an early fork of sigfish) that integrates the HARU sDTW accelerator, and flash the prebuilt PetaLinux image onto a micro SD card for the Kria KV260. Load the accelerator with 'xmutil loadapp haru-dtw-firmware', then run './sigfish-haru dtw -g nCoV-2019.reference.fasta -s reads_0_0.blow5 > output.paf' to map raw squiggles against the SARS-CoV-2 reference in the signal domain. To benchmark, run the same dtw subcommand with the CPU-only sigfish-cpu binary (built without fpga=1) and time both runs to compute reads/s throughput. For fixed-point accuracy studies, sweep integer scaling factors (2 to 128) applied to normalized signal values; HARU uses a factor of 32 for ~99% accuracy, while factors above 128 cause sDTW overflow and degraded accuracy.

## Related tools

- **HARU** (FPGA sDTW accelerator source (Verilog HDL core and user-space device driver) that performs the signal alignment on the Kria KV260 MPSoC) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept application exposing the dtw subcommand that drives the HARU accelerator for squiggle mapping; sigfish-cpu is the CPU-only fallback for benchmarking) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Modified RUscripts Python pipeline supporting Python 3.6+, BLOW5 format, and R9.4 chemistry, used as the unoptimized software baseline for signal mapping throughput) — https://github.com/beebdev/RUscripts-R9

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- output.paf exists and contains mapping positions for the reads in the input BLOW5 file against the reference FASTA
- FPGA-accelerated throughput is ~2.5x faster than the optimized multithreaded CPU version on a 36-core Xeon for the SARS-CoV-2 dataset
- Runtimes increase monotonically with reference length (1 kb–295 kb) and the fitted power-law exponent falls within [1.8, 2.2], confirming O(MN) sDTW behavior
- Fixed-point accuracy at scaling factor 2 falls in 75–85% (reported ~80%) and at scaling factor 32 falls in 97–100% (reported ~99%)
- xmutil listapps confirms the haru-dtw-firmware accelerator loaded successfully onto the FPGA

## Limitations

- Building the core accelerator requires proprietary AMD Xilinx software (Vivado/Vitis 2021.1, Petalinux 2021.1) and the build steps are not intuitive; versions lower than 2020.2 do not support Kria
- Throughput depends heavily on host hardware: ~12.52 reads/s (Python baseline), ~162 reads/s (optimized C, 10-core i9 desktop), ~432 reads/s (36-core server)
- Single-thread CPU sDTW query time grows superlinearly (toward O(MN)) with reference length, so CPU alignment becomes progressively slower for large references
- Fixed-point scaling factors above 128 cause data overflow during sDTW, degrading mapping accuracy

## Evidence

- [abstract] HARU on a Xilinx FPGA with 4-core ARM processor is around 2.5x faster than a highly optimized multithreaded software version running on a 36-core Intel Xeon server for a SARS-CoV-2 dataset: "HARU on a Xilinx FPGA embedded with a 4-core ARM processor is around 2.5× faster than a highly optimized multithreaded software version"
- [results] Fixed-point scaling factor trade-off: [evidence span withheld — anchor status: not_found]
- [methods] dtw invocation: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] Accelerator loading: "xmutil unloadapp
    # load our HARU sDTW accelerator
    xmutil loadapp haru-dtw-firmware"
- [readme] Build warning: "The building of the core acccelerator is not intuitive and require proprietary software from AMD Xilinx."

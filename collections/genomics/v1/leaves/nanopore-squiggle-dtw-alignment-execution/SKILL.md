---
name: nanopore-squiggle-dtw-alignment-execution
description: Use when raw nanopore signal data in BLOW5 format must be aligned to
  a reference genome in the signal (squiggle) domain for Read Until / adaptive sampling,
  especially when real-time performance on resource-constrained devices is the goal.
when_to_use_negative:
- When reads have already been basecalled and base-domain sequence alignment is sufficient
  — tools like Minimap2 on FASTQ are the appropriate path, not signal-domain sDTW.
- When no FPGA/MPSoC hardware is available and batch (non-real-time) analysis on a
  server is acceptable — the software-only sDTW path spends over 98% of runtime on
  alignment and will be slow.
- When the input is sequence data (FASTQ/BAM) rather than raw signal (BLOW5) — sDTW
  squiggle mapping requires raw signal as input.
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Sigfish-HARU
  role: Proof-of-concept squiggle-mapping application that invokes the HARU sDTW accelerator
    via the `dtw` subcommand to map BLOW5 signal reads against a FASTA reference
  repo: https://github.com/beebdev/sigfish-haru
- name: HARU
  role: Verilog HDL sDTW core accelerator plus user-space device driver that offloads
    the sDTW computation onto the Kria KV260 FPGA fabric
  repo: https://github.com/beebdev/HARU
- name: MinION
  role: Oxford Nanopore sequencer producing the raw signal reads that are subject
    to sDTW squiggle mapping for Read Until
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:cb1319c12a7048b0fc82e369957a6a5cf5abfb19b6ddd21d756c86551ea641d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - MinION
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nanopore-squiggle-dtw-alignment-execution

## Summary

Executes subsequence dynamic time warping (sDTW) squiggle mapping of raw nanopore signal reads (BLOW5) against a reference genome (FASTA) using sigfish-haru, optionally accelerated by the HARU FPGA sDTW accelerator on a Xilinx Kria KV260, enabling real-time raw-signal Read Until selective sequencing.

## When to use

Use when raw nanopore signal data in BLOW5 format must be aligned to a reference genome in the signal (squiggle) domain for Read Until / adaptive sampling, especially when real-time performance on resource-constrained devices is the goal.

## When NOT to use

- When reads have already been basecalled and base-domain sequence alignment is sufficient — tools like Minimap2 on FASTQ are the appropriate path, not signal-domain sDTW.
- When no FPGA/MPSoC hardware is available and batch (non-real-time) analysis on a server is acceptable — the software-only sDTW path spends over 98% of runtime on alignment and will be slow.
- When the input is sequence data (FASTQ/BAM) rather than raw signal (BLOW5) — sDTW squiggle mapping requires raw signal as input.

## Inputs

- Nanopore raw signal reads in BLOW5 format (e.g., reads_0_0.blow5, or the SARS-CoV-2 dataset, Zenodo DOI 10.5281/zenodo.7314838)
- Reference genome in FASTA format (e.g., nCoV-2019.reference.fasta)
- HARU sDTW accelerator firmware loaded onto the Xilinx Kria KV260 FPGA (optional, for the hardware-accelerated path)

## Outputs

- Squiggle-mapping alignment results in PAF format (output.paf)
- Per-stage runtime breakdown (sDTW alignment vs. preprocessing vs. I/O) if timing instrumentation is enabled

## How to apply

Obtain the raw nanopore reads in BLOW5 format (e.g., the SARS-CoV-2 dataset from Zenodo DOI 10.5281/zenodo.7314838) and a reference FASTA such as nCoV-2019.reference.fasta. Build sigfish-haru from the beebdev/sigfish-haru repository (clone with --recursive) and, for hardware acceleration, install the HARU accelerator on the Kria board via haru_install.sh and load it with `xmutil loadapp haru-dtw-firmware`. Run the sDTW alignment stage with `./sigfish-haru dtw -g <reference.fasta> -s <reads.blow5> > output.paf`; use the sigfish-cpu binary (built without the fpga=1 flag) for the software-only baseline. The rationale is that sDTW signal alignment is the computational bottleneck of raw-signal Read Until — over 98% of software-only runtime — so offloading it to the FPGA reduces the sDTW share to under 64% and yields roughly 2.5x speedup over a 36-core Xeon server.

## Related tools

- **Sigfish-HARU** (Proof-of-concept squiggle-mapping application that invokes the HARU sDTW accelerator via the `dtw` subcommand to map BLOW5 signal reads against a FASTA reference) — https://github.com/beebdev/sigfish-haru
- **HARU** (Verilog HDL sDTW core accelerator plus user-space device driver that offloads the sDTW computation onto the Kria KV260 FPGA fabric) — https://github.com/beebdev/HARU
- **MinION** (Oxford Nanopore sequencer producing the raw signal reads that are subject to sDTW squiggle mapping for Read Until)

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The command exits without error and produces a non-empty output.paf containing squiggle-mapping records.
- With timing instrumentation, the sDTW stage accounts for more than 98% of total runtime in the software-only (sigfish-cpu) run, consistent with the >98% share reported in Fig. 6.
- In the HARU-accelerated run, the sDTW share of runtime drops below 64% for the SARS-CoV-2 dataset.
- The hardware-accelerated run completes roughly 2.5x faster than the optimized multithreaded software version on the SARS-CoV-2 dataset.
- Loading verification via `xmutil listapps` confirms the haru-dtw-firmware accelerator is resident on the FPGA before running sigfish-haru.

## Limitations

- The HARU accelerator requires proprietary AMD Xilinx tooling (Vivado/Vitis 2021.1, PetaLinux 2021.1) to rebuild, and is targeted specifically at the Xilinx Kria KV260 Vision AI Starter Kit (Zynq Ultrascale+ MPSoC).
- Building the core accelerator is described as 'not intuitive' and depends on Xilinx tool versions (2021.1 tested; versions below 2020.2 need upgrading).
- Security caveat: the default PetaLinux image user is root with no password; a password must be set via `passwd` before connecting the board to a network.
- Sigfish-HARU is an early fork of sigfish, so feature coverage may lag upstream sigfish.

## Evidence

- [readme] Example dtw invocation from README quickstart: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] Loading the accelerator onto the FPGA: "# load our HARU sDTW accelerator
    xmutil loadapp haru-dtw-firmware"
- [intro] sDTW as the computational bottleneck offloaded to FPGA: "HARU tackles the computational bottleneck by accelerating the sDTW algorithm with field-programmable gate arrays (FPGAs)"
- [results] Software vs hardware sDTW runtime share: [evidence span withheld — anchor status: not_found]
- [discussion] Repository contents description: "Source code for the HARU accelerator, including the Verilog HDL core accelerator and user-space device driver"

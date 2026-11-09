---
name: sdtw-squiggle-alignment
description: Use when raw nanopore signal reads (BLOW5) must be mapped directly to
  a reference genome (FASTA) without basecalling, especially for Read Until selective
  sequencing or when comparing fixed-point vs. floating-point sDTW mapping accuracy.
when_to_use_negative:
- When a basecalled FASTQ/POA reference alignment via sequence mapping tools like
  Minimap2 is already available and signal-level analysis is not required
- When no FPGA hardware (Kria KV260 / Zynq Ultrascale+ MPSoC with HARU firmware) is
  available — the software sigfish-cpu fallback runs sDTW without the accelerator
  but is slow
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: FPGA sDTW hardware accelerator (Verilog HDL core + user-space device driver)
    performing the squiggle mapping on the Kria KV260
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Proof-of-concept application that integrates the HARU accelerator for squiggle
    mapping via the 'dtw' subcommand
  repo: https://github.com/beebdev/sigfish-haru
- name: Minimap2
  role: Referenced baseline sequence-mapping technique for base-level read mapping,
    contrasted with signal-domain sDTW mapping
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:f3525e223e039120b47cc8a4bf87eb19c57657005b3aac9b533fb601a61e5088
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Minimap2
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# sdtw-squiggle-alignment

## Summary

Maps raw nanopore signal squiggles to a reference genome using the subsequence DTW (sDTW) algorithm, accelerated on FPGA hardware via the HARU accelerator and driven by the sigfish-haru software. This signal-domain alignment avoids compute-intensive basecalling, enabling real-time Read Until selective sequencing on resource-constrained devices.

## When to use

Use when raw nanopore signal reads (BLOW5) must be mapped directly to a reference genome (FASTA) without basecalling, especially for Read Until selective sequencing or when comparing fixed-point vs. floating-point sDTW mapping accuracy.

## When NOT to use

- When a basecalled FASTQ/POA reference alignment via sequence mapping tools like Minimap2 is already available and signal-level analysis is not required
- When no FPGA hardware (Kria KV260 / Zynq Ultrascale+ MPSoC with HARU firmware) is available — the software sigfish-cpu fallback runs sDTW without the accelerator but is slow

## Inputs

- Reference genome in FASTA format (e.g., test_data/nCoV-2019.reference.fasta)
- Nanopore raw signal reads in BLOW5 format (e.g., test_data/reads_0_0.blow5)
- HARU sDTW firmware loaded on the FPGA (haru-dtw-firmware)

## Outputs

- Mapping positions per read in PAF format (output.paf)
- Optional fixed-point accuracy comparison table (e.g., accuracy_vs_scaling_factor.csv)

## How to apply

Load the HARU sDTW accelerator onto the FPGA of the Kria KV260 (Zynq Ultrascale+ MPSoC) with `xmutil loadapp haru-dtw-firmware`, then run sigfish-haru in dtw mode, passing the reference genome (e.g., nCoV-2019.reference.fasta) via -g and the nanopore reads in BLOW5 format (e.g., reads_0_0.blow5) via -s, writing mapping positions to a PAF output. The accelerator implements a memory-efficient sDTW using 16-bit fixed-point arithmetic; normalized signal values are scaled by an integer scaling factor, and a factor of 32 is used because it gives ~99% accuracy versus a 32-bit floating-point baseline, while a factor of 2 drops accuracy to ~80% and factors above 128 cause data overflow that degrades accuracy. To judge correctness, verify the PAF output is produced and that mapping positions agree with a floating-point sDTW baseline (e.g., accuracy at scaling factor 32 in the 97–100% range).

## Related tools

- **HARU** (FPGA sDTW hardware accelerator (Verilog HDL core + user-space device driver) performing the squiggle mapping on the Kria KV260) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept application that integrates the HARU accelerator for squiggle mapping via the 'dtw' subcommand) — https://github.com/beebdev/sigfish-haru
- **Minimap2** (Referenced baseline sequence-mapping technique for base-level read mapping, contrasted with signal-domain sDTW mapping)

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The 'dtw' run produces a PAF output containing mapping positions for every read (e.g., output.paf is non-empty)
- Fixed-point mapping positions at scaling factor 32 agree with the 32-bit floating-point baseline for ~99% of reads (validation window 97–100%)
- Accuracy at scaling factor 2 falls in the 75–85% range (reported ~80%), and 7 scaling-factor rows (2, 4, 8, 16, 32, 64, 128) are tabulated without overflow degradation above 128
- The accelerator is verified as loaded with `xmutil listapps` before running sigfish-haru

## Limitations

- Fixed-point sDTW accuracy degrades above scaling factor 128 due to data overflow, so the scaling factor must stay within the validated range (HARU uses 32)
- Building the core accelerator requires proprietary AMD Xilinx tools (Vivado/Vitis 2021.1 or compatible versions; article also mentions Vivado 2022.2) and targets the Kria KV260 platform
- Sigfish-haru is an early fork of sigfish demonstrating proof-of-concept integration, not a full production squiggle-mapping tool

## Evidence

- [abstract] HARU accelerator for raw signal alignment: [evidence span withheld — anchor status: not_found]
- [methods] dtw invocation with reference and BLOW5 inputs: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] Loading the accelerator onto the FPGA: "xmutil unloadapp
# load our HARU sDTW accelerator
xmutil loadapp haru-dtw-firmware"
- [readme] HARU repository scope and target platform: [evidence span withheld — anchor status: not_found]
- [intro] Signal-domain avoids basecalling bottleneck: "the prerequisite basecalling step is compute-intensive and is a significant bottleneck for Read Until implementations"

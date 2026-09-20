---
name: subsequence-dynamic-time-warping-implementation
description: Use when mapping raw nanopore signal reads against a reference converted
  to expected signal levels for selective sequencing (Read Until), where only the
  query location's end position and normalized sDTW score are needed and O(MN) memory
  is prohibitive.
when_to_use_negative:
- You need full alignment paths or start positions requiring backtracking — the collapsed
  cost array discards the information needed to recover the starting position
- Base-level alignment of already-basecalled reads is sufficient — sequence mappers
  like Minimap2 operate on base sequences, not raw signal
- You require the full O(MN) cost matrix for visualization or posterior analysis rather
  than just end position and score
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: FPGA-accelerated sDTW hardware accelerator (Verilog HDL core + device driver)
    implementing the memory-optimized subsequence DTW on Xilinx Kria KV260
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Software application driving the HARU accelerator for squiggle mapping; provides
    the dtw subcommand and test data
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:ee175d07a7419812920c456db00b9e34a2761ec3bbe99ac7660acba0cab0ac5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# subsequence-dynamic-time-warping-implementation

## Summary

Implements memory-optimized subsequence dynamic time warping (sDTW) for nanopore squiggle mapping, collapsing the full M × N cost matrix to a 1-D cost array of size M+1 because Read Until only needs the alignment end position, not backtracking to recover the start.

## When to use

Use when mapping raw nanopore signal reads against a reference converted to expected signal levels for selective sequencing (Read Until), where only the query location's end position and normalized sDTW score are needed and O(MN) memory is prohibitive.

## When NOT to use

- You need full alignment paths or start positions requiring backtracking — the collapsed cost array discards the information needed to recover the starting position
- Base-level alignment of already-basecalled reads is sufficient — sequence mappers like Minimap2 operate on base sequences, not raw signal
- You require the full O(MN) cost matrix for visualization or posterior analysis rather than just end position and score

## Inputs

- Reference genome FASTA (e.g., nCoV-2019.reference.fasta, SARS-CoV-2 MN908947.3)
- R9.4 k-mer model for converting sequence to expected signal levels (length M)
- Raw nanopore signal reads in BLOW5 format (e.g., reads_0_0.blow5)

## Outputs

- Per-read mapping results with query end position, reference end position, and minimum normalized sDTW score (e.g., mapping_positions.tsv with columns read_id, end_position, score)
- PAF-style output from sigfish-haru dtw (output.paf)

## How to apply

Convert the reference genome (e.g., SARS-CoV-2 MN908947.3 from nCoV-2019.reference.fasta) into expected signal levels of length M using the R9.4 k-mer model, then read raw signal from the BLOW5 file (e.g., reads_0_0.blow5). For each read, run the sDTW recurrence with the cost matrix collapsed to a 1-D cost array of size M+1 instead of a full O(MN) matrix, streaming the outer loop along the reference and updating the array row by row. Keep only the running minimum normalized alignment score and its corresponding reference end position, since backtracking to recover the starting position is unnecessary for Read Until decisions. Record per-read query end position, reference end position, and minimum normalized sDTW score. This reduction is what allows HARU's FPGA accelerator to run sDTW within the memory constraints of the Kria KV260's Zynq Ultrascale+ MPSoC.

## Related tools

- **HARU** (FPGA-accelerated sDTW hardware accelerator (Verilog HDL core + device driver) implementing the memory-optimized subsequence DTW on Xilinx Kria KV260) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Software application driving the HARU accelerator for squiggle mapping; provides the dtw subcommand and test data) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- End positions and minimum normalized sDTW scores match those of a full O(MN) sDTW reference implementation within floating-point tolerance
- Script runs end-to-end on the example SARS-CoV-2 dataset (nCoV-2019.reference.fasta + reads_0_0.blow5) without memory errors
- Output file mapping_positions.tsv contains the required columns read_id, end_position, and score for every read processed
- Peak memory usage stays O(M) (cost array of size M+1) rather than O(MN) during the alignment recurrence
- sigfish-haru output (output.paf) is produced successfully on the Kria board after loading haru-dtw-firmware via xmutil

## Limitations

- Backtracking is impossible — only the alignment end position is retained, so starting positions cannot be recovered from the reduced data structure
- The proof-of-concept targets the Xilinx Kria KV260 (Zynq Ultrascale+ MPSoC) with R9.4 chemistry; building the accelerator requires proprietary Vivado/PetaLinux tools (2021.1/2022.2)
- Signal-domain Read Until assumes the hypothesis that signal-domain alignment outperforms base-domain approaches only if sufficient optimization and acceleration is applied

## Evidence

- [methods] Cost array reduction rationale: [evidence span withheld — anchor status: not_found]
- [abstract] Article abstract on memory-efficient accelerator: "We present Hardware Accelerated Read Until (HARU) (Fig. 1), a software–hardware codesign system for raw signal-alignment Read Until that uses the memory-efficient sDTW hardware accelerator"
- [readme] README quickstart invocation: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] README accelerator description: "Our current proof-of-concept implementation of HARU utilises a custom subsequence DTW accelerator primarily targeted for a [Xilinx's Kria AI Starter Kit]"

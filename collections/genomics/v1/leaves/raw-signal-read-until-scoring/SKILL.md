---
name: raw-signal-read-until-scoring
description: Use when raw nanopore signal reads (BLOW5) must be mapped to expected
  signal levels of a reference genome in real time for selective sequencing, and only
  the alignment end position and normalized sDTW score are required rather than a
  full alignment path.
when_to_use_negative:
- When the full alignment path or query start position is required (backtracking is
  deliberately omitted in the M+1 cost-array formulation)
- When basecalled sequence (FASTQ) is already available and base-domain Read Until
  or Minimap2-based sequence mapping is preferred over raw-signal alignment
- When a full O(MN) reference implementation's complete cost matrix is the target
  output rather than end positions and scores
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: HARU
  role: Hardware sDTW accelerator (Verilog HDL core and device driver) that computes
    the memory-optimized sDTW scoring for Read Until on a Xilinx Kria FPGA
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Proof-of-concept application that integrates the HARU accelerator for nanopore
    squiggle mapping; provides the dtw subcommand used for scoring
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:5124c90c5e646951a9ebc9f48f50fca8fd4d498ae0fc7d725405c4845b7fb899
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# raw-signal-read-until-scoring

## Summary

Computes subsequence dynamic time warping (sDTW) alignment scores between raw nanopore signal reads and expected signal levels derived from an R9.4 k-mer model, to support Read Until decisions. HARU reduces the full M × N cost matrix to a 1-D cost array of size M+1 because only the alignment end position—not backtracking—is needed for selective sequencing.

## When to use

Use when raw nanopore signal reads (BLOW5) must be mapped to expected signal levels of a reference genome in real time for selective sequencing, and only the alignment end position and normalized sDTW score are required rather than a full alignment path.

## When NOT to use

- When the full alignment path or query start position is required (backtracking is deliberately omitted in the M+1 cost-array formulation)
- When basecalled sequence (FASTQ) is already available and base-domain Read Until or Minimap2-based sequence mapping is preferred over raw-signal alignment
- When a full O(MN) reference implementation's complete cost matrix is the target output rather than end positions and scores

## Inputs

- Reference genome FASTA (nCoV-2019.reference.fasta, SARS-CoV-2 MN908947.3)
- R9.4 k-mer model file for converting reference bases to expected signal levels
- Raw nanopore signal reads in BLOW5 format (reads_0_0.blow5)

## Outputs

- mapping_positions.tsv with columns read_id, end_position, score
- Per-read minimum normalized sDTW score and reference end position (optionally emitted as output.paf by sigfish-haru)

## How to apply

Load the R9.4 k-mer model and convert the reference genome (e.g., SARS-CoV-2 MN908947.3 from nCoV-2019.reference.fasta) into expected signal levels of length M. Read raw nanopore signal reads from the BLOW5 file (e.g., reads_0_0.blow5). For each read, implement the sDTW recurrence with the cost matrix collapsed to a 1-D cost array of size M+1 instead of a full O(MN) matrix: stream the outer loop along the reference sequence, updating the cost array row by row while keeping only the running minimum normalized alignment score and its corresponding reference end position. Rationale: for Read Until, the alignment end position suffices to determine the query location, so backtracking to recover the starting position is unnecessary—this removes the O(MN) memory requirement that makes software sDTW the computational bottleneck. Record query end position, reference end position, and minimum normalized sDTW score per read, then write per-read results to mapping_positions.tsv with columns read_id, end_position, and score.

## Related tools

- **HARU** (Hardware sDTW accelerator (Verilog HDL core and device driver) that computes the memory-optimized sDTW scoring for Read Until on a Xilinx Kria FPGA) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept application that integrates the HARU accelerator for nanopore squiggle mapping; provides the dtw subcommand used for scoring) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- The reduced-memory script runs end-to-end on the example data (nCoV-2019.reference.fasta + reads_0_0.blow5) without error
- End positions and scores match those of a full O(MN) sDTW reference implementation within floating-point tolerance
- Cost array length is M+1 (where M is the number of expected signal levels), not O(MN), confirming the memory optimization
- Output mapping_positions.tsv contains exactly the columns read_id, end_position, score with one row per read
- Each read's reported reference end position corresponds to the running minimum normalized sDTW score over streamed reference rows

## Limitations

- No backtracking is performed, so the query/alignment start position cannot be recovered from the cost-array formulation
- The proof-of-concept HARU accelerator is targeted at a Xilinx Kria KV260 (Zynq Ultrascale+ MPSoC); building the core accelerator requires proprietary AMD Xilinx tools (Vivado/Vitis 2021.1)
- Prebuilt workflow depends on a custom PetaLinux 2021.1 image; on-board accounts should set a password before network access for security

## Evidence

- [other] HARU reduces the sDTW cost matrix to a cost array of size M + 1 because for selective sequencing the alignment end position suffices to determine the query location, making backtracking to recover the starting position unnecessary.: [evidence span withheld — anchor status: not_found]
- [intro] HARU, a software–hardware codesign system for raw signal-alignment Read Until that uses the memory-efficient sDTW hardware accelerator: "software–hardware codesign system for raw signal-alignment Read Until that uses the memory-efficient sDTW hardware accelerator"
- [methods] ./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] This repository contains the source code for this accelerator, including the Verilog HDL core accelerator and user space device driver.: [evidence span withheld — anchor status: not_found]

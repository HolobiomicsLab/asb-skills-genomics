---
name: memory-constrained-dtw-cost-array-optimization
description: Use when implementing sDTW-based squiggle mapping for selective sequencing
  (Read Until) on memory-constrained devices, where only the alignment end position
  — not a backtracked alignment path — is needed to localize a read.
when_to_use_negative:
- When a full alignment path or start position on the reference is required — collapsing
  to a 1-D cost array forfeits backtracking, so only the end position is recoverable.
- When memory is not a constraint and a full M×N sDTW matrix implementation with traceback
  is already acceptable, since the reduction trades recoverable alignment information
  for memory.
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0080
tools:
- name: HARU
  role: Source implementation of the memory-efficient sDTW hardware accelerator; the
    cost-array optimization mirrors what the Verilog HDL core performs on the FPGA
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Example application that invokes the HARU sDTW accelerator on reference FASTA
    and BLOW5 signal reads for squiggle mapping
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:22d345a855b3cd7342462a79ff71ab2231c93ea22e1c7c06c898b93d6f912d2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# memory-constrained-dtw-cost-array-optimization

## Summary

Collapses the full O(M×N) subsequence DTW (sDTW) cost matrix into a 1-D cost array of size M+1 so raw nanopore signal reads can be aligned to reference expected signal levels with O(M) memory, keeping only the running minimum score and its reference end position for Read Until decisions.

## When to use

Use when implementing sDTW-based squiggle mapping for selective sequencing (Read Until) on memory-constrained devices, where only the alignment end position — not a backtracked alignment path — is needed to localize a read.

## When NOT to use

- When a full alignment path or start position on the reference is required — collapsing to a 1-D cost array forfeits backtracking, so only the end position is recoverable.
- When memory is not a constraint and a full M×N sDTW matrix implementation with traceback is already acceptable, since the reduction trades recoverable alignment information for memory.

## Inputs

- Reference genome FASTA (e.g., nCoV-2019.reference.fasta, SARS-CoV-2 MN908947.3)
- R9.4 k-mer model for converting the reference to expected signal levels of length M
- Raw nanopore signal reads in BLOW5 format (e.g., reads_0_0.blow5)

## Outputs

- Per-read alignment end position and normalized sDTW score (e.g., mapping_positions.tsv with read_id, end_position, score columns)

## How to apply

Convert the reference genome (e.g., SARS-CoV-2 MN908947.3 from nCoV-2019.reference.fasta) into expected signal levels of length M using the R9.4 k-mer model, then stream raw nanopore reads from the BLOW5 file (e.g., reads_0_0.blow5). For each read, run the sDTW recurrence with the cost matrix collapsed to a 1-D cost array of size M+1 instead of a full M×N matrix, streaming the outer loop along the reference sequence row by row. Because for selective sequencing the alignment end position alone suffices to determine the query location, discard backtracking entirely and keep only the running minimum normalized sDTW score and its corresponding reference end position. Record the query end position, reference end position, and minimum normalized score per read (e.g., to mapping_positions.tsv with read_id, end_position, score). Validate against a full O(MN) sDTW reference implementation; end positions and scores must match within floating-point tolerance.

## Related tools

- **HARU** (Source implementation of the memory-efficient sDTW hardware accelerator; the cost-array optimization mirrors what the Verilog HDL core performs on the FPGA) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Example application that invokes the HARU sDTW accelerator on reference FASTA and BLOW5 signal reads for squiggle mapping) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- End positions and scores from the cost-array implementation match those of a full O(MN) sDTW reference implementation within floating-point tolerance.
- Peak memory for the DTW core scales as O(M+1) (the cost array length), not O(M×N).
- The script runs end-to-end on the example data (nCoV-2019.reference.fasta + reads_0_0.blow5) and writes per-read rows to mapping_positions.tsv with read_id, end_position, and score columns.
- Each read yields a running minimum normalized sDTW score paired with a valid reference end position in [0, M].

## Limitations

- Only the alignment end position is recovered; no backtracking means no alignment path or start position, which is acceptable for Read Until but not for full alignment reporting (e.g., PAF with coordinates).
- Scores are floating-point dependent, so exact equality with a reference implementation should not be expected — compare within floating-point tolerance.
- This skill concerns the software-side algorithmic reduction; reproducing HARU's hardware acceleration additionally requires the Kria KV260 setup, Vivado, and PetaLinux toolchain.

## Evidence

- [other] HARU reduces the sDTW cost matrix to a cost array of size M + 1: [evidence span withheld — anchor status: not_found]
- [intro] HARU tackles the computational bottleneck by accelerating the sDTW algorithm with field-programmable gate arrays (FPGAs): "HARU tackles the computational bottleneck by accelerating the sDTW algorithm with field-programmable gate arrays (FPGAs)"
- [readme] Example data invocation from README: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] HARU is a heterogeneous compute solution for Read Until: "HARU is a heterogenous compute solution for Oxford Nanopore Technologies' adaptive sampling (also known as selective sequencing and Read Until)"

---
name: nanopore-squiggle-mapping
description: Use when raw nanopore signal reads in BLOW5 format must be aligned directly
  against a reference genome at the signal level for Read Until / adaptive sampling
  decisions, especially when basecalling is too slow or resource-intensive.
when_to_use_negative:
- Reads are already basecalled to FASTQ/FASTA — use sequence mapping (e.g., Minimap2)
  instead, since base-domain alignment is sufficient and squiggle mapping is unnecessary
- The task requires the full alignment path or start position via backtracking — the
  memory-optimized M+1 cost array deliberately discards backtracking information
- No Xilinx Kria KV260/FPGA platform (or proprietary Vivado/PetaLinux toolchain) is
  available and CPU sDTW speed (sigfish-cpu) is too slow for real-time Read Until
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: HARU
  role: FPGA sDTW hardware accelerator (Verilog HDL core plus user-space driver) that
    performs the memory-efficient squiggle alignment for Read Until
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: Proof-of-concept squiggle mapping application that drives the HARU accelerator;
    its 'dtw' subcommand maps BLOW5 signal reads against a FASTA reference
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Modified Read Until scripts supporting Python 3.6+, BLOW5 format, and ONT
    R9.4 chemistry for the selective sequencing context
  repo: https://github.com/beebdev/RUscripts-R9
- name: Minimap2
  role: Contrast/base-domain alternative for mapping basecalled sequences; mentioned
    in the article as the base-sequence mapping approach
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:4346f4687bb2211293a1147beb2ce768923ecf495ac6edd9b021f63c5d46fb8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - Minimap2
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nanopore-squiggle-mapping

## Summary

Map raw Oxford Nanopore signal reads (squiggles) directly to a reference genome using subsequence dynamic time warping (sDTW) via the HARU accelerator and sigfish-haru software, enabling signal-domain Read Until selective sequencing without basecalling.

## When to use

Use when raw nanopore signal reads in BLOW5 format must be aligned directly against a reference genome at the signal level for Read Until / adaptive sampling decisions, especially when basecalling is too slow or resource-intensive.

## When NOT to use

- Reads are already basecalled to FASTQ/FASTA — use sequence mapping (e.g., Minimap2) instead, since base-domain alignment is sufficient and squiggle mapping is unnecessary
- The task requires the full alignment path or start position via backtracking — the memory-optimized M+1 cost array deliberately discards backtracking information
- No Xilinx Kria KV260/FPGA platform (or proprietary Vivado/PetaLinux toolchain) is available and CPU sDTW speed (sigfish-cpu) is too slow for real-time Read Until

## Inputs

- Reference genome in FASTA format (e.g., test_data/nCoV-2019.reference.fasta, SARS-CoV-2 MN908947.3)
- Raw nanopore signal reads in BLOW5 format (e.g., test_data/reads_0_0.blow5)
- ONT R9.4 k-mer model for converting sequence to expected signal levels

## Outputs

- Per-read mapping records: read_id, query/reference end_position, and normalized sDTW score (e.g., mapping_positions.tsv)
- PAF-formatted squiggle mapping output (output.paf) from sigfish-haru dtw

## How to apply

Load the R9.4 k-mer model and convert the reference genome (e.g., SARS-CoV-2 MN908947.3 supplied as nCoV-2019.reference.fasta) into expected signal levels of length M. Read raw nanopore signal reads from the BLOW5 file (e.g., reads_0_0.blow5). For each read, run the sDTW recurrence with the cost matrix collapsed to a 1-D cost array of size M+1 rather than a full O(MN) matrix: stream the outer loop along the reference sequence, updating the cost array row by row and retaining only the running minimum normalized sDTW score and its reference end position, because for selective sequencing the alignment end position suffices and backtracking to recover the start position is unnecessary. Record per-read query end position, reference end position, and minimum normalized score, then emit results (e.g., as PAF via sigfish-haru, or mapping_positions.tsv with read_id, end_position, score). Correctness is judged by agreement of end positions and scores with a full O(MN) sDTW reference implementation within floating-point tolerance, and, on Kria hardware, by comparing hardware-accelerated output against the sigfish-cpu binary.

## Related tools

- **HARU** (FPGA sDTW hardware accelerator (Verilog HDL core plus user-space driver) that performs the memory-efficient squiggle alignment for Read Until) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept squiggle mapping application that drives the HARU accelerator; its 'dtw' subcommand maps BLOW5 signal reads against a FASTA reference) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Modified Read Until scripts supporting Python 3.6+, BLOW5 format, and ONT R9.4 chemistry for the selective sequencing context) — https://github.com/beebdev/RUscripts-R9
- **Minimap2** (Contrast/base-domain alternative for mapping basecalled sequences; mentioned in the article as the base-sequence mapping approach)

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- End positions and normalized sDTW scores match a full O(MN) sDTW reference implementation within floating-point tolerance
- sigfish-haru dtw on test_data/nCoV-2019.reference.fasta and test_data/reads_0_0.blow5 produces a non-empty output.paf consistent with sigfish-cpu results
- Peak memory scales with O(M+1) cost array, not a full M x N cost matrix
- Per-read results contain well-formed columns read_id, end_position, and score with finite normalized scores
- xmutil listapps confirms haru-dtw-firmware was loaded successfully onto the FPGA before running the accelerated path

## Limitations

- The accelerator targets the Xilinx Kria KV260 Vision AI Starter Kit (Zynq Ultrascale+ MPSoC); building the core requires proprietary tools (Vivado 2021.1, PetaLinux 2021.1) and the README warns 'The building of the core acccelerator is not intuitive and require proprietary software from AMD Xilinx'
- Only the alignment end position and minimum score are retained; no backtracking, so the alignment start position cannot be recovered
- Implementation is demonstrated for R9.4 chemistry and squiggle mapping on a SARS-CoV-2 dataset; the sigfish-haru README describes it as 'an early fork of sigfish' demonstrating proof-of-concept integration
- When connected to a network, the Kria root account should have a password set ('make sure at least you set a password using `passwd` command')

## Evidence

- [methods] Cost-array sDTW rationale (no backtracking): [evidence span withheld — anchor status: not_found]
- [methods] Example dtw invocation on SARS-CoV-2 data: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [abstract] HARU purpose for Read Until: "we present Hardware Accelerated Read Until (HARU), a resource-efficient hardware-software codesign-based method"
- [readme] Accelerator contents in repository: "This repository contains the source code for this accelerator, including the [Verilog HDL core accelerator](https://github.com/beebdev/HARU/tree/main/hdl) and [user space device"
- [readme] Loading the sDTW firmware: "# load our HARU sDTW accelerator
    xmutil loadapp haru-dtw-firmware"

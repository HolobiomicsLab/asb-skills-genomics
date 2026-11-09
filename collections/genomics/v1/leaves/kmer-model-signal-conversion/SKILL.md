---
name: kmer-model-signal-conversion
description: 'Use when performing raw-signal (squiggle) alignment for selective sequencing:
  you have a FASTA reference (e.g., SARS-CoV-2 MN908947.3), an R9.4 pore chemistry
  k-mer model, and raw reads in BLOW5 format to align in the signal domain.'
when_to_use_negative:
- Reads are already basecalled and a base-domain aligner such as Minimap2 on the basecalls
  is sufficient — the article frames this skill specifically for raw-signal Read Until
  to avoid the compute-intensive basecalling step
- Signal data come from a pore chemistry other than R9.4, since the k-mer model's
  expected current levels would not match the recorded signals
- A full alignment path (start position via backtracking) is required — HARU's cost-array
  design deliberately discards the O(MN) matrix and only returns the alignment end
  position
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: HARU
  role: Consumes the expected-signal reference produced by the k-mer model conversion
    and runs the memory-optimized sDTW hardware accelerator against raw BLOW5 signals
    for Read Until decisions
  repo: https://github.com/beebdev/HARU
- name: Sigfish-HARU
  role: 'Proof-of-concept application that drives squiggle mapping: takes the k-mer-derived
    reference (-g) and BLOW5 signals (-s) and emits output.paf'
  repo: https://github.com/beebdev/sigfish-haru
- name: Minimap2
  role: Alternative base-domain mapping approach mentioned by the article that this
    signal-domain skill avoids by converting the reference to expected signal levels
    instead
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:7085146b9b2e79be2d38728a76c7ffbbbacb2c14c3496611ff4c2fcb343fddbb
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# k-mer model signal conversion

## Summary

Converts a nucleotide reference genome into expected ONT current (signal) levels using the R9.4 k-mer model, producing a reference-length expected-signal sequence that can be aligned directly to raw nanopore squiggles with sDTW — the step that makes signal-domain Read Until possible without basecalling.

## When to use

Use when performing raw-signal (squiggle) alignment for selective sequencing: you have a FASTA reference (e.g., SARS-CoV-2 MN908947.3), an R9.4 pore chemistry k-mer model, and raw reads in BLOW5 format to align in the signal domain.

## When NOT to use

- Reads are already basecalled and a base-domain aligner such as Minimap2 on the basecalls is sufficient — the article frames this skill specifically for raw-signal Read Until to avoid the compute-intensive basecalling step
- Signal data come from a pore chemistry other than R9.4, since the k-mer model's expected current levels would not match the recorded signals
- A full alignment path (start position via backtracking) is required — HARU's cost-array design deliberately discards the O(MN) matrix and only returns the alignment end position

## Inputs

- Reference genome in FASTA format (nCoV-2019.reference.fasta, accession MN908947.3 for SARS-CoV-2)
- R9.4 nanopore pore-chemistry k-mer model (k-mer -> expected current level table)
- Raw nanopore signal reads in BLOW5 format (e.g., reads_0_0.blow5 from the sample dataset)

## Outputs

- Expected-signal-level reference sequence of length M derived from the k-merized reference genome
- Per-read mapping results (alignment end position, normalized sDTW score), written as output.paf by sigfish-haru or as mapping_positions.tsv with columns read_id, end_position, and score

## How to apply

Load the R9.4 k-mer model and convert each k-mer of the reference genome (e.g., SARS-CoV-2 MN908947.3, provided as nCoV-2019.reference.fasta) into its expected current level, concatenating into an expected-signal sequence of length M. This reference-level signal is the query target for the sDTW recurrence in HARU's Read Until pipeline, so its accuracy gates every downstream alignment score. Raw reads are then read from the BLOW5 file (e.g., reads_0_0.blow5) and aligned against the expected-signal reference; HARU's memory-optimized sDTW keeps only a cost array of size M+1 and tracks the running minimum normalized score with its reference end position, since Read Until only needs the alignment end position, not backtracking. The correct R9.4 model must match the pore chemistry in the BLOW5 file or expected levels (and hence scores and end positions) will be wrong.

## Related tools

- **HARU** (Consumes the expected-signal reference produced by the k-mer model conversion and runs the memory-optimized sDTW hardware accelerator against raw BLOW5 signals for Read Until decisions) — https://github.com/beebdev/HARU
- **Sigfish-HARU** (Proof-of-concept application that drives squiggle mapping: takes the k-mer-derived reference (-g) and BLOW5 signals (-s) and emits output.paf) — https://github.com/beebdev/sigfish-haru
- **Minimap2** (Alternative base-domain mapping approach mentioned by the article that this signal-domain skill avoids by converting the reference to expected signal levels instead)

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- End positions and normalized sDTW scores from the reduced cost-array implementation match a full O(MN) sDTW reference implementation within floating-point tolerance
- The expected-signal reference has length M consistent with k-merization of the reference genome under the R9.4 model (M k-mers -> M expected levels)
- Running sigfish-haru on the bundled example data (nCoV-2019.reference.fasta + reads_0_0.blow5) produces a non-empty output.paf
- Every read in the BLOW5 file yields a row with a valid end position (within reference bounds) and a finite normalized score in mapping_positions.tsv

## Limitations

- The k-mer model is pore-chemistry specific (R9.4); mismatched chemistry invalidates expected signal levels
- The HARU sDTW design returns only the alignment end position — backtracking to recover the start position is intentionally not supported for Read Until
- Building the hardware path requires proprietary AMD Xilinx tooling (Vivado/Vitis 2021.1, PetaLinux 2021.1) and a Kria KV260 board; no changelog is published

## Evidence

- [other] Load the R9.4 k-mer model and convert the SARS-CoV-2 reference genome (MN908947.3, provided as nCoV-2019.reference.fasta) into expected signal levels of length M.: [evidence span withheld — anchor status: not_found]
- [other] HARU reduces the sDTW cost matrix to a cost array of size M + 1 because for selective sequencing the alignment end position suffices: [evidence span withheld — anchor status: not_found]
- [methods] ./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [readme] Our current proof-of-concept implementation of HARU utilises a custom subsequence DTW accelerator primarily targeted for a Xilinx's Kria AI Starter Kit: "Our current proof-of-concept implementation of HARU utilises a custom subsequence DTW accelerator primarily targeted for a [Xilinx's Kria AI Starter Kit]"

---
name: basecalling-with-dorado
description: Use when you have simulated raw nanopore signals (BLOW5/POD5) and need
  nucleotide FASTQ/FASTA reads to benchmark the simulator against basecalling accuracy
  metrics such as median match rate and median PHRED score.
when_to_use_negative:
- Input is already basecalled nucleotide reads (FASTQ) — skip directly to alignment-based
  error profiling
- The goal is to evaluate raw-signal fidelity against real reads rather than basecalling-derived
  accuracy metrics
- Signals were generated for a chemistry for which no compatible dorado model is available
  (e.g. a custom-trained seq2squiggle profile without a matching dorado model)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dorado
  role: Basecalls the simulated squiggle signals to produce nucleotide reads with
    PHRED quality scores for benchmark evaluation
- name: minimap2
  role: Aligns dorado basecalls back to the reference genomes to recover per-read
    alignment records for error-rate computation
- name: seq2squiggle
  role: Generates the simulated nanopore signals (BLOW5/POD5) in Read-mode and Genome-mode
    that are basecalled in this workflow
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Comparison simulator whose signals are basecalled with dorado to provide the
    baseline median match rate (87.71–93.00) and PHRED (8.91–11.91) ranges
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:93dfa058e3e229e679b053cdac1b9f73d7debee05c5538065b9c2e82df401965
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dorado
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# basecalling-with-dorado

## Summary

Basecalls simulated nanopore squiggle signals (e.g. seq2squiggle/squigulator BLOW5/POD5 output for R10.4.1 chemistry) into nucleotide reads with dorado, so downstream alignment can quantify simulation accuracy as match rate and PHRED score.

## When to use

Use when you have simulated raw nanopore signals (BLOW5/POD5) and need nucleotide FASTQ/FASTA reads to benchmark the simulator against basecalling accuracy metrics such as median match rate and median PHRED score.

## When NOT to use

- Input is already basecalled nucleotide reads (FASTQ) — skip directly to alignment-based error profiling
- The goal is to evaluate raw-signal fidelity against real reads rather than basecalling-derived accuracy metrics
- Signals were generated for a chemistry for which no compatible dorado model is available (e.g. a custom-trained seq2squiggle profile without a matching dorado model)

## Inputs

- Simulated nanopore raw signals in SLOW5/BLOW5 or POD5 format (e.g. from seq2squiggle or squigulator)
- Reference genome FASTA files (Human HCT116, D. melanogaster, E. coli R10.4.1) for downstream minimap2 alignment
- Dorado-compatible basecalling model configuration for the target chemistry

## Outputs

- Basecalled nucleotide reads (FASTQ/FASTA) with per-read PHRED quality scores for every simulator/mode/organism combination
- Per-read alignment records (via minimap2) used to derive match/mismatch/insertion/deletion rates and median PHRED score
- CSV benchmark table per dataset comparing median match rate and median PHRED score across simulators

## How to apply

In the seq2squiggle benchmark design, squiggle signals simulated in Read-mode and Genome-mode for each organism (Human HCT116, D. melanogaster, E. coli) on R10.4.1 chemistry are passed through dorado to obtain nucleotide reads for every simulator/mode/organism combination. Align the dorado basecalls back to the same reference genome FASTA with minimap2 to recover per-read alignment records. From those records and the basecall quality strings, compute the median match rate, mismatch rate, insertion rate, deletion rate, and median PHRED score per combination. Judge correctness by assembling these into one CSV benchmark table per dataset and checking that seq2squiggle results fall in the reported ranges (match rate 91.91–95.29, PHRED 11.61–13.67) versus squigulator (87.71–93.00 match rate, PHRED 8.91–11.91).

## Related tools

- **dorado** (Basecalls the simulated squiggle signals to produce nucleotide reads with PHRED quality scores for benchmark evaluation)
- **minimap2** (Aligns dorado basecalls back to the reference genomes to recover per-read alignment records for error-rate computation)
- **seq2squiggle** (Generates the simulated nanopore signals (BLOW5/POD5) in Read-mode and Genome-mode that are basecalled in this workflow) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Comparison simulator whose signals are basecalled with dorado to provide the baseline median match rate (87.71–93.00) and PHRED (8.91–11.91) ranges)

## Evaluation signals

- Per simulator/mode/organism combination, dorado yields a non-empty basecalled FASTQ with one record per simulated signal
- Median match rate for seq2squiggle-generated reads falls within the reported 91.91–95.29 range on R10.4.1 datasets (e.g. 91.91 human Read-mode, 95.29 E. coli Genome-mode)
- Median PHRED score for seq2squiggle reads falls within 11.61–13.67, exceeding squigulator's 8.91–11.91 range
- One benchmark CSV table is produced per dataset with match rate, mismatch rate, insertion rate, deletion rate, and median PHRED columns
- Basecalls align cleanly against the same reference FASTA used for simulation (minimap2 records recoverable for nearly all reads)

## Limitations

- Basecalling accuracy depends on a dorado model compatible with the simulated chemistry (R10.4.1); mismatched models will deflate match rate and PHRED
- For GPU-scale workloads, seq2squiggle inference is recommended on GPU; CPU-only runs are slower, and 100,000 reads per organism are required per benchmark cell
- Reported ranges (match rate 91.91–95.29, PHRED 11.61–13.67) apply to R10.4.1 Read-mode/Genome-mode benchmarks and may not generalize to other chemistries or simulators

## Evidence

- [other] Basecalling workflow grounded in the benchmark design: [evidence span withheld — anchor status: not_found]
- [other] Metrics computed from alignments of basecalls: [evidence span withheld — anchor status: not_found]
- [abstract] Simulator quality claim motivating the benchmark: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
- [intro] Nature of the simulated input signals: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."

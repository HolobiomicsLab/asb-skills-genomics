---
name: synthetic-read-generation
description: Use when you need to simulate nanopore raw signals from existing FASTA/FASTQ
  reads or a reference genome — e.g. to create synthetic R9.4.1/R10.4.1 chemistry
  data for benchmarking signal-comparison or basecalling pipelines.
when_to_use_negative:
- You need basecalled sequences or variant-level simulation rather than raw signal
  squiggles — seq2squiggle outputs signal data, not consensus reads.
- The goal is to preserve exact read sequences while adding signal variability but
  splitting reads is required — Read mode samples input reads without splitting them.
- Static k-mer-model simulation fidelity is explicitly acceptable and a lightweight
  CPU tool is preferred over a transformer model with model-weight downloads.
edam_operation: http://edamontology.org/operation_0283
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: seq2squiggle
  role: Transformer-based, non-autoregressive simulator that generates nanopore sequencing
    signals from nucleotide sequences; invoked via `seq2squiggle predict`
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Static k-mer-model-based simulator used as the baseline comparator whose signals
    show larger median DTW deviation (10.01) from real reads
- name: uncalled4
  role: Generates event-level signal alignments (eventalign tables) that can serve
    as segmented training data for training a new seq2squiggle model
  repo: https://github.com/skovaka/uncalled4
- name: f5c
  role: Alternative to uncalled4 for producing eventalign training data (`f5c eventalign
    --samples --signal-index`)
  repo: https://github.com/hasindu2008/f5c
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:0bef6947422c6b54cb68e3b932cd6a3e2ca4dea07efe510b2839adf56551d034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - Python
  - f5c
  - linmdtw
  - mrmsdtw
  - seq2squiggle
  - squigulator
  - uncalled4
  edam_operation: http://edamontology.org/operation_0283
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# synthetic-read-generation

## Summary

Generate artificial nanopore sequencing signals (raw squiggles in BLOW5/POD5) from nucleotide sequences using seq2squiggle, a transformer-based, non-autoregressive simulator, so that simulated signals can be benchmarked against real experimental reads (e.g. via DTW distance).

## When to use

Use when you need to simulate nanopore raw signals from existing FASTA/FASTQ reads or a reference genome — e.g. to create synthetic R9.4.1/R10.4.1 chemistry data for benchmarking signal-comparison or basecalling pipelines.

## When NOT to use

- You need basecalled sequences or variant-level simulation rather than raw signal squiggles — seq2squiggle outputs signal data, not consensus reads.
- The goal is to preserve exact read sequences while adding signal variability but splitting reads is required — Read mode samples input reads without splitting them.
- Static k-mer-model simulation fidelity is explicitly acceptable and a lightweight CPU tool is preferred over a transformer model with model-weight downloads.

## Inputs

- FASTA file (genome/transcriptome reference) for Reference mode
- FASTA or FASTQ file of pre-existing (e.g. basecalled) reads for Read mode (--read-input)
- Pretrained model weights file (path via --model, or auto-downloaded)
- Sample-count / coverage / read-length parameters (-n, -c, -r, --distr) and chemistry profile (e.g. --profile dna_r9_min)

## Outputs

- SLOW5/BLOW5 file of simulated nanopore raw signal reads (default)
- POD5 file of simulated nanopore raw signal reads (via output format option)

## How to apply

Run `seq2squiggle predict` on an input FASTA/FASTQ with a compatible pretrained model (`--model`, or let the tool auto-download). Choose the simulation mode: default Reference mode samples synthetic reads from a genome/transcriptome and simulates a signal each, controlled by `-n`, `-c` (coverage), `-r` (average read length) and `--distr`; Read mode (`--read-input`) simulates signals directly from pre-existing reads — one signal per read, or `-n` signals sampled across reads without splitting them, preserving real read characteristics. Select a chemistry profile such as `--profile dna_r9_min` for R9.4.1 data, and optionally tune noise via `--noise-sampling`, `--noise-std`, `--duration-sampling`, `--dwell-mean`, `--dwell-std` (e.g. `--dwell-mean 10.0 --dwell-std 0.0` for ideal event lengths). Output defaults to SLOW5/BLOW5 (preferred for stability); POD5 export is also supported. Rationale: unlike static k-mer-model simulators such as squigulator, seq2squiggle's transformer learns sequential contextual information from segmented signal data, yielding signals with lower median DTW deviation from real reads (9.78 vs 10.01 on R10.4.1 HCT116 data, measured with mrmsdtw/linmdtw). GPU inference is recommended for large numbers of samples.

## Related tools

- **seq2squiggle** (Transformer-based, non-autoregressive simulator that generates nanopore sequencing signals from nucleotide sequences; invoked via `seq2squiggle predict`) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Static k-mer-model-based simulator used as the baseline comparator whose signals show larger median DTW deviation (10.01) from real reads)
- **uncalled4** (Generates event-level signal alignments (eventalign tables) that can serve as segmented training data for training a new seq2squiggle model) — https://github.com/skovaka/uncalled4
- **f5c** (Alternative to uncalled4 for producing eventalign training data (`f5c eventalign --samples --signal-index`)) — https://github.com/hasindu2008/f5c

## Examples

```
seq2squiggle predict example.fastq -o example.blow5 --read-input -n 2000
```

## Evaluation signals

- Output file is produced in BLOW5/POD5 format and contains the requested number of signals (-n) or the requested coverage (-c).
- In Read mode, one signal is generated per input read (1:1 mapping) unless -n is given, in which case exactly n signals are sampled across the input reads.
- Benchmark the simulated reads against real experimental nanopore reads with length-normalized DTW distances (mrmsdtw/linmdtw): expected median DTW ≈ 9.78 for seq2squiggle vs ≈ 10.01 for squigulator on R10.4.1 HCT116 data, i.e. seq2squiggle should show the lower median deviation.
- Model weights used are compatible with the target chemistry/profile (e.g. dna_r9_min profile for R9.4.1); seq2squiggle auto-downloads a compatible model if none is given.
- Noise settings are honored: with `--duration-sampling False --dwell-mean 10.0 --dwell-std 0.0 --noise-sampling False --noise-std 0.0` each k-mer event has ideal length 10 with no amplitude noise.

## Limitations

- Compatible pretrained model weights are required for predictions; without a --model file the tool must download one.
- GPU inference is strongly recommended for large sample counts; CPU-only inference is slow.
- The project's testing workflow is noted as 'Not implemented yet' in the methods.
- Read mode with -n samples n signals across input reads without splitting them — it does not simulate n signals per read.
- BLOW5 is preferred for stability; POD5 export requires specifying the model path through the configuration file.

## Evidence

- [abstract] Transformer-based signal generation: "we introduce seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences"
- [abstract] Superior similarity to real data: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
- [readme] Read mode usage: [evidence span withheld — anchor status: external_doc]
- [readme] Chemistry profile selection: [evidence span withheld — anchor status: external_doc]
- [readme] Model weights requirement: [evidence span withheld — anchor status: not_found]
- [readme] Read mode semantics: [evidence span withheld — anchor status: external_doc]
- [readme] GPU recommendation: [evidence span withheld — anchor status: external_doc]

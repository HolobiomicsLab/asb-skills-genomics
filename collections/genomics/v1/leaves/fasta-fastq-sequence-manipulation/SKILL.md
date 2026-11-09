---
name: fasta-fastq-sequence-manipulation
description: Use when simulating nanopore reads requires fragmenting a reference genome
  FASTA into synthetic reads, or when pre-existing FASTA/FASTQ reads must be supplied
  as signal-simulation input.
when_to_use_negative:
- Input already contains reads and no further fragmentation or synthetic sequence
  generation is desired — use --read-input only if signals are the goal, otherwise
  signal simulation is unnecessary
- The task is basecalling or real sequencing rather than in-silico read sampling from
  a reference
- The target output is a read-length distribution analysis alone with no signal simulation
  requirement
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Samples synthetic reads from the FASTA reference or ingests FASTA/FASTQ reads
    (--read-input) and simulates corresponding nanopore signals
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Runtime language of seq2squiggle used for FASTA/FASTQ parsing and read sampling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:bb3ef6086d5f531451b6fb646d06b240026ccfbf1c3aaa7db93a8747f5644b39
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fasta-fastq-sequence-manipulation

## Summary

Loading and sampling nucleotide subsequences from FASTA references (or ingesting pre-existing FASTA/FASTQ reads) to produce synthetic read sets that feed downstream nanopore signal simulation with seq2squiggle.

## When to use

Use when simulating nanopore reads requires fragmenting a reference genome FASTA into synthetic reads, or when pre-existing FASTA/FASTQ reads must be supplied as signal-simulation input.

## When NOT to use

- Input already contains reads and no further fragmentation or synthetic sequence generation is desired — use --read-input only if signals are the goal, otherwise signal simulation is unnecessary
- The task is basecalling or real sequencing rather than in-silico read sampling from a reference
- The target output is a read-length distribution analysis alone with no signal simulation requirement

## Inputs

- Reference genome or contigs in FASTA format
- Pre-existing reads in FASTA or FASTQ format (for --read-input mode)
- Simulation parameters: -n (read count), -c (coverage), -r (average read length), --distr (read-length distribution)

## Outputs

- Synthetic or selected nucleotide reads in FASTA/FASTQ
- Simulated nanopore raw signals in SLOW5/BLOW5 format (default) or POD5

## How to apply

In seq2squiggle's reference mode, load the reference genome from the input FASTA file and uniformly sample start positions until the user-specified coverage (-c), read count (-n), or average read length (-r) target is satisfied; control the read-length distribution with --distr, adopting one of the DeepSimulator-derived distributions (exponential, beta, or a mixed gamma distribution). Extract the subsequence at each sampled position and write the reads, which are then converted into raw nanopore signals and exported to SLOW5/BLOW5 or POD5 format. When reads already exist (e.g. basecalled FASTQ), use --read-input mode to bypass fragmentation: one signal per input read by default, or sample n signals across the reads with -n without splitting them. Choose reference mode when the goal is synthetic reads plus signals; choose read mode when real read characteristics should be preserved.

## Related tools

- **seq2squiggle** (Samples synthetic reads from the FASTA reference or ingests FASTA/FASTQ reads (--read-input) and simulates corresponding nanopore signals) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Runtime language of seq2squiggle used for FASTA/FASTQ parsing and read sampling)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 -n 10000 --distr <exp|beta|gamma> --profile dna_r9_min
```

## Evaluation signals

- The number of generated reads matches the -n count, or total read length divided by genome size matches the -c coverage target
- The average read length of sampled reads matches the -r target
- The empirical read-length histogram of generated reads matches the distribution specified by --distr (exponential, beta, or mixed gamma)
- With --read-input, exactly one signal is produced per input read; with --read-input -n, exactly n signals are produced without splitting reads
- Output file is a valid SLOW5/BLOW5 (or POD5) container with one signal per generated read

## Limitations

- Read mode with -n samples n signals across input reads (not n signals per read), so it does not amplify per-read signal diversity
- Compatible pretrained model weights must be specified via --model or downloaded; the model file path is required in the configuration file for POD5 export
- POD5 output is supported but BLOW5 is preferred for its stability
- GPU inference is recommended for high sample counts; CPU-only runs are slower

## Evidence

- [readme] Synthetic reads are sampled from the reference sequence.: [evidence span withheld — anchor status: external_doc]
- [readme] Use -n, -c, or -r to control number, coverage, or average read length.: [evidence span withheld — anchor status: external_doc]
- [readme] Read mode input FASTA or FASTQ with pre-existing reads, no additional fragmentation: [evidence span withheld — anchor status: external_doc]
- [results] Sequence generator adopts three DeepSimulator read-length distributions to mimic experimental nanopore read length distributions: [evidence span withheld — anchor status: not_found]
- [intro] seq2squiggle is a deep learning-based tool for generating artificial nanopore signals from DNA sequence data: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."

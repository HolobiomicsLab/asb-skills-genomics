---
name: read-sampling-from-reference-genome
description: Use when generating synthetic nanopore read sets from a reference genome/contigs
  in FASTA format, matching a specified coverage, read count, or empirical read-length
  distribution.
when_to_use_negative:
- Input is already a FASTA/FASTQ of pre-existing reads — use seq2squiggle's --read-input
  mode instead of sampling fresh reads from a reference
- The goal is only to simulate signals for existing reads without additional fragmentation,
  not to synthesize new read sequences
- Amplicon- or transcript-based read structures are required where uniform random
  sampling over the reference would be inappropriate
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: seq2squiggle
  role: Reference-mode simulator that samples synthetic reads from the input FASTA
    and simulates corresponding nanopore signals; --distr controls the read-length
    distribution
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Implementation language for the sampling workflow and numerical distribution-comparison
    checks
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:96fce0da836a71bdf42ac42c816575fc31e58f2c5b781071f634ea8126d55a82
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - seq2squiggle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# read-sampling-from-reference-genome

## Summary

Simulates synthetic nanopore reads by uniformly sampling start positions on a FASTA reference genome or contigs, drawing read lengths from user-selected target distributions (e.g. the exponential, beta, and mixed-gamma distributions adopted from DeepSimulator), and emitting a FASTA/FASTQ read set that can be fed to signal simulation with seq2squiggle.

## When to use

Use when generating synthetic nanopore read sets from a reference genome/contigs in FASTA format, matching a specified coverage, read count, or empirical read-length distribution.

## When NOT to use

- Input is already a FASTA/FASTQ of pre-existing reads — use seq2squiggle's --read-input mode instead of sampling fresh reads from a reference
- The goal is only to simulate signals for existing reads without additional fragmentation, not to synthesize new read sequences
- Amplicon- or transcript-based read structures are required where uniform random sampling over the reference would be inappropriate

## Inputs

- Reference genome or contig FASTA file
- User-specified coverage target (-c), read count (-n), or average read length (-r)
- Read-length distribution choice (--distr; exponential, beta, or mixed gamma as defined in DeepSimulator)

## Outputs

- Synthetic read set in FASTA or FASTQ format (sampled_reads)
- Empirical read-length histogram for validation against the target distribution

## How to apply

Load the reference genome or contigs from the input FASTA file and uniformly sample random start positions until the user-specified coverage (sum of read lengths / genome size), read count (-n), or average read length (-r) target is satisfied. For each read, draw a read length from one of the three DeepSimulator read-length distributions (exponential, beta, or a mixture of two gamma distributions) selected via the --distr parameter. Extract the subsequence at each start position and write the reads to a FASTA/FASTQ file, which can be passed directly to seq2squiggle predict to simulate raw nanopore signals. Validate by computing the empirical read-length histogram of the generated reads and numerically comparing it to the target distribution (e.g. via moments or a two-sample distribution distance).

## Related tools

- **seq2squiggle** (Reference-mode simulator that samples synthetic reads from the input FASTA and simulates corresponding nanopore signals; --distr controls the read-length distribution) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Implementation language for the sampling workflow and numerical distribution-comparison checks)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 -c 30 -r 5000
```

## Evaluation signals

- The empirical read-length distribution of sampled_reads matches the specified target distribution numerically within the reported comparison metric (moments or two-sample distance)
- Achieved coverage equals sum of read lengths divided by genome size and matches the user-specified -c value
- Number of generated reads equals the requested -n value when read count is used instead of coverage
- Every sampled read sequence is an exact subsequence of the input reference at a valid start position
- Output FASTA/FASTQ parses cleanly and is accepted by seq2squiggle predict for downstream signal simulation

## Limitations

- Start positions are drawn uniformly, so read placement does not model coverage biases or reference mappability effects
- Read-length fidelity depends on choosing one of the three supported distributions (exponential, beta, mixed gamma); other experimental length profiles are not natively supported
- GPU is recommended for fast downstream signal inference with seq2squiggle; CPU-only runs are slower
- The seq2squiggle README notes a POD5 export requires specifying a model path through the configuration file, and BLOW5 output is preferred for stability

## Evidence

- [other] Uniform sampling of start positions with length-distribution draws and validation: [evidence span withheld — anchor status: not_found]
- [other] DeepSimulator read-length distributions adopted by seq2squiggle: [evidence span withheld — anchor status: not_found]
- [readme] Reference mode described in the README: [evidence span withheld — anchor status: not_found]
- [readme] Distribution parameter from README: [evidence span withheld — anchor status: external_doc]
- [intro] Tool purpose from article intro: "`seq2squiggle` is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."

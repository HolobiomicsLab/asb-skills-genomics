---
name: read-length-distribution-sampling
description: Use when generating synthetic nanopore reads from a reference FASTA and
  you need realistic read-length heterogeneity rather than fixed-length reads, e.g.
  before seq2squiggle signal simulation.
when_to_use_negative:
- Input is already a FASTA/FASTQ of pre-existing reads — use seq2squiggle's --read-input
  mode, which simulates one signal per input read without additional fragmentation
- A uniform or fixed read length is explicitly required by the downstream benchmark
  design
- The goal is basecalling or read modification rather than de novo synthetic read
  generation from a reference
edam_operation: http://edamontology.org/operation_3520
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Generates synthetic reads from a FASTA reference by sampling read lengths
    from the chosen distribution (--distr) under -n/-c/-r control, then simulates
    corresponding nanopore signals
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Implementation language for the sequence-generator module and for numerically
    comparing the empirical read-length histogram against the target distribution
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:d5a5814bcf2601cefb798972725370ccabaf0dce31b1d5f83f2da63351e8dedb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - Python
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3520
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# read-length-distribution-sampling

## Summary

Sample synthetic nanopore read lengths from one of three distributions inherited from DeepSimulator — an exponential, a beta, or a mixed (two-gamma) distribution — so that simulated reads mimic experimental ONT read-length profiles before signal simulation with seq2squiggle.

## When to use

Use when generating synthetic nanopore reads from a reference FASTA and you need realistic read-length heterogeneity rather than fixed-length reads, e.g. before seq2squiggle signal simulation.

## When NOT to use

- Input is already a FASTA/FASTQ of pre-existing reads — use seq2squiggle's --read-input mode, which simulates one signal per input read without additional fragmentation
- A uniform or fixed read length is explicitly required by the downstream benchmark design
- The goal is basecalling or read modification rather than de novo synthetic read generation from a reference

## Inputs

- Reference genome or contig sequences in FASTA format
- User-specified target coverage (-c), read count (-n), or average read length (-r)
- Choice of read-length distribution: exponential, beta, or mixed gamma (via --distr)

## Outputs

- sampled_reads.fasta / sampled_reads.fastq containing synthetic reads with lengths drawn from the chosen distribution
- Empirical read-length histogram of the generated reads for validation against the target distribution

## How to apply

Load the reference genome or contigs from the input FASTA file and uniformly sample random start positions until the user-specified coverage (sum of read lengths divided by genome size) or read count is satisfied. For each read, draw a read length from one of the three DeepSimulator length distributions — exponential, beta, or a mixture of two gamma distributions — selected by the user via seq2squiggle's --distr parameter, or control the average length with -r alongside -n or -c. Extract the subsequence at each accepted start position and write all reads to a FASTA/FASTQ file that is then passed to seq2squiggle predict for signal simulation. The rationale is that experimental ONT read lengths are highly skewed, so fixed-length sampling produces unrealistic coverage profiles; the DeepSimulator distributions capture this heterogeneity.

## Related tools

- **seq2squiggle** (Generates synthetic reads from a FASTA reference by sampling read lengths from the chosen distribution (--distr) under -n/-c/-r control, then simulates corresponding nanopore signals) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Implementation language for the sequence-generator module and for numerically comparing the empirical read-length histogram against the target distribution)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 -c 30 -r 5000 --distr <exponential|beta|mixed_gamma>
```

## Evaluation signals

- Empirical read-length histogram of sampled_reads.fasta matches the specified target distribution (exponential, beta, or mixed gamma) within the reported comparison metric (e.g. moments or a two-sample distribution distance)
- Total sampled length divided by genome size equals the requested coverage when -c is used
- Number of reads in the output FASTA/FASTQ equals the requested -n value
- Mean read length approximates the value passed via -r when average-length control is used
- All sampled subsequences lie fully within reference bounds at their uniformly drawn start positions

## Limitations

- Read sampling is uniform over start positions, so it does not model positional or regional coverage biases of real sequencing runs
- Only the three distributions inherited from DeepSimulator (exponential, beta, mixed gamma) are available; custom empirical length histograms require modifying the generator
- Accurate signal simulation additionally depends on compatible pretrained model weights (--model); seq2squiggle will attempt to download a model file automatically if none is provided

## Evidence

- [other] Uniformly sample random start positions on the reference until the user-specified coverage (sum of read lengths / genome size) or read count is satisfied.: [evidence span withheld — anchor status: not_found]
- [other] DeepSimulator distributions: [evidence span withheld — anchor status: not_found]
- [readme] README read-length control: [evidence span withheld — anchor status: not_found]
- [abstract] seq2squiggle purpose: "we introduce seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences"

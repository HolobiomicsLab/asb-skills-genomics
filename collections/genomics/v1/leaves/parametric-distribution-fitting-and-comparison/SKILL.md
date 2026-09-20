---
name: parametric-distribution-fitting-and-comparison
description: Use when simulating sequencing reads or other stochastic quantities that
  must follow a realistic empirical length/size distribution, and the sampled output
  needs numerical validation against that target distribution.
when_to_use_negative:
- Input is already a set of pre-existing reads (use seq2squiggle's --read-input mode,
  which simulates signals directly without additional fragmentation)
- The goal is to model signal-level noise or dwell-time variation rather than read-length
  sampling (use the noise/duration samplers or static normal distributions for amplitude
  and event durations instead)
edam_operation: http://edamontology.org/operation_3579
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Samples synthetic reads from a FASTA reference using the selected read-length
    distribution (--distr) and simulates corresponding nanopore signals
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Python
  role: Implementation language for the sequence generator and for computing empirical
    histograms and distribution-comparison metrics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:0845ed891de0c577f1501fea874c3c887591ed1cda3a8540eb65fc653b161bd7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3579
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# parametric-distribution-fitting-and-comparison

## Summary

Draw and validate simulated quantities (e.g. nanopore read lengths) from parametric distributions — exponential, beta, or a mixed/two-gamma mixture inherited from DeepSimulator — and numerically verify that the empirical histogram of generated samples matches the target distribution.

## When to use

Use when simulating sequencing reads or other stochastic quantities that must follow a realistic empirical length/size distribution, and the sampled output needs numerical validation against that target distribution.

## When NOT to use

- Input is already a set of pre-existing reads (use seq2squiggle's --read-input mode, which simulates signals directly without additional fragmentation)
- The goal is to model signal-level noise or dwell-time variation rather than read-length sampling (use the noise/duration samplers or static normal distributions for amplitude and event durations instead)

## Inputs

- Reference genome or contigs in FASTA format
- User-specified target coverage, read count, or average read length
- Selected read-length distribution name (exponential, beta, or mixed gamma)

## Outputs

- Synthetic reads written to a FASTA/FASTQ file (e.g. sampled_reads.fasta)
- Empirical read-length histogram of the generated reads
- Numerical comparison metric between empirical and target distributions

## How to apply

Load the reference sequences from a FASTA file and uniformly sample random start positions until the user-specified coverage (sum of read lengths / genome size) or read count (-n / -c / -r in seq2squiggle) is reached. For each read, draw a read length from one of three parametric distributions adopted from DeepSimulator — an exponential distribution, a beta distribution, or a mixture of two gamma distributions — selected by the user via the --distr parameter. Extract the subsequence at each sampled start position and write all reads to a FASTA/FASTQ file. To validate, compute the empirical read-length histogram of the generated reads and numerically compare it against the specified target distribution (e.g. via moments or a two-sample distribution distance); the simulation is correct only if this comparison falls within the reported metric's tolerance.

## Related tools

- **seq2squiggle** (Samples synthetic reads from a FASTA reference using the selected read-length distribution (--distr) and simulates corresponding nanopore signals) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Python** (Implementation language for the sequence generator and for computing empirical histograms and distribution-comparison metrics)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 -c 30 -r 5000 --distr <exponential|beta|mixed-gamma>
```

## Evaluation signals

- Empirical read-length histogram of sampled_reads.fasta matches the specified target distribution numerically within the reported comparison metric (moments or two-sample distribution distance)
- Sum of sampled read lengths divided by genome size equals the user-specified coverage target (when -c is used)
- Number of generated reads equals the requested count (when -n is used)
- Read start positions are uniformly distributed across the reference, with subsequences extracted correctly at each start position

## Limitations

- The length distributions are limited to the three parametric forms adopted from DeepSimulator (exponential, beta, mixed gamma); arbitrary empirical length distributions are not supported by this mechanism
- Sampling is uniform across the reference, so coverage bias or reference-region-specific length effects are not modeled
- seq2squiggle's testing workflow is listed as 'Not implemented yet', so distribution validation must be performed externally on the sampled reads

## Evidence

- [other] The sequence generator adopts three read-length distributions previously defined in DeepSimulator — an exponential distribution, a beta distribution, and a mixed gamma distribution — to mimic experimental nanopore read length distributions.: [evidence span withheld — anchor status: not_found]
- [readme] Use --distr to specify the read length distribution.: [evidence span withheld — anchor status: not_found]
- [readme] Synthetic reads are sampled from the reference sequence.: [evidence span withheld — anchor status: not_found]
- [other] Validation criterion from the source card workflow requiring empirical-vs-target distribution agreement.: [evidence span withheld — anchor status: not_found]

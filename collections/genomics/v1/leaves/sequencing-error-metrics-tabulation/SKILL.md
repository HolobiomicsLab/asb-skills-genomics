---
name: sequencing-error-metrics-tabulation
description: Use when comparing simulated nanopore read quality across simulators
  (e.g. seq2squiggle vs squigulator) on R10.4.1 datasets and you need median match
  rate and median PHRED score per organism/mode.
when_to_use_negative:
- Input is real sequencing data rather than simulator output — the metric ranges (match
  rate 91.91–95.29, PHRED 11.61–13.67) only serve as simulator benchmark expectations,
  not real-data quality targets
- The task is signal generation itself (noise/duration sampler tuning) rather than
  tabulating downstream basecalling error metrics
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Transformer-based simulator generating nanopore signals and synthetic reads
    from FASTA/FASTQ input; its output quality is the treatment being benchmarked
    against squigulator
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Comparison simulator (static k-mer model baseline) whose basecalling accuracy
    metrics are tabulated alongside seq2squiggle's
- name: dorado
  role: Basecaller used to convert simulated squiggle signals (SLOW5/BLOW5) into nucleotide
    reads with PHRED quality scores
- name: minimap2
  role: Aligner used to map dorado basecalls back to reference genomes to recover
    per-read alignment records for error-rate computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:24a72ed7b04e6ab6681463459ac53acf1f781c28aaaccdf2c0fb57fbce02edef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dorado
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# sequencing-error-metrics-tabulation

## Summary

Benchmark nanopore read simulators (seq2squiggle vs squigulator) by tabulating per-organism, per-mode basecalling error metrics — median match rate, mismatch/insertion/deletion rates, and median PHRED score — from dorado basecalls of simulated squiggle signals aligned with minimap2.

## When to use

Use when comparing simulated nanopore read quality across simulators (e.g. seq2squiggle vs squigulator) on R10.4.1 datasets and you need median match rate and median PHRED score per organism/mode.

## When NOT to use

- Input is real sequencing data rather than simulator output — the metric ranges (match rate 91.91–95.29, PHRED 11.61–13.67) only serve as simulator benchmark expectations, not real-data quality targets
- The task is signal generation itself (noise/duration sampler tuning) rather than tabulating downstream basecalling error metrics

## Inputs

- Reference genome FASTA files (Human HCT116, D. melanogaster, E. coli) for R10.4.1 chemistry
- Pre-existing basecalled reads as FASTA/FASTQ for Read-mode simulation
- Simulated raw squiggle signals in SLOW5/BLOW5 (or POD5) format
- Per-read alignment records (SAM/BAM) from minimap2
- Basecalled FASTQ reads with PHRED qualities from dorado

## Outputs

- One CSV benchmark table per dataset with median match rate, mismatch rate, insertion rate, deletion rate, and median PHRED score per simulator/mode/organism combination
- Simulated read sets (100,000 reads per organism per simulator/mode)

## How to apply

Obtain the reference genomes (Human HCT116, D. melanogaster, E. coli) as FASTA and simulate 100,000 reads per organism with seq2squiggle in both Read-mode (--read-input) and Genome-mode (default reference mode), repeating the simulations with squigulator. Basecall the resulting SLOW5/BLOW5 squiggle signals with dorado to obtain nucleotide reads, then align the basecalls back to their reference genomes with minimap2 to recover per-read alignment records. From the alignment records and basecall qualities, compute the median match rate, mismatch rate, insertion rate, deletion rate, and median PHRED score for each simulator/mode/organism combination, assembling the metrics into one CSV benchmark table per dataset. Judge correctness by checking that seq2squiggle's ranges (match rate 91.91–95.29, PHRED 11.61–13.67) fall above squigulator's (87.71–93.00 match rate, Q 8.91–11.91), as reported in the R10.4.1 benchmarks.

## Related tools

- **seq2squiggle** (Transformer-based simulator generating nanopore signals and synthetic reads from FASTA/FASTQ input; its output quality is the treatment being benchmarked against squigulator) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Comparison simulator (static k-mer model baseline) whose basecalling accuracy metrics are tabulated alongside seq2squiggle's)
- **dorado** (Basecaller used to convert simulated squiggle signals (SLOW5/BLOW5) into nucleotide reads with PHRED quality scores)
- **minimap2** (Aligner used to map dorado basecalls back to reference genomes to recover per-read alignment records for error-rate computation)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 -n 10000  # simulate reads/signals; then basecall with dorado and align basecalls with minimap2 to tabulate error metrics
```

## Evaluation signals

- Median match rate for seq2squiggle reads falls within 91.91–95.29 across R10.4.1 Read-mode and Genome-mode benchmarks (91.91 human Read-mode to 95.29 E. coli Genome-mode)
- Median match rate for squigulator reads falls within 87.71–93.00, i.e. below seq2squiggle for every organism/mode combination
- Median PHRED scores are 11.61–13.67 for seq2squiggle versus 8.91–11.91 for squigulator
- Each simulated dataset contains ~100,000 reads per organism per simulator/mode, and basecalls align successfully back to the reference with minimap2
- One CSV benchmark table is produced per dataset with all five metrics (match, mismatch, insertion, deletion, PHRED) populated for every combination

## Limitations

- Metrics are specific to R10.4.1 chemistry benchmarks (Human HCT116, D. melanogaster, E. coli); they do not generalize to other pore chemistries without re-running the pipeline
- seq2squiggle requires compatible pretrained model weights specified via the --model parameter; without a model file it attempts to auto-download one
- GPU inference is recommended for high sample counts; CPU-only inference is slower
- Testing workflow for the tooling is not implemented, per the repository methods documentation

## Evidence

- [abstract] Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution"
- [results] Reads achieved a higher median match rate (91.91 in human Read-mode up to 95.29 in E. coli Genome-mode) than squigulator: [evidence span withheld — anchor status: not_found]
- [readme] README: simulation inputs and modes: [evidence span withheld — anchor status: external_doc]
- [readme] README: model weights requirement: [evidence span withheld — anchor status: external_doc]

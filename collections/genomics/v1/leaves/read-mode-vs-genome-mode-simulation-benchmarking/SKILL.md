---
name: read-mode-vs-genome-mode-simulation-benchmarking
description: Use when you need to quantitatively compare nanopore signal simulators'
  output quality on R10.4.1 chemistry — e.g. comparing simulated-read basecalling
  accuracy (match rate, PHRED) between seq2squiggle and k-mer-model tools like squigulator.
when_to_use_negative:
- You need biological downstream validity of simulated reads (variant calling, assembly
  benchmarks) rather than basecalling-accuracy metrics — this skill only measures
  match rate/PHRED of basecalled simulated signals.
- Your data uses non-R10.4.1 chemistry (e.g. R9.4.1) without a compatible pretrained
  model; seq2squiggle requires matching model weights for predictions.
- You want to evaluate simulator signal fidelity directly (e.g. per-k-mer amplitude/dwell
  distributions) rather than through a basecalling/alignment round-trip.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3307
tools:
- name: seq2squiggle
  role: Transformer-based simulator generating nanopore signals in Read-mode and Genome/Reference-mode;
    the tool under evaluation
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Static k-mer-model-based signal simulator used as the comparison baseline
- name: dorado
  role: Basecaller that converts simulated squiggle signals (BLOW5/POD5) into nucleotide
    reads for accuracy measurement
- name: minimap2
  role: Aligns dorado basecalls back to the reference genome to recover per-read alignment
    records for metric computation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:c3bf1da53d5223a653331c33e8004b0f1df1715418bdb7f564865132404504b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
  tools:
  - dorado
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# read-mode-vs-genome-mode-simulation-benchmarking

## Summary

A benchmarking skill that compares nanopore read simulators (seq2squiggle vs squigulator) in Read-mode and Genome-mode on R10.4.1 datasets by basecalling simulated signals with dorado, aligning them with minimap2, and reporting median match rate and median PHRED score.

## When to use

Use when you need to quantitatively compare nanopore signal simulators' output quality on R10.4.1 chemistry — e.g. comparing simulated-read basecalling accuracy (match rate, PHRED) between seq2squiggle and k-mer-model tools like squigulator.

## When NOT to use

- You need biological downstream validity of simulated reads (variant calling, assembly benchmarks) rather than basecalling-accuracy metrics — this skill only measures match rate/PHRED of basecalled simulated signals.
- Your data uses non-R10.4.1 chemistry (e.g. R9.4.1) without a compatible pretrained model; seq2squiggle requires matching model weights for predictions.
- You want to evaluate simulator signal fidelity directly (e.g. per-k-mer amplitude/dwell distributions) rather than through a basecalling/alignment round-trip.

## Inputs

- Reference genome FASTA files (Human HCT116, D. melanogaster, E. coli) for Genome/Reference-mode simulation
- Pre-existing basecalled FASTA/FASTQ read files for Read-mode (--read-input) simulation
- Pretrained R10.4.1 model weights specified via --model or a chemistry profile
- Simulated raw signals in BLOW5/POD5 format from both simulators

## Outputs

- Dorado basecalled FASTQ reads for each simulator/mode/organism combination
- Per-read minimap2 alignment records against the reference genomes
- Per-combination metrics: median match rate, mismatch rate, insertion rate, deletion rate, median PHRED score
- One CSV benchmark table per dataset comparing seq2squiggle vs squigulator

## How to apply

Obtain reference genomes (Human HCT116, D. melanogaster, E. coli) in FASTA and simulate 100,000 reads per organism with seq2squiggle in Read-mode (--read-input on pre-existing basecalled reads) and in default Reference/Genome-mode (sampling synthetic reads from the FASTA), repeating each simulation with squigulator. Basecall the resulting BLOW5/POD5 squiggle signals with dorado, align the basecalled FASTQ reads back to each reference with minimap2, and compute per-read match, mismatch, insertion and deletion rates plus median PHRED score from basecall qualities. Assemble metrics into one CSV per dataset and compare simulator ranges: seq2squiggle reached median match rates 91.91–95.29 and PHRED 11.61–13.67 versus squigulator's 87.71–93.00 match and 8.91–11.91 PHRED. Run seq2squiggle on GPU for inference speed when simulating large read counts, and specify the R10.4.1-compatible model via --profile/--model.

## Related tools

- **seq2squiggle** (Transformer-based simulator generating nanopore signals in Read-mode and Genome/Reference-mode; the tool under evaluation) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Static k-mer-model-based signal simulator used as the comparison baseline)
- **dorado** (Basecaller that converts simulated squiggle signals (BLOW5/POD5) into nucleotide reads for accuracy measurement)
- **minimap2** (Aligns dorado basecalls back to the reference genome to recover per-read alignment records for metric computation)

## Examples

```
seq2squiggle predict example.fastq -o example.blow5 --read-input -n 100000
```

## Evaluation signals

- Median match rate of seq2squiggle-simulated reads falls in the 91.91–95.29 range across organisms/modes, exceeding squigulator's 87.71–93.00.
- Median PHRED score of seq2squiggle basecalls falls in 11.61–13.67 versus squigulator's 8.91–11.91.
- Each simulator/mode/organism combination yields exactly 100,000 simulated reads before basecalling.
- Read-mode runs produce one signal per input read (1:1 mapping) when --read-input is used without -n.
- Benchmark CSV exists per dataset with all five metrics (match, mismatch, insertion, deletion rates, median PHRED) populated for every combination.

## Limitations

- seq2squiggle requires compatible pretrained model weights (via --model) matching the pore chemistry; without them predictions cannot proceed, and CPU-only inference is slow for large read counts.
- BLOW5 export needs the model path specified through the configuration file; POD5 export is supported but BLOW5 is preferred for stability.
- The benchmark workflow measures basecalling accuracy of simulated signals only; seq2squiggle's testing workflow is noted as not implemented in the repository.
- Read-mode with -n samples n total signals across input reads, not n signals per read, so coverage scaling differs from Genome-mode sampling.

## Evidence

- [readme] seq2squiggle is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data.: "seq2squiggle is a deep learning-based tool for generating artifical nanopore signals from DNA sequence data."
- [readme] Read-mode one-signal-per-read behavior: [evidence span withheld — anchor status: external_doc]
- [readme] Performance guidance for large simulations: [evidence span withheld — anchor status: external_doc]
- [abstract] Superiority finding motivating the benchmark: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution"
- [abstract] Contrast with static k-mer simulators like squigulator: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"

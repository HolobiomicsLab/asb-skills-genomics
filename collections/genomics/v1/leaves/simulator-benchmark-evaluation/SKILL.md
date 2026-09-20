---
name: simulator-benchmark-evaluation
description: Use when comparing nanopore signal simulators (e.g. seq2squiggle vs squigulator)
  on similarity of simulated raw signals to real experimental reads, via DTW distance
  on a fixed read set.
when_to_use_negative:
- The goal is to generate synthetic reads from a reference genome rather than compare
  simulators on existing reads — use seq2squiggle's default Reference mode instead
  of Read-mode evaluation
- Only basecalled sequences are available with no real raw signal data to serve as
  the ground-truth comparison set
- Evaluating sequence-level fidelity (base accuracy) rather than raw signal-level
  similarity to experimental reads
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Transformer-based, non-autoregressive simulator generating nanopore signals
    from input reads in Read-mode (--read-input); the tool being benchmarked
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Static k-mer-model-based simulator used as the comparison baseline on the
    same input sequences
- name: mrmsdtw
  role: Computes pairwise length-normalized DTW distances between simulated and real
    signal read sets
- name: linmdtw
  role: Backend package used by mrmsdtw for fast DTW computation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:9c8b56d6704d46517969be5a4a7a39d7ef5ac98eaa563eaa56d2062cd128c857
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - linmdtw
  - mrmsdtw
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# simulator-benchmark-evaluation

## Summary

Benchmark nanopore signal simulators by measuring how closely their simulated raw signals resemble real experimental reads, using length-normalized dynamic-time-warping (DTW) distances. This is the evaluation strategy the seq2squiggle paper used to show transformer-based simulation outperforms static k-mer-model simulators (median DTW ≈ 9.78 vs ≈ 10.01 on R10.4.1 data).

## When to use

Use when comparing nanopore signal simulators (e.g. seq2squiggle vs squigulator) on similarity of simulated raw signals to real experimental reads, via DTW distance on a fixed read set.

## When NOT to use

- The goal is to generate synthetic reads from a reference genome rather than compare simulators on existing reads — use seq2squiggle's default Reference mode instead of Read-mode evaluation
- Only basecalled sequences are available with no real raw signal data to serve as the ground-truth comparison set
- Evaluating sequence-level fidelity (base accuracy) rather than raw signal-level similarity to experimental reads

## Inputs

- FASTA/FASTQ file of nucleotide sequences shared by all simulators under comparison (e.g. 2000 pre-existing basecalled reads)
- Real experimental nanopore raw signal reads from the same chemistry (e.g. R10.4.1 HCT116 data)
- Pretrained seq2squiggle model weights compatible with the pore chemistry

## Outputs

- Per-simulator raw signal files (BLOW5/SLOW5 or POD5) for the 2000 simulated reads
- Pairwise length-normalized DTW distances between each simulated read set and the real reads
- Median DTW distance per simulator written to median_dtw.tsv (seq2squiggle ≈ 9.78, squigulator ≈ 10.01)

## How to apply

Generate an equal number of simulated reads from the identical nucleotide sequences with each simulator under comparison — here 2000 reads per simulator, one from the transformer-based seq2squiggle and one from the static k-mer-model-based squigulator — in Read-mode (--read-input) so signals are simulated directly from pre-existing basecalled reads without additional fragmentation. Load the matched real experimental nanopore reads (e.g. R10.4.1 HCT116 data) and compute pairwise length-normalized DTW distances between each simulated set and the real reads using mrmsdtw, backed by the linmdtw package. Aggregate per-pair DTW distances per simulator and compare medians rather than means, since DTW distance distributions are skewed. Judge correctness by consistency with the published ordering: seq2squiggle median DTW ≈ 9.78 versus squigulator ≈ 10.01, with the smaller median indicating the simulator whose signals are closer to real data.

## Related tools

- **seq2squiggle** (Transformer-based, non-autoregressive simulator generating nanopore signals from input reads in Read-mode (--read-input); the tool being benchmarked) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Static k-mer-model-based simulator used as the comparison baseline on the same input sequences)
- **mrmsdtw** (Computes pairwise length-normalized DTW distances between simulated and real signal read sets)
- **linmdtw** (Backend package used by mrmsdtw for fast DTW computation)

## Examples

```
seq2squiggle predict example.fastq -o simulated.blow5 --read-input -n 2000
```

## Evaluation signals

- Median DTW distance for seq2squiggle simulated reads ≈ 9.78 and for squigulator ≈ 10.01 against real R10.4.1 reads, preserving the published ordering
- Each simulator receives the identical 2000 input sequences so DTW distances are directly comparable
- DTW distances are length-normalized before aggregation so reads of different lengths contribute comparably
- Medians (not means) are extracted per simulator since the paper reports median deviation values
- Result file median_dtw.tsv contains exactly the two median values, one per simulator

## Limitations

- seq2squiggle requires compatible pretrained model weights; without a model file it will auto-download one, and a GPU is recommended for fast inference though CPU works more slowly
- Read-mode with -n samples n total signals across input reads (without splitting reads) rather than n signals per read, which must be understood when scaling datasets
- DTW-based comparison reflects signal-level similarity only and depends on the chemistry-specific real-read ground truth (here R10.4.1); results may not transfer to other pore chemistries
- The seq2squiggle repository notes testing workflow was not yet implemented

## Evidence

- [results] Reads generated by squigulator showed a slightly larger median DTW deviation (10.01) from real experimental reads than reads generated by seq2squiggle (9.78), indicating seq2squiggle's signals are closer to real data.: [evidence span withheld — anchor status: not_found]
- [abstract] Transformer-based vs static k-mer simulation: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"
- [readme] Read-mode input behavior: [evidence span withheld — anchor status: external_doc]
- [readme] Model weights requirement: [evidence span withheld — anchor status: external_doc]
- [readme] Performance guidance: [evidence span withheld — anchor status: external_doc]

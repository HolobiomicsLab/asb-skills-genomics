---
name: length-normalized-dtw-signal-alignment
description: Use when evaluating how closely simulated nanopore raw signals (BLOW5/POD5)
  resemble real experimental reads on the same chemistry (e.g. R10.4.1), via DTW distance
  medians.
when_to_use_negative:
- The goal is Reference-mode simulation of reads and signals from a genome/transcriptome
  FASTA rather than comparison of signals for existing reads
- Input is basecalled sequence data only, with no raw signal (BLOW5/POD5) available
  for the real reads — DTW on signals cannot be computed
- Comparing simulators across different pore chemistries or kits, where signal-level
  DTW distances are not directly comparable
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Transformer-based simulator generating the test signals from input reads in
    Read-mode (--read-input) for the real-vs-simulated DTW comparison
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Static k-mer-model-based simulator generating the baseline signal set for
    the DTW benchmark
- name: mrmsdtw
  role: Computes pairwise length-normalized DTW distances between simulated and real
    nanopore signals
- name: linmdtw
  role: Backend providing the linear-memory DTW algorithm used by mrmsdtw
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:0cc6f14e5d51fc348672aa59f214c482d6d51d231ed4bdf60291e6cdf7130234
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - linmdtw
  - mrmsdtw
  - seq2squiggle
  - squigulator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# length-normalized-dtw-signal-alignment

## Summary

Compares simulated nanopore signal reads (seq2squiggle vs squigulator) to real experimental reads by computing pairwise length-normalized dynamic-time-warping distances and aggregating per-simulator medians. This is the benchmark's quantitative read-mode similarity test on R10.4.1 HCT116 data.

## When to use

Use when evaluating how closely simulated nanopore raw signals (BLOW5/POD5) resemble real experimental reads on the same chemistry (e.g. R10.4.1), via DTW distance medians.

## When NOT to use

- The goal is Reference-mode simulation of reads and signals from a genome/transcriptome FASTA rather than comparison of signals for existing reads
- Input is basecalled sequence data only, with no raw signal (BLOW5/POD5) available for the real reads — DTW on signals cannot be computed
- Comparing simulators across different pore chemistries or kits, where signal-level DTW distances are not directly comparable

## Inputs

- Nucleotide sequences of 2000 pre-existing reads (FASTA or FASTQ)
- 2000 real experimental nanopore signal reads (R10.4.1 chemistry, HCT116 data)
- Simulated signal reads in SLOW5/BLOW5 (or POD5) format from seq2squiggle and from a k-mer-model-based simulator (squigulator)

## Outputs

- Per-pair length-normalized DTW distances between each simulated set and the real reads
- median_dtw.tsv with one median DTW distance per simulator (seq2squiggle ≈ 9.78, squigulator ≈ 10.01)

## How to apply

Generate 2000 simulated signal reads per simulator from the same nucleotide sequences: seq2squiggle (transformer-based, end-to-end) and squiggle-style static k-mer-model simulation, using Read-mode (--read-input) so signals map 1:1 to the pre-existing reads without fragmentation. Load the 2000 real experimental nanopore reads for the same chemistry and compute pairwise length-normalized DTW distances between each simulated set and the real reads using mrmsdtw, backed by the linmdtw package. Aggregate per-pair distances per simulator and extract the median. A lower median DTW distance indicates closer resemblance to real data; the validated reference result is seq2squiggle ≈ 9.78 versus squigulator ≈ 10.01, confirming seq2squiggle's superior similarity. Write the two medians to median_dtw.tsv and check the ordering matches the expected benchmark.

## Related tools

- **seq2squiggle** (Transformer-based simulator generating the test signals from input reads in Read-mode (--read-input) for the real-vs-simulated DTW comparison) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Static k-mer-model-based simulator generating the baseline signal set for the DTW benchmark)
- **mrmsdtw** (Computes pairwise length-normalized DTW distances between simulated and real nanopore signals)
- **linmdtw** (Backend providing the linear-memory DTW algorithm used by mrmsdtw)

## Examples

```
seq2squiggle predict reads.fastq -o simulated.blow5 --read-input -n 2000
```

## Evaluation signals

- seq2squiggle median DTW ≈ 9.78 and squigulator median DTW ≈ 10.01; the invariant is median(seq2squiggle) < median(squigulator)
- median_dtw.tsv contains exactly two rows/median values, one per simulator
- Each simulated read maps 1:1 to an input read in Read-mode (exactly one signal per input read)
- All pairwise DTW distances are length-normalized and non-negative; no failed/NaN pairs remain after aggregation
- Per-simulator set contains 2000 signals sampled across the input reads (not n signals per read)

## Limitations

- DTW medians are chemistry- and dataset-specific (R10.4.1, HCT116); the ordering may not transfer to other pores, flowcells, or kits
- Benchmark comparison requires both simulators to be run on the same input sequences and same number of reads (2000) for a fair median comparison
- seq2squiggle requires compatible pretrained model weights (--model); without a model file it auto-downloads one, and README notes testing workflow is not implemented

## Evidence

- [abstract] we introduce seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences: "we introduce seq2squiggle, a novel transformer-based, non-autoregressive model designed to generate nanopore sequencing signals from nucleotide sequences"
- [abstract] superior similarity to real data: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data"
- [abstract] Unlike existing simulators relying on static k-mer models: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"
- [readme] Read-mode invocation: [evidence span withheld — anchor status: external_doc]
- [readme] one signal per input read: [evidence span withheld — anchor status: external_doc]

---
name: median-distance-statistics-reporting
description: Use when comparing two sets of simulated nanopore raw signals against
  a real read set and you need a robust per-simulator similarity statistic (median
  DTW distance) written to a TSV.
when_to_use_negative:
- Input is nucleotide sequence data only and signals have not yet been simulated —
  first generate signals with seq2squiggle/squigulator before computing DTW distances
- The comparison target is read accuracy of basecalled sequences rather than raw-signal
  similarity — DTW on raw squiggles is not the right metric
- Only one simulator is available and no ranking between simulators is needed
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: seq2squiggle
  role: Transformer-based simulator generating the simulated nanopore signals being
    scored (Read mode, --read-input, -n)
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: squigulator
  role: Static k-mer-model-based simulator providing the baseline simulated signals
    for the DTW comparison
- name: mrmsdtw
  role: Computes pairwise length-normalized DTW distances between simulated and real
    signal reads
- name: linmdtw
  role: Backend DTW engine used by mrmsdtw for linear-memory dynamic time warping
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:55566e52d8d69530e4734c45ce768822a30b9a59ad449d76bc098c3e05b217c0
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
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Median Distance Statistics Reporting

## Summary

Computes length-normalized DTW distances between simulated nanopore signal reads (seq2squiggle vs squigulator) and real experimental R10.4.1 reads, then aggregates per-simulator medians (e.g. 9.78 vs 10.01) into median_dtw.tsv to rank simulator fidelity.

## When to use

Use when comparing two sets of simulated nanopore raw signals against a real read set and you need a robust per-simulator similarity statistic (median DTW distance) written to a TSV.

## When NOT to use

- Input is nucleotide sequence data only and signals have not yet been simulated — first generate signals with seq2squiggle/squigulator before computing DTW distances
- The comparison target is read accuracy of basecalled sequences rather than raw-signal similarity — DTW on raw squiggles is not the right metric
- Only one simulator is available and no ranking between simulators is needed

## Inputs

- 2000 simulated nanopore signal reads from seq2squiggle (BLOW5/POD5 output)
- 2000 simulated nanopore signal reads from squigulator (BLOW5/POD5 output)
- 2000 real experimental R10.4.1 HCT116 nanopore reads
- Input FASTA or FASTQ files of basecalled reads for Read-mode simulation

## Outputs

- median_dtw.tsv with one median length-normalized DTW distance per simulator (seq2squiggle ≈ 9.78, squigulator ≈ 10.01)
- per-pair DTW distance distributions per simulator

## How to apply

Simulate 2000 nanopore signal reads from the same nucleotide sequences with both seq2squiggle (transformer-based end-to-end simulator) and squigulator (static k-mer-model simulator), using seq2squiggle's Read mode (--read-input) to simulate signals directly from pre-existing basecalled reads without additional fragmentation. Load the 2000 real experimental R10.4.1 reads and compute pairwise length-normalized dynamic-time-warping distances between each simulated set and the real reads using mrmsdtw, backed by the linmdtw package. Aggregate the per-pair DTW distances per simulator and extract the median, then write the two median values to median_dtw.tsv. Rationale: median deviation is robust to outliers in DTW distance distributions, and the benchmark compares a deep-learning simulator against static k-mer-model baselines.

## Related tools

- **seq2squiggle** (Transformer-based simulator generating the simulated nanopore signals being scored (Read mode, --read-input, -n)) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **squigulator** (Static k-mer-model-based simulator providing the baseline simulated signals for the DTW comparison)
- **mrmsdtw** (Computes pairwise length-normalized DTW distances between simulated and real signal reads)
- **linmdtw** (Backend DTW engine used by mrmsdtw for linear-memory dynamic time warping)

## Examples

```
seq2squiggle predict example.fastq -o example.blow5 --read-input -n 2000
```

## Evaluation signals

- Median DTW for seq2squiggle ≈ 9.78 and for squigulator ≈ 10.01, reproducing the reported ordering (seq2squiggle < squigulator)
- median_dtw.tsv contains exactly two rows/median values, one per simulator
- All pairwise distances are computed after length normalization, so values are comparable across reads of different lengths
- Each simulated set is derived from the same 2000 input sequences and compared against the same 2000 real reads

## Limitations

- The benchmark is specific to R10.4.1 HCT116 data and 2000 reads; medians may differ for other chemistries, flowcells (e.g. FLO-MIN114), or sample sizes
- seq2squiggle inference is much faster on GPU; on CPU-only systems inference speed is slower
- Model weights must match the chemistry; if no model file is provided seq2squiggle attempts to download a compatible one automatically

## Evidence

- [other] Reads generated by squigulator showed a slightly larger median DTW deviation (10.01) from real experimental reads than reads generated by seq2squiggle (9.78): [evidence span withheld — anchor status: not_found]
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [abstract] abstract: "Seq2squiggle consistently outperforms existing tools across multiple datasets, demonstrating superior similarity to real data and offering a robust solution for simulating nanopore sequencing signals"

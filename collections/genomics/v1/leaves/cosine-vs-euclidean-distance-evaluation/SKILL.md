---
name: cosine-vs-euclidean-distance-evaluation
description: Use when tuning a k-mer-feature-based sequence similarity search and
  you must decide which distance metric (cosine vs Euclidean) and feature weighting
  (TF, IDF, TF-IDF, raw) to feed an approximate nearest neighbor search such as NNDescent.
when_to_use_negative:
- The feature representation is already fixed and normalized (e.g. you only need to
  run a single Fedrann invocation, not compare metric/weighting combinations)
- The task is alignment-level identity computation rather than ANN-based neighbor
  ranking — use a dedicated aligner (minimap2, DALIGN) instead of re-running the k-mer
  feature pipeline
- Input is already a feature matrix or overlap graph; this skill operates upstream,
  at the Fedrann feature-extraction/ANN-search configuration level
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Runs the feature-extraction + NNDescent pipeline once per weighting scheme
    and distance metric, emitting overlaps.tsv for each of the 8 configurations
  repo: https://github.com/jzhang-dev/fedrann
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows used to score overlap graphs
    against reference truth
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: minimap2
  role: Source of reference/ground-truth alignments used to judge whether predicted
    overlaps are true
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:6c26b1eb32585a6b7e24eda4a2cd868433c480f25d1db7ca3fb4f8d3e08b0bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
  tools:
  - Fedrann
  - kNN-overlap-finder
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# cosine-vs-euclidean-distance-evaluation

## Summary

A benchmarking skill for systematically comparing similarity metrics (cosine vs Euclidean distance) in NNDescent-based overlap detection on k-mer feature matrices, cross-tabulated against k-mer weighting schemes (raw/binary, TF, IDF/ICF, TF-IDF) to identify the configuration with the lowest overlap graph error rate.

## When to use

Use when tuning a k-mer-feature-based sequence similarity search and you must decide which distance metric (cosine vs Euclidean) and feature weighting (TF, IDF, TF-IDF, raw) to feed an approximate nearest neighbor search such as NNDescent.

## When NOT to use

- The feature representation is already fixed and normalized (e.g. you only need to run a single Fedrann invocation, not compare metric/weighting combinations)
- The task is alignment-level identity computation rather than ANN-based neighbor ranking — use a dedicated aligner (minimap2, DALIGN) instead of re-running the k-mer feature pipeline
- Input is already a feature matrix or overlap graph; this skill operates upstream, at the Fedrann feature-extraction/ANN-search configuration level

## Inputs

- Long-read sequencing reads (FASTA/FASTQ, e.g. test/data/reads.fasta.gz)
- Reference truth overlap set (e.g. minimap2/DALIGN-derived alignments for benchmarking)
- Fedrann feature weighting configuration per run (raw/binary, TF, IDF/ICF, TF-IDF)

## Outputs

- overlaps.tsv per run (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- Comparison table of overlap error rates across all weighting x distance-metric combinations (8 configurations)
- Identified best configuration (expected: IDF + cosine distance)

## How to apply

Run Fedrann once per feature weighting scheme (raw/binary, TF, IDF approximated by inverse collection frequency, and TF-IDF) on the long-read FASTA/FASTQ input with fixed parameters (k=15, --kmer-sample-fraction 0.05, --kmer-min-multiplicity 2, --seed 602, --threads 32), each time using NNDescent with cosine distance, collecting each run's overlaps.tsv. Repeat all weighting-scheme runs with NNDescent configured for Euclidean distance to form the 8 weighting x metric combinations. Score every resulting overlap graph against reference truth overlaps, filtering alignments to >= 100 matched bases and >= 10% alignment identity to define true overlaps, then compute overlap error rates for all configurations into a single comparison table. The expected outcome, grounded in the article's finding, is that IDF weighting with cosine distance yields the lowest overlap graph error rate; the IDF transformation is fused with SRP by pre-multiplying the SRP projection matrix with the IDF weight vector, and cosine distance is naturally suited to the sparse, normalized k-mer vectors this produces.

## Related tools

- **Fedrann** (Runs the feature-extraction + NNDescent pipeline once per weighting scheme and distance metric, emitting overlaps.tsv for each of the 8 configurations) — https://github.com/jzhang-dev/fedrann
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows used to score overlap graphs against reference truth) — https://github.com/jzhang-dev/kNN-overlap-finder
- **minimap2** (Source of reference/ground-truth alignments used to judge whether predicted overlaps are true)

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --threads 32  # repeat per weighting scheme, toggling cosine vs Euclidean distance in NNDescent
```

## Evaluation signals

- The IDF + cosine configuration exhibits the lowest overlap graph error rate among all 8 weighting x distance combinations, consistent with the reported pipeline accuracy of the IDF + SRP + NNDescent combination
- Every run produces an overlaps.tsv with the expected schema (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance) and neighbor_rank starting at 1
- All configurations were run with identical k-mer parameters (k=15, sample fraction 0.05, min multiplicity 2, seed 602) so the only varying factors are weighting scheme and distance metric
- Truth overlaps are consistently defined across all evaluations by the alignment filters: >= 100 matched bases and >= 10% alignment identity
- Each comparison-table cell has a finite, non-negative error rate; missing or empty overlaps.tsv for any configuration invalidates the comparison

## Limitations

- IDF is approximated by total collection frequency (CF) of a k-mer rather than a true document frequency, so 'IDF' weighting here is an efficient approximation (ICF)
- Random projection and k-mer sampling (5% of k-mers) introduce stochasticity; results depend on the random seed (--seed 602 in the reference run)
- The benchmark finding (IDF + cosine superiority) was established on chr22-style benchmark data with simulated/real long reads; behavior on other k values or read types may differ
- Accuracy claims rely on the specific truth-alignment thresholds (>= 100 matched bases, >= 10% identity); stricter or looser filters could change the ranking

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [discussion] All alignments were filtered using two criteria: [evidence span withheld — anchor status: not_found]
- [discussion] Fedrann therefore adopts the total count: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors."
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"

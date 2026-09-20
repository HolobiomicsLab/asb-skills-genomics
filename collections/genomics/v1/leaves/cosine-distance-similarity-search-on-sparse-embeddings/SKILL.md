---
name: cosine-distance-similarity-search-on-sparse-embeddings
description: Use when long-read FASTQ/FASTA reads must be searched for candidate overlaps
  via k-NN on high-dimensional sparse k-mer embeddings, and exact brute-force search
  is too slow.
when_to_use_negative:
- When the read set is small enough that exact brute-force neighbor computation is
  tractable — ANN precision loss is unnecessary.
- 'When memory is severely constrained: the paper notes Fedrann''s memory consumption
  is higher than most existing overlap detection tools.'
- When reads are not long-read sequencing data (e.g. short reads or already-assembled
  reference sequences), since the pipeline targets long-read overlap detection.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Fedrann
  role: End-to-end pipeline that performs k-mer feature extraction, ICF weighting,
    SRP dimensionality reduction, and cosine-distance NNDescent k-NN search on long
    reads
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Jellyfish
  role: k-mer counting and multiplicity filtering preceding feature extraction
- name: kNN-overlap-finder
  role: Benchmarking and evaluation workflows for validating neighbor precision and
    runtime against exact neighbors
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Python
  role: Implementation language; C-accelerated numerical libraries and batch-based
    matrix operations give performance parity with compiled tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:7467601dbbb78b8f806935b5b461d8dfe49800ba493add7e5ebe08a66fe499eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Jellyfish
  - Python
  - kNN-overlap-finder
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cosine-distance-similarity-search-on-sparse-embeddings

## Summary

Performs approximate k-nearest-neighbor search with cosine distance on sparse, SRP/ICF-reduced k-mer feature matrices from long-read data (Fedrann), where NNDescent provides the best speed–accuracy trade-off for overlap graph construction.

## When to use

Use when long-read FASTQ/FASTA reads must be searched for candidate overlaps via k-NN on high-dimensional sparse k-mer embeddings, and exact brute-force search is too slow.

## When NOT to use

- When the read set is small enough that exact brute-force neighbor computation is tractable — ANN precision loss is unnecessary.
- When memory is severely constrained: the paper notes Fedrann's memory consumption is higher than most existing overlap detection tools.
- When reads are not long-read sequencing data (e.g. short reads or already-assembled reference sequences), since the pipeline targets long-read overlap detection.

## Inputs

- gzipped FASTQ/FASTA file of long sequencing reads (e.g. reads.fasta.gz)
- sparse SRP-reduced feature matrix in .npz format (feature_matrix.npz)
- k-mer sampling/min-multiplicity parameters (e.g. --kmer-sample-fraction 0.05, --kmer-min-multiplicity 2)

## Outputs

- overlaps.tsv with query_name, query_orientation, target_name, target_orientation, neighbor_rank, and distance columns
- neighbor indices and distances per query read
- fedrann.log pipeline progress log
- optional feature_matrix.npz sparse feature matrix

## How to apply

Extract k-mer features with Fedrann (Jellyfish counting, --kmer-min-multiplicity 2 to drop sequencing-error k-mers, --kmer-sample-fraction 0.05), weight k-mers by inverse collection frequency (ICF, an approximation of IDF), and project with Sparse Random Projection fused as a pre-multiplied SRP_transpose x ICF matrix. Run graph-based NNDescent with cosine distance on the resulting sparse feature matrix (feature_matrix.npz) to obtain neighbor indices and distances; NNDescent was chosen because it achieved precision comparable to HNSW while matching IVF-PQ's speed, whereas PQ and RPF were less accurate. For each query, emit ranked neighbors with orientations and dissimilarity into overlaps.tsv, where smaller cosine distance values indicate higher similarity. Match the k-mer size to the dataset context (e.g. k=30 for H4C, k=50 for H4O; the README example uses k=15) and set a fixed --seed for reproducibility.

## Related tools

- **Fedrann** (End-to-end pipeline that performs k-mer feature extraction, ICF weighting, SRP dimensionality reduction, and cosine-distance NNDescent k-NN search on long reads) — https://github.com/jzhang-dev/FEDRANN
- **Jellyfish** (k-mer counting and multiplicity filtering preceding feature extraction)
- **kNN-overlap-finder** (Benchmarking and evaluation workflows for validating neighbor precision and runtime against exact neighbors) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Python** (Implementation language; C-accelerated numerical libraries and batch-based matrix operations give performance parity with compiled tools)

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- Neighbor precision computed against exact (brute-force) neighbor sets should match HNSW-level precision and exceed PQ/IVF-PQ/RPF for the same embedding matrix.
- In overlaps.tsv, distance values must decrease monotonically as neighbor_rank increases (rank 1 is the best match, smallest distance).
- Same --seed and parameters should reproduce identical neighbor indices and distances across runs.
- test/test.sh installation check passes and feature_matrix.npz is a valid scipy sparse format file when --save-feature-matrix is set.
- Index-build and query runtimes recorded in ann_benchmark_results.csv should place NNDescent among the fastest methods (comparable to IVF-PQ).

## Limitations

- Fedrann's memory consumption is higher than most existing overlap detection tools; memory-efficient representations or alternative ANN implementations remain future work.
- PCA and UMAP cannot scale to human-genome-sized feature matrices, constraining dimensionality reduction to sparse projections.
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive is provided.
- ANN candidate neighbors are approximate; overlap accuracy depends on the recall of the chosen method (PQ, IVF-PQ, and RPF were less accurate than NNDescent and HNSW on SRP embeddings).

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [methods] Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"
- [readme] neighbor_rank: The similarity rank of the target sequence among all potential matches for the query sequence, where `1` is the best match: "The similarity rank of the target sequence among all potential matches for the query sequence, where `1` is the best match, `2` is the second best, and so on."
- [readme] distance column semantics in overlaps.tsv: "Measures the dissimilarity between the embedded vectors of the query and target sequences. A smaller value indicates higher similarity between the sequences."

---
name: ann-runtime-profiling
description: Use when you must choose among ANN schemes (NNDescent, HNSW, PQ, IVF-PQ,
  RPF) for k-NN search on a reduced-dimension embedding matrix (e.g. SRP features
  from long reads) and need a quantified precision-vs-runtime ranking before committing
  to one index.
when_to_use_negative:
- The goal is overlap detection itself rather than index selection — run the Fedrann
  pipeline end-to-end instead of the ANN benchmark
- The input is raw k-mer count space rather than a dimensionality-reduced matrix;
  the article notes PCA and UMAP fail to scale to human-genome feature matrices, so
  profiling ANN methods on unreduced high-dimensional features is out of scope
- The embedding matrix is tiny enough that exact brute-force search is affordable,
  making ANN approximation unnecessary
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Fedrann
  role: Produces the sparse SRP feature matrix (feature_matrix.npz) that serves as
    the common input for benchmarking the five ANN schemes
  repo: https://github.com/jzhang-dev/FEDRANN
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows that house the ANN runtime/precision
    comparison
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Python
  role: Implementation language for the ANN benchmark, leveraging C-accelerated numerical
    libraries and batch-based matrix operations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:b6b0148ebb7dd5d71c1cfad851680adb88140be2e51c0030c6c2e749be313000
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Python
  - kNN-overlap-finder
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# ann-runtime-profiling

## Summary

Benchmarks approximate nearest neighbor (ANN) implementations — NNDescent, HNSW, PQ, IVF-PQ, and RPF — on SRP-reduced long-read k-mer embeddings, tabulating neighbor precision against brute-force exact neighbors alongside index-build and query runtime to select the best speed-accuracy trade-off for overlap graph construction.

## When to use

Use when you must choose among ANN schemes (NNDescent, HNSW, PQ, IVF-PQ, RPF) for k-NN search on a reduced-dimension embedding matrix (e.g. SRP features from long reads) and need a quantified precision-vs-runtime ranking before committing to one index.

## When NOT to use

- The goal is overlap detection itself rather than index selection — run the Fedrann pipeline end-to-end instead of the ANN benchmark
- The input is raw k-mer count space rather than a dimensionality-reduced matrix; the article notes PCA and UMAP fail to scale to human-genome feature matrices, so profiling ANN methods on unreduced high-dimensional features is out of scope
- The embedding matrix is tiny enough that exact brute-force search is affordable, making ANN approximation unnecessary

## Inputs

- Sparse SRP-reduced embedding matrix produced by Fedrann with --save-feature-matrix (feature_matrix.npz)
- Long-read FASTA/FASTQ sequencing reads used to generate the embeddings
- Exact (brute-force) k-NN neighbor sets computed on the same embeddings as the ground truth
- Matched neighbor count (k) settings shared across all five ANN methods

## Outputs

- ann_benchmark_results.csv with per-method neighbor precision, index-build runtime, and query runtime
- Ranking of the five ANN methods on speed-accuracy trade-off for overlap graph construction
- Selected ANN method (expected: NNDescent) for scalable overlap detection

## How to apply

Load the sparse SRP embedding matrix (feature_matrix.npz) emitted by the Fedrann feature-extraction pipeline (Jellyfish k-mer counting, ICF weighting, and SRP projection fused as SRP_transpose × ICF). Run NNDescent with cosine distance on the embeddings to obtain neighbor indices and distances, then run HNSW, PQ, IVF-PQ, and RPF on the same matrix with matched neighbor-count settings so the comparison is controlled. For each method, compute neighbor precision against exact brute-force neighbor sets and record both index-build and query runtime. Tabulate precision and runtime per method, rank the five schemes on their speed-accuracy trade-off, and write the table to ann_benchmark_results.csv. The paper's expected outcome is that NNDescent and HNSW achieve high precision while PQ, IVF-PQ, and RPF are less accurate, with NNDescent and IVF-PQ fastest — so NNDescent should emerge as the best overall trade-off for overlap graph construction.

## Related tools

- **Fedrann** (Produces the sparse SRP feature matrix (feature_matrix.npz) that serves as the common input for benchmarking the five ANN schemes) — https://github.com/jzhang-dev/FEDRANN
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows that house the ANN runtime/precision comparison) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Python** (Implementation language for the ANN benchmark, leveraging C-accelerated numerical libraries and batch-based matrix operations)

## Evaluation signals

- Reproduced ranking matches the paper's profile: NNDescent and HNSW show high neighbor precision; PQ, IVF-PQ, and RPF are less accurate
- NNDescent and IVF-PQ are the fastest on build/query runtime, and NNDescent ranks first on the combined speed-accuracy trade-off
- All five methods were run on the identical embedding matrix with matched neighbor-count settings, so precision differences are attributable to the ANN scheme
- Each method's precision is validated against exact brute-force neighbor sets, not self-reported distances
- ann_benchmark_results.csv contains both runtime (build and query) and precision columns for every method — no method missing a metric

## Limitations

- Fedrann's memory consumption is higher than most existing overlap detection tools; memory-efficient data representations, shared-memory designs, or alternative ANN implementations are future work
- The benchmark was developed on SRP-reduced embeddings; behavior of PQ/IVF-PQ/RPF on other reduction schemes is not characterized here
- Reproducibility depends solely on GitHub repositories (kNN-overlap-finder); no DOI or Zenodo archive is provided

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [methods] Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"
- [discussion] highlighting an important direction for future optimization through more memory-efficient data representations, shared-memory designs, or alternative implementations of ANN methods: "highlighting an important direction for future optimization through more memory-efficient data representations, shared-memory designs, or alternative implementations of ANN methods"
- [discussion] Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude: "Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude"
- [readme] This repository contains all testing, data processing, benchmarking, and evaluation workflows for the development and publication of FEDRANN software: [evidence span withheld — anchor status: external_doc]

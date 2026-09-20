---
name: approximate-nearest-neighbor-benchmarking
description: Use when choosing among ANN algorithms (NNDescent, HNSW, PQ, IVF-PQ,
  RPF) for k-NN search on a high-volume embedding matrix where both query speed and
  neighbor precision must be quantified against a brute-force ground truth.
when_to_use_negative:
- The goal is exact nearest-neighbor search or sequence alignment (e.g., minimap2/BLAST-style
  overlap detection), not approximate search parameter selection
- The input is raw sequencing reads rather than an already-reduced embedding matrix
  — feature extraction and dimensionality reduction must be run first via the Fedrann
  pipeline
- You need end-to-end overlap-graph quality or assembly contiguity evaluation rather
  than ANN method comparison (use the kNN-overlap-finder benchmarking workflows instead)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Fedrann
  role: Produces the sparse SRP-reduced feature matrix and implements the NNDescent
    ANN search over long-read embeddings
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Python
  role: Implementation language for the benchmarking pipeline and C-accelerated numerical
    libraries used for the ANN comparisons
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows for the ANN comparison
    reported in the article
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:8001b49fe3a4773c87f1e7eafe9f2e6f89cc9041aa5ad43d75f696d934b49b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Python
  - kNN-overlap-finder
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# approximate-nearest-neighbor-benchmarking

## Summary

Benchmarks multiple approximate nearest neighbor (ANN) schemes — NNDescent, HNSW, PQ, IVF-PQ, and RPF — on low-dimensional embeddings (e.g., SRP-reduced k-mer feature matrices) by measuring neighbor precision against exact brute-force neighbors and index-build/query runtime, then ranks methods on their speed-accuracy trade-off for a downstream task such as overlap graph construction.

## When to use

Use when choosing among ANN algorithms (NNDescent, HNSW, PQ, IVF-PQ, RPF) for k-NN search on a high-volume embedding matrix where both query speed and neighbor precision must be quantified against a brute-force ground truth.

## When NOT to use

- The goal is exact nearest-neighbor search or sequence alignment (e.g., minimap2/BLAST-style overlap detection), not approximate search parameter selection
- The input is raw sequencing reads rather than an already-reduced embedding matrix — feature extraction and dimensionality reduction must be run first via the Fedrann pipeline
- You need end-to-end overlap-graph quality or assembly contiguity evaluation rather than ANN method comparison (use the kNN-overlap-finder benchmarking workflows instead)

## Inputs

- Sparse embedding matrix (feature_matrix.npz) produced by FEDRANN feature extraction with IDF/ICF weighting and Sparse Random Projection
- Exact brute-force k-NN neighbor sets (ground truth) on the same matrix
- Matched neighbor count (k) setting applied uniformly across all ANN methods

## Outputs

- ann_benchmark_results.csv tabulating neighbor precision and index-build/query runtime per ANN method
- Speed-accuracy ranking of the five ANN methods used to select one for overlap graph construction

## How to apply

Load the sparse SRP-reduced embedding matrix (feature_matrix.npz) and run the NNDescent graph-based ANN with cosine distance to obtain neighbor indices and distances. Run the four alternatives — HNSW, PQ, IVF-PQ, and RPF — on the same matrix with matched neighbor-count settings. For each method, compute neighbor precision against exact brute-force neighbor sets and record index-build and query runtime. Tabulate precision and runtime per method and rank the methods on their speed-accuracy trade-off; in the FEDRANN context, NNDescent won this trade-off for overlap graph construction on SRP embeddings, so validate that the reproduced ranking is consistent with that profile.

## Related tools

- **Fedrann** (Produces the sparse SRP-reduced feature matrix and implements the NNDescent ANN search over long-read embeddings) — https://github.com/jzhang-dev/FEDRANN
- **Python** (Implementation language for the benchmarking pipeline and C-accelerated numerical libraries used for the ANN comparisons)
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows for the ANN comparison reported in the article) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
nndescent = NNDescent(X_srp, n_neighbors=15, metric='cosine'); neighbors, distances = nndescent.neighbor_graph
```

## Evaluation signals

- NNDescent and HNSW should achieve high neighbor precision while PQ, IVF-PQ, and RPF are less accurate on SRP embeddings
- NNDescent and IVF-PQ should be the fastest methods by query runtime
- The reproduced speed-accuracy ranking should select NNDescent as the best trade-off, matching the paper's reported profile
- ann_benchmark_results.csv must contain precision and runtime rows for all five methods under matched neighbor-count settings
- Neighbor precision values are computed against exact brute-force neighbor sets on the same embedding matrix, so precision for each method is in [0, 1]

## Limitations

- PCA and UMAP do not scale to human-genome-sized feature matrices, so direct ANN application requires the SRP reduction step first
- Memory consumption of the pipeline is higher than most existing overlap detection tools; alternative ANN implementations are noted as future work
- The benchmark is specific to overlap detection embeddings; no validation yet for metagenomic binning, taxonomic classification, or other applications

## Evidence

- [results] NNDescent selected via speed-accuracy benchmark: [evidence span withheld — anchor status: not_found]
- [abstract] Pipeline composition: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [readme] Feature matrix output file: "`feature_matrix.npz`: (Optional) This sparse-format file contains the feature matrix generated during the analysis."
- [methods] SRP method: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"
- [abstract] Python performance parity: "Fedrann achieves performance parity with tools written in compiled languages by leveraging C-accelerated numerical libraries and optimized batch-based matrix operations"

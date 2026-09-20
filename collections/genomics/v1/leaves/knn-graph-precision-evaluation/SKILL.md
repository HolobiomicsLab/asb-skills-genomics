---
name: knn-graph-precision-evaluation
description: Decision support for choosing an ANN algorithm when building k-NN overlap
  graphs from sparse, dimensionality-reduced long-read embeddings and needing quantitative
  precision-vs-runtime rankings rather than a single default method.
when_to_use_negative:
- Input is raw FASTA/FASTQ reads rather than an already-reduced embedding matrix —
  run the FEDRANN feature-extraction/DR stages first
- The goal is end-to-end overlap detection or assembly quality (contiguity/BUSCO)
  rather than ANN method selection — use the full Fedrann benchmarking workflow instead
- The dataset is small enough that exact brute-force k-NN is tractable, making ANN
  approximation unnecessary
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Produces the sparse SRP-reduced feature matrix (feature_matrix.npz) that is
    the input for ANN benchmarking
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Python
  role: Implementation environment in which NNDescent, HNSW, PQ, IVF-PQ, and RPF are
    run and precision/runtime are tabulated
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows used to run the ANN comparison
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:6335d9e4af3959389861188977e096fc742b3b94300dedc233edded3eb6de447
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# knn-graph-precision-evaluation

## Summary

Benchmarks approximate nearest-neighbor (ANN) schemes (NNDescent, HNSW, PQ, IVF-PQ, RPF) on SRP-reduced k-mer feature matrices by scoring neighbor precision against exact brute-force neighbors and recording index-build/query runtime, to select the best speed-accuracy trade-off for overlap graph construction.

## When to use

Decision support for choosing an ANN algorithm when building k-NN overlap graphs from sparse, dimensionality-reduced long-read embeddings and needing quantitative precision-vs-runtime rankings rather than a single default method.

## When NOT to use

- Input is raw FASTA/FASTQ reads rather than an already-reduced embedding matrix — run the FEDRANN feature-extraction/DR stages first
- The goal is end-to-end overlap detection or assembly quality (contiguity/BUSCO) rather than ANN method selection — use the full Fedrann benchmarking workflow instead
- The dataset is small enough that exact brute-force k-NN is tractable, making ANN approximation unnecessary

## Inputs

- Sparse SRP-reduced embedding matrix saved as feature_matrix.npz from the FEDRANN pipeline
- Matched neighbor count (k) settings shared across all five ANN methods
- Exact (brute-force) k-NN neighbor sets used as ground-truth reference

## Outputs

- ann_benchmark_results.csv with per-method neighbor precision and build/query runtimes
- Ranked speed-accuracy trade-off table over NNDescent, HNSW, PQ, IVF-PQ, and RPF

## How to apply

Load the sparse SRP embedding matrix (feature_matrix.npz) produced by the FEDRANN feature-extraction pipeline (Jellyfish k-mer counting with --kmer-min-multiplicity 2, ICF weighting fused with SRP via a pre-multiplied projection matrix). Run NNDescent with cosine distance on the embeddings to obtain neighbor indices and distances, then run the four alternatives — HNSW, PQ, IVF-PQ, and RPF — on the same matrix with matched neighbor-count settings. For each method, compute neighbor precision against exact (brute-force) neighbor sets and record index-build and query runtime separately. Tabulate precision and runtime per method and rank the five methods on their speed-accuracy trade-off; in the paper's profile NNDescent combined the high precision of graph-based methods with near-IVF-PQ speed, so it was selected for scalable overlap graph construction.

## Related tools

- **Fedrann** (Produces the sparse SRP-reduced feature matrix (feature_matrix.npz) that is the input for ANN benchmarking) — https://github.com/jzhang-dev/FEDRANN
- **Python** (Implementation environment in which NNDescent, HNSW, PQ, IVF-PQ, and RPF are run and precision/runtime are tabulated)
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows used to run the ANN comparison) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
fedrann -i reads.fasta.gz -o out/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- Reproduced method ranking matches the paper's profile: NNDescent and HNSW achieve high precision while PQ, IVF-PQ, and RPF are less accurate, and NNDescent offers the best overall speed-accuracy trade-off
- Neighbor precision is computed against exact brute-force neighbor sets, not against another ANN method's output
- Build runtime and query runtime are recorded separately for each method, not merged into a single timing
- Identical neighbor count settings and the same embedding matrix are used across all five methods so precision/runtime differences are attributable to the ANN scheme
- Output ann_benchmark_results.csv contains one row per method with non-null precision and runtime values

## Limitations

- Benchmark conclusions are specific to SRP-reduced embeddings; ANN search methods had not been directly applied to overlap identification prior to this work because PCA and UMAP fail to scale to human-genome-scale feature matrices
- Fedrann's memory consumption is higher than most existing overlap detection tools; more memory-efficient data representations or alternative ANN implementations are noted as future work
- No DOI or Zenodo archive is provided, so reproducibility of the benchmark depends solely on the GitHub repositories

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [abstract] Fedrann achieves performance parity with tools written in compiled languages by leveraging C-accelerated numerical libraries and optimized batch-based matrix operations.: "Fedrann achieves performance parity with tools written in compiled languages by leveraging C-accelerated numerical libraries and optimized batch-based matrix operations."
- [readme] feature_matrix.npz: (Optional) This sparse-format file contains the feature matrix generated during the analysis.: "`feature_matrix.npz`: (Optional) This sparse-format file contains the feature matrix generated during the analysis."
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [methods] Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"

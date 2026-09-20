---
name: approximate-nearest-neighbor-graph-search
description: Use when you have a low-dimensional embedding matrix of long sequencing
  reads (e.g. ICF/IDF-weighted SRP features from FASTQ/FASTA) and need ranked candidate
  overlaps for each read to build an overlap graph for de novo assembly.
when_to_use_negative:
- Input is short-read data or the goal is exact pairwise alignment with base-level
  coordinates — this stage yields candidate overlaps (ranked neighbor pairs with distances),
  not aligned bases; benchmark against minimap2 instead
- The feature matrix is so large that PCA/UMAP-class dense dimensionality reduction
  is required — those methods fail to scale to human-genome-sized matrices, so only
  randomized projections (SRP/GRP/SimHash) are viable here
- Memory is severely constrained — Fedrann's memory consumption is higher than most
  existing overlap detection tools
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3474
tools:
- name: Fedrann
  role: End-to-end pipeline implementing k-mer feature extraction, ICF-weighted SRP
    embedding, and NNDescent ANN search that writes overlaps.tsv
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Python
  role: Implementation language of Fedrann, leveraging C-accelerated numerical libraries
    and optimized batch-based matrix operations for the ANN stage
  repo: https://github.com/jzhang-dev/FEDRANN
- name: minimap2
  role: Generates reference alignments used to evaluate the accuracy of the ANN-produced
    candidate overlap sets
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:fada533f837b699a333023f6474f45c0fe8ab7c5261aa4e3704e2919c890614b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Python
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# approximate-nearest-neighbor-graph-search

## Summary

Runs graph-based approximate nearest-neighbor search (NNDescent, cosine metric) on low-dimensional read embeddings to produce ranked candidate overlaps for long-read overlap graph construction. It is the ANN stage of the FEDRANN pipeline, turning IDF-weighted SRP-embedded reads into a query/target overlap table.

## When to use

Use when you have a low-dimensional embedding matrix of long sequencing reads (e.g. ICF/IDF-weighted SRP features from FASTQ/FASTA) and need ranked candidate overlaps for each read to build an overlap graph for de novo assembly.

## When NOT to use

- Input is short-read data or the goal is exact pairwise alignment with base-level coordinates — this stage yields candidate overlaps (ranked neighbor pairs with distances), not aligned bases; benchmark against minimap2 instead
- The feature matrix is so large that PCA/UMAP-class dense dimensionality reduction is required — those methods fail to scale to human-genome-sized matrices, so only randomized projections (SRP/GRP/SimHash) are viable here
- Memory is severely constrained — Fedrann's memory consumption is higher than most existing overlap detection tools

## Inputs

- FASTQ/FASTA file of long sequencing reads (e.g. reads.fasta.gz)
- Sparse k-mer count vectors per read built by kmer_searcher
- IDF/ICF-weighted, SRP-projected low-dimensional embedding matrix (optionally feature_matrix.npz)

## Outputs

- overlaps.tsv with query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance columns
- fedrann.log pipeline progress log
- feature_matrix.npz (optional sparse embedding matrix)

## How to apply

First build the embedding: count k-mers with Jellyfish (k=15), filter k-mers below --kmer-min-multiplicity 2 to remove sequencing errors, sample 5% of k-mers (--kmer-sample-fraction 0.05, --seed 602) into a forward/reverse k-mer library, locate k-mer positions with the C++ kmer_searcher to build sparse count vectors, then apply the fused ICF-weighted SRP transformation to obtain normalized low-dimensional vectors. Run NNDescent graph-based ANN search on this embedding matrix with the cosine distance metric so each read gets a ranked list of most-similar reads. Write the ranked neighbor pairs to overlaps.tsv with columns query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance; use multiple threads (--threads 32) and optionally persist the embedding as feature_matrix.npz. Choose SRP over PCA/UMAP-class methods because only SRP, GRP, and SimHash completed on the largest H3-class datasets, and SRP was the fastest with the lowest memory usage; the ANN stage returns candidate overlaps, which downstream evaluation benchmarks against minimap2 reference alignments.

## Related tools

- **Fedrann** (End-to-end pipeline implementing k-mer feature extraction, ICF-weighted SRP embedding, and NNDescent ANN search that writes overlaps.tsv) — https://github.com/jzhang-dev/FEDRANN
- **Python** (Implementation language of Fedrann, leveraging C-accelerated numerical libraries and optimized batch-based matrix operations for the ANN stage) — https://github.com/jzhang-dev/FEDRANN
- **minimap2** (Generates reference alignments used to evaluate the accuracy of the ANN-produced candidate overlap sets)

## Examples

```
fedrann -i reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- overlaps.tsv exists and is a valid TSV containing exactly the six required columns (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- neighbor_rank values start at 1 for each query's best match, and smaller distance values indicate higher sequence similarity between embedded vectors
- fedrann.log records pipeline progress from start to finish
- when --save-feature-matrix is set, feature_matrix.npz is a valid sparse-format file
- the candidate overlap sets benchmarked against minimap2 reference alignments yield overlap graph accuracy comparable to MECAT2, minimap2, and wtdbg2

## Limitations

- Memory consumption of Fedrann is higher than most existing overlap detection tools; memory-efficient data representations, shared-memory designs, or alternative ANN implementations are future work
- PCA and UMAP fail to scale effectively to human-genome-sized feature matrices, restricting the DR choices feeding the ANN stage
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive is provided
- Ann-stage results are candidate overlaps requiring downstream evaluation against reference alignments (e.g. minimap2) rather than validated true overlaps

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [results] In the ANN stage of FEDRANN, approximate k-NN algorithms identify the most similar sequences for each query read: [evidence span withheld — anchor status: not_found]
- [discussion] Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude: "Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude"
- [readme] This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics.: "`overlaps.tsv`: This file lists all identified candidate overlaps with sequence names, orientations, and their similarity metrics."
- [methods] Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"

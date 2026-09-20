---
name: dimensionality-reduction-method-benchmarking
description: Decision support for choosing a dimensionality-reduction method when
  embedding large sparse k-mer matrices (e.g., from long-read overlap detection) and
  needing to know which methods complete within fixed runtime/memory budgets.
when_to_use_negative:
- When the goal is benchmarking alignment-based overlap tools (minimap2, MHAP, MECAT2,
  wtdbg2, xRead, BLEND) rather than comparing DR methods on a shared feature matrix
- When the input is already an embedding or an overlap set — this skill requires the
  raw-read-to-sparse-k-mer-feature-matrix stage as its starting point
- When the dataset is small enough that PCA/UMAP-class methods are known to complete
  — the scalability comparison is only meaningful at human-genome-scale feature matrices
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Fedrann
  role: Runs feature extraction (Jellyfish k-mer counting, multiplicity filtering,
    ICF/IDF weighting) and hosts the DR and NNDescent steps whose runtime and memory
    are benchmarked
  repo: https://github.com/jzhang-dev/FEDRANN
- name: minimap2
  role: Produces reference alignments used as ground truth for evaluating each DR
    method's predicted overlap sets
- name: Python
  role: Implementation language of Fedrann; performance parity with compiled tools
    is achieved via C-accelerated numerical libraries and batch-based matrix operations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:48efce134a23b55582c9cbac9517c7e32a8233b22c19d5db6df3c9351b9e5c2b
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dimensionality-reduction-method-benchmarking

## Summary

Benchmark multiple dimensionality-reduction methods (PCA, UMAP, Spectral Embedding, scBiMapping, SRP, GRP, SimHash) applied to IDF/ICF-weighted sparse k-mer feature matrices from long-read data, recording overlap accuracy, wall-clock runtime, and peak memory to identify which methods scale to large H3-class datasets.

## When to use

Decision support for choosing a dimensionality-reduction method when embedding large sparse k-mer matrices (e.g., from long-read overlap detection) and needing to know which methods complete within fixed runtime/memory budgets.

## When NOT to use

- When the goal is benchmarking alignment-based overlap tools (minimap2, MHAP, MECAT2, wtdbg2, xRead, BLEND) rather than comparing DR methods on a shared feature matrix
- When the input is already an embedding or an overlap set — this skill requires the raw-read-to-sparse-k-mer-feature-matrix stage as its starting point
- When the dataset is small enough that PCA/UMAP-class methods are known to complete — the scalability comparison is only meaningful at human-genome-scale feature matrices

## Inputs

- Long-read sequencing data in FASTQ/FASTA format (e.g., ONT R9 SRR10028111, PacBio HiFi HG002)
- Sparse IDF/ICF-weighted k-mer feature matrix (feature_matrix.npz) built with a chosen k (k=30 for H4C, k=50 for H4O)
- Reference alignments from minimap2 (PAF files) used as ground-truth overlap labels

## Outputs

- Comparison table of accuracy metrics, wall-clock runtime, and peak memory per DR method (PCA, UMAP, Spectral Embedding, scBiMapping, SRP, GRP, SimHash)
- Candidate overlap sets per method from NNDescent cosine-distance search
- fedrann.log records of per-stage runtime and memory consumption

## How to apply

First run Fedrann feature extraction on the long-read FASTQ/FASTA input: Jellyfish k-mer counting with --kmer-min-multiplicity 2, k-mer sampling (--kmer-sample-fraction 0.05), and ICF/IDF weighting to build the sparse k-mer feature matrix. Then apply each DR method in turn (PCA, UMAP, Spectral Embedding, scBiMapping, GRP, SimHash, SRP) to the IDF-weighted matrix to produce read embeddings, noting that Fedrann fuses IDF weighting and SRP by pre-multiplying the SRP projection matrix with the IDF weight vector. For each embedding, run NNDescent approximate nearest-neighbor search with the cosine distance metric to generate candidate overlap sets. Evaluate each overlap graph against minimap2 reference alignments (filtered at >= 100 matched bases and >= 10% alignment identity) and record wall-clock runtime and peak memory from the fedrann.log output, using a fixed random seed (--seed 602) and multi-threaded execution for reproducibility. Accept the comparison table only if every method row contains both accuracy and resource measurements and the winning scalable method (SRP) finishes under 6 hours with peak memory under 1 TB on the largest H3-class dataset.

## Related tools

- **Fedrann** (Runs feature extraction (Jellyfish k-mer counting, multiplicity filtering, ICF/IDF weighting) and hosts the DR and NNDescent steps whose runtime and memory are benchmarked) — https://github.com/jzhang-dev/FEDRANN
- **minimap2** (Produces reference alignments used as ground truth for evaluating each DR method's predicted overlap sets)
- **Python** (Implementation language of Fedrann; performance parity with compiled tools is achieved via C-accelerated numerical libraries and batch-based matrix operations)

## Examples

```
fedrann -i reads.fasta.gz -o output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- Every method row in the comparison table contains both overlap-accuracy and resource (runtime, peak memory) measurements — reject the table if any cell is missing
- On the largest H3-class dataset (H3C, H3O, H3P), only SRP, GRP, and SimHash complete; verify PCA/UMAP-class methods fail within the predefined budget
- The fastest method (SRP) finishes under 6 hours wall-clock with peak memory under 1 TB on the largest dataset
- Each embedding's NNDescent search uses the cosine distance metric and a fixed random seed (--seed 602) so runs are reproducible
- Accuracy metrics for each method's overlap graph are computed against minimap2 reference alignments filtered at >= 100 matched bases and >= 10% alignment identity

## Limitations

- PCA and UMAP fail to scale effectively to human-genome-scale feature matrices, so accuracy comparison against them is not possible on the largest datasets
- Fedrann's memory consumption is higher than most existing overlap detection tools; memory-efficient representations and alternative ANN implementations are future work
- The DR methods were tested only on IDF-weighted k-mer features; conclusions may not transfer to other feature representations or to metagenomic binning and taxonomic classification, which have not been empirically validated
- Reproducibility depends solely on GitHub repositories; no DOI or Zenodo archive is provided

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [discussion] Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude: "Common dimensionality reduction techniques, including PCA and UMAP, also fail to scale effectively to matrices of this magnitude"
- [discussion] The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"
- [discussion] the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector: "the implementation fuses these steps by pre-multiplying the SRP projection matrix with the IDF weight vector"
- [methods] Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions: "Sparse Random Projection (SRP): Projects high-dimensional k-mer space to lower dimensions"
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"

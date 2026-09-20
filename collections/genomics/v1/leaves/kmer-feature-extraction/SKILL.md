---
name: kmer-feature-extraction
description: Use when converting long-read FASTQ/FASTA data into a sparse k-mer feature
  matrix as input to dimensionality reduction and ANN-based overlap detection, replicating
  Fedrann's FE stage.
when_to_use_negative:
- Input is already an embedding or feature matrix — this skill is the first FE stage
  and would duplicate feature construction
- Short-read data or tasks requiring base-level alignment rather than read-level overlap
  detection — Fedrann targets long-read overlap detection, not alignment
- Analysis requires exact similarity computation rather than the sparse, sampled k-mer
  representation
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: End-to-end pipeline implementing k-mer counting, filtering, sampling, and
    ICF weighting to build the feature matrix
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Jellyfish
  role: Efficient k-mer counting and multiplicity-based filtering before feature extraction
- name: Python
  role: Implementation language; sparse matrix construction and .npz output via C-accelerated
    numerical libraries
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:e48b8282030034a1653423156667937d256bdad2d7bec0406841bb6e6e43c179
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Jellyfish
  - Python
  - minimap2
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kmer-feature-extraction

## Summary

Builds sparse k-mer feature matrices from long-read sequencing data by counting k-mers (Jellyfish), filtering by minimum multiplicity to remove sequencing errors, sampling a k-mer fraction, and applying ICF/IDF weighting — the feature-extraction stage of the Fedrann overlap-detection pipeline.

## When to use

Use when converting long-read FASTQ/FASTA data into a sparse k-mer feature matrix as input to dimensionality reduction and ANN-based overlap detection, replicating Fedrann's FE stage.

## When NOT to use

- Input is already an embedding or feature matrix — this skill is the first FE stage and would duplicate feature construction
- Short-read data or tasks requiring base-level alignment rather than read-level overlap detection — Fedrann targets long-read overlap detection, not alignment
- Analysis requires exact similarity computation rather than the sparse, sampled k-mer representation

## Inputs

- FASTQ or FASTA file of long-read sequencing reads (possibly gzipped)
- k-mer size (e.g. -k 15; k=30 for H4C, k=50 for H4O datasets)
- k-mer minimum multiplicity threshold (e.g. 2)
- k-mer sampling fraction (e.g. 0.05)

## Outputs

- Sparse k-mer feature matrix (feature_matrix.npz) with one vector per read, ICF/IDF-weighted
- fedrann.log recording pipeline progress and resource usage
- k-mer library of sampled k-mers (forward and reverse complement)

## How to apply

Count all k-mers with Jellyfish and retain only k-mers whose multiplicities exceed a threshold (e.g. --kmer-min-multiplicity 2) to filter out sequencing-error k-mers. Optionally sample a fraction of the total k-mers (--kmer-sample-fraction 0.05) to build the k-mer library, covering both forward and reverse-complement strands, and locate k-mer positions per read with kmer_searcher to build the sparse vector per read. Weight k-mers by inverse collection frequency (ICF), using each k-mer's total count in the dataset as a computationally efficient approximation of document frequency (DF). Choose k per dataset (e.g. k=30 for H4C, k=50 for H4O human data; k=15 in the README example). Save the feature matrix in sparse .npz format via --save-feature-matrix for downstream dimensionality reduction.

## Related tools

- **Fedrann** (End-to-end pipeline implementing k-mer counting, filtering, sampling, and ICF weighting to build the feature matrix) — https://github.com/jzhang-dev/FEDRANN
- **Jellyfish** (Efficient k-mer counting and multiplicity-based filtering before feature extraction)
- **Python** (Implementation language; sparse matrix construction and .npz output via C-accelerated numerical libraries)

## Examples

```
fedrann -i reads.fasta.gz -o output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- fedrann.log shows k-mer counting, sampling, and ICF-weighting stages completed without error
- feature_matrix.npz is produced as a valid sparse file when --save-feature-matrix is set
- Filtered k-mers exclude those below the minimum multiplicity threshold (default: appearing less than 2 times)
- Feature matrix has one sparse vector per input read and supports downstream SRP projection (SRP_transpose × ICF precomputation)
- Fixed seed (--seed 602 in the README example) makes the sampled k-mer library and matrix reproducible across runs

## Limitations

- k-mer sampling (--kmer-sample-fraction) discards most k-mers, so the representation is approximate and seed-dependent
- Errors removed only via the multiplicity filter; residual sequencing-error k-mers can add noise to the feature space
- Downstream PCA and UMAP fail to scale to matrices of this magnitude; only sparse projections like SRP remain tractable

## Evidence

- [discussion] k-mer filtering by multiplicity threshold before feature extraction: "retaining only k-mers whose multiplicities exceed a predefined threshold to remove noise arising from sequencing errors"
- [discussion] IDF weighting approximation: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [readme] README CLI parameters: "Use only 5% of the total k-mers for feature extraction.
    --kmer-sample-fraction 0.05"
- [methods] multiplicity filter default: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [discussion] Jellyfish role: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"

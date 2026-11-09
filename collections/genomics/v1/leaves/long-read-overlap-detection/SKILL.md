---
name: long-read-overlap-detection
description: Use when you have a FASTQ/FASTA file of long reads (PacBio, ONT, BGI
  CycloneSEQ) and need to build an overlap graph for de novo assembly or benchmark
  overlap detection against tools like minimap2, MECAT2, MHAP, wtdbg2, xRead, or BLEND.
when_to_use_negative:
- Input is already a set of alignments or an overlap graph — this skill generates
  candidate overlaps from raw reads, not post-hoc overlap filtering
- Short-read data where all-vs-all k-mer embedding overlap detection is unnecessary;
  the pipeline targets long reads (>= 5 kb in the benchmark filtering)
- 'Memory-constrained environments: the paper notes Fedrann''s memory consumption
  is higher than most existing overlap detection tools'
edam_operation: http://edamontology.org/operation_3209
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0654
tools:
- name: Fedrann
  role: 'End-to-end overlap detection pipeline: k-mer counting/sampling, IDF+SRP embedding,
    NNDescent ANN search, and overlaps.tsv output'
  repo: https://github.com/jzhang-dev/fedrann
- name: Jellyfish
  role: Efficient k-mer counting and multiplicity filtering before feature extraction
- name: kmer_searcher
  role: C++ tool to locate sampled k-mer positions in reads and build sparse k-mer
    count vectors
  repo: https://github.com/jzhang-dev/FEDRANN
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows against minimap2, MHAP,
    MECAT2, wtdbg2, xRead, and BLEND
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Python
  role: Primary implementation language of Fedrann, leveraging C-accelerated numerical
    libraries and batch-based matrix operations
  repo: https://github.com/jzhang-dev/fedrann
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_006
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:f58111e58d8bc78e7653a7cbb7b73269be5d62660a4279bc2ab069f6d1ef2646
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/fedrann
  tools:
  - BLEND
  - Fedrann
  - Jellyfish
  - MECAT2
  - MHAP
  - Python
  - kNN-overlap-finder
  - kmer_searcher
  - minimap2
  - wtdbg2
  - xRead
  edam_operation: http://edamontology.org/operation_3209
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# long-read-overlap-detection

## Summary

Detect candidate overlaps between long sequencing reads by combining k-mer-based feature extraction with IDF weighting, sparse random projection (SRP) dimensionality reduction, and NNDescent approximate nearest-neighbor search, as implemented in the Fedrann pipeline for overlap-layout-consensus genome assembly.

## When to use

Use when you have a FASTQ/FASTA file of long reads (PacBio, ONT, BGI CycloneSEQ) and need to build an overlap graph for de novo assembly or benchmark overlap detection against tools like minimap2, MECAT2, MHAP, wtdbg2, xRead, or BLEND.

## When NOT to use

- Input is already a set of alignments or an overlap graph — this skill generates candidate overlaps from raw reads, not post-hoc overlap filtering
- Short-read data where all-vs-all k-mer embedding overlap detection is unnecessary; the pipeline targets long reads (>= 5 kb in the benchmark filtering)
- Memory-constrained environments: the paper notes Fedrann's memory consumption is higher than most existing overlap detection tools

## Inputs

- FASTQ or FASTA file of long sequencing reads (e.g. test/data/reads.fasta.gz)
- k-mer size parameter (-k, e.g. 15, or dataset-appropriate values such as k=30 for H4C and k=50 for H4O)
- Reference assembly for ground-truth evaluation (e.g. T2T-CHM13 v2.0, WBcel235 GCA_000002985.3, Drosophila Release 6 GCF_000001215.4)

## Outputs

- overlaps.tsv with columns query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance
- fedrann.log pipeline progress log
- feature_matrix.npz (optional sparse SRP embedding matrix)

## How to apply

Count k-mers with Jellyfish and filter k-mers whose multiplicity is below the minimum threshold (default 2, --kmer-min-multiplicity) to remove sequencing-error noise; sample 5% of total k-mers (--kmer-sample-fraction 0.05, --seed 602) to build a k-mer library covering both forward and reverse strands at a chosen k-mer size (e.g. k=15, or k=30/k=50 for human H4C/H4O datasets). Locate k-mer positions in reads with the C++ kmer_searcher tool to build sparse k-mer count vectors. Compute inverse collection frequency (ICF) weights as a practical approximation of IDF, pre-multiply them with the sparse random projection matrix (SRP_transpose x ICF), and apply the fused transformation with 32 threads to obtain normalized low-dimensional embeddings (optionally saved as feature_matrix.npz). Run NNDescent graph-based approximate k-NN search on the embeddings using cosine distance — NNDescent and HNSW gave the highest precision on SRP embeddings while NNDescent offered the best speed-accuracy trade-off — and write ranked neighbor pairs to overlaps.tsv. Validate that overlaps.tsv is a valid TSV with the six required columns and that fedrann.log records progress from start to finish.

## Related tools

- **Fedrann** (End-to-end overlap detection pipeline: k-mer counting/sampling, IDF+SRP embedding, NNDescent ANN search, and overlaps.tsv output) — https://github.com/jzhang-dev/fedrann
- **Jellyfish** (Efficient k-mer counting and multiplicity filtering before feature extraction)
- **kmer_searcher** (C++ tool to locate sampled k-mer positions in reads and build sparse k-mer count vectors) — https://github.com/jzhang-dev/FEDRANN
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows against minimap2, MHAP, MECAT2, wtdbg2, xRead, and BLEND) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Python** (Primary implementation language of Fedrann, leveraging C-accelerated numerical libraries and batch-based matrix operations) — https://github.com/jzhang-dev/fedrann

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- overlaps.tsv exists and is a valid TSV containing all six required columns (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- distance values are present and interpretable: smaller values indicate higher sequence similarity between embedded vectors
- fedrann.log records pipeline progress from start to finish
- For benchmarking, error rates are computed against ground-truth alignments filtered by >= 100 matched bases and >= 10% identity; the IDF + cosine configuration should yield the lowest error rate among weighting x distance combinations
- For full benchmarks, results table has one row per tool-dataset combination (7 tools x 5 datasets) with error rate, mean overlap size, #COC, #CC, runtime, and peak memory populated

## Limitations

- Memory consumption of Fedrann is higher than most existing overlap detection tools; memory-efficient data representations, shared-memory designs, or alternative ANN implementations are future work
- Common DR techniques such as PCA and UMAP fail to scale to human-genome-sized feature matrices, which constrains the choice of dimensionality reduction to SRP
- No empirical validation yet of FEDRANN for metagenomic binning, taxonomic classification, or other long-read applications
- Fedrann-Shasta whole-genome results are a proof-of-concept constrained by the internal architecture of established tools

## Evidence

- [discussion] The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"
- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [methods] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [readme] Use only 5% of the total k-mers for feature extraction: "Use only 5% of the total k-mers for feature extraction"
- [abstract] Fedrann achieves performance parity with tools written in compiled languages by leveraging C-accelerated numerical libraries and optimized batch-based matrix operations: "Fedrann achieves performance parity with tools written in compiled languages by leveraging C-accelerated numerical libraries and optimized batch-based matrix operations"

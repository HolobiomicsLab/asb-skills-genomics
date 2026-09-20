---
name: kmer-counting-and-multiplicity-filtering
description: 'Use when preprocessing long-read sequencing reads (FASTQ/FASTA) for
  embedding-based overlap detection: noisy k-mers must be removed before computing
  IDF-weighted sparse vectors for ANN search.'
when_to_use_negative:
- Input is already a feature matrix or embedding (feature_matrix.npz); this skill
  operates on raw sequencing reads only.
- Reads are very short and error-prone such that k=15 k-mers rarely exceed multiplicity
  2; multiplicity filtering would remove nearly all features.
- The goal is exact pairwise alignment rather than approximate k-NN candidate generation;
  alignment tools like minimap2 or BLAST are more appropriate.
edam_operation: http://edamontology.org/operation_0283
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Implements the k-mer counting, multiplicity filtering, and sampling stage
    within its feature-extraction step and builds sparse count vectors via its bundled
    kmer_searcher tool
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Jellyfish
  role: Used by the pipeline for efficient k-mer counting and filtering, retaining
    only k-mers whose multiplicities exceed a predefined threshold
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:d24dd114d1a92b09d634fa9500e011fef091ab95f60217ffd798e64a8e8bbb81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Jellyfish
  - Python
  edam_operation: http://edamontology.org/operation_0283
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# k-mer counting and multiplicity filtering

## Summary

Feature-extraction stage of the FEDRANN long-read overlap pipeline: count k-mers from FASTQ/FASTA reads with Jellyfish, filter out k-mers below a minimum multiplicity threshold to remove sequencing-error noise, then sample a fraction of the k-mers to build the k-mer library used for sparse count vectors.

## When to use

Use when preprocessing long-read sequencing reads (FASTQ/FASTA) for embedding-based overlap detection: noisy k-mers must be removed before computing IDF-weighted sparse vectors for ANN search.

## When NOT to use

- Input is already a feature matrix or embedding (feature_matrix.npz); this skill operates on raw sequencing reads only.
- Reads are very short and error-prone such that k=15 k-mers rarely exceed multiplicity 2; multiplicity filtering would remove nearly all features.
- The goal is exact pairwise alignment rather than approximate k-NN candidate generation; alignment tools like minimap2 or BLAST are more appropriate.

## Inputs

- FASTQ or FASTA file of long sequencing reads (e.g., reads.fasta.gz)
- k-mer size (k=15)
- k-mer sampling fraction (0.05)
- random seed (602)
- k-mer minimum multiplicity threshold (2)

## Outputs

- Sampled k-mer library covering forward and reverse strands
- Sparse k-mer count vectors per read (via kmer_searcher positions)
- fedrann.log recording pipeline progress from start to finish

## How to apply

Run Jellyfish to count all k-mers in the input reads with k-mer size 15 (-k 15) and discard k-mers whose multiplicity is below the minimum threshold of 2 (--kmer-min-multiplicity 2), since low-multiplicity k-mers arise from sequencing errors rather than genuine overlaps. From the filtered k-mer universe, sample 5% of the total k-mers (--kmer-sample-fraction 0.05) using a fixed random seed of 602 for reproducibility, and build the k-mer library for both forward and reverse strands. Locate k-mer positions in the reads with the C++ kmer_searcher tool (built via kmer_searcher/build.sh) to produce the sparse k-mer count vectors that feed downstream ICF weighting and sparse random projection. The rationale is that multiplicity filtering removes noise while k-mer sampling bounds the feature dimensionality so the pipeline scales to large read sets.

## Related tools

- **Fedrann** (Implements the k-mer counting, multiplicity filtering, and sampling stage within its feature-extraction step and builds sparse count vectors via its bundled kmer_searcher tool) — https://github.com/jzhang-dev/FEDRANN
- **Jellyfish** (Used by the pipeline for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold)

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --save-feature-matrix --threads 32
```

## Evaluation signals

- fedrann.log exists and records pipeline progress from start to finish through the k-mer counting and sampling stages.
- The k-mer library is built for both forward and reverse strands using the specified seed 602 and sample fraction 0.05.
- No k-mer in the retained library has multiplicity below the configured minimum of 2, confirming error-noise filtering took effect.
- The downstream kmer_searcher output yields a non-empty sparse count vector for each input read, formatted for subsequent ICF/SRP transformation.
- Re-running with identical -k, --kmer-sample-fraction, and --seed parameters reproduces the same sampled k-mer library (deterministic sampling).

## Limitations

- The minimum-multiplicity filter assumes sequencing errors produce singleton k-mers; very low-coverage or high-error-rate inputs may lose genuine k-mers.
- Sampling only 5% of total k-mers trades feature completeness for scale and depends on a fixed seed (602) for reproducibility.
- Jellyfish counting is an extra dependency that must be built separately (cd kmer_searcher && ./build.sh) and configured in the Conda environment (environment.yml).

## Evidence

- [discussion] The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold: "The pipeline utilizes Jellyfish for efficient k-mer counting and filtering, retaining only k-mers whose multiplicities exceed a predefined threshold"
- [methods] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [readme] Use only 5% of the total k-mers for feature extraction: "Use only 5% of the total k-mers for feature extraction"
- [discussion] retaining only k-mers whose multiplicities exceed a predefined threshold to remove noise arising from sequencing errors: "retaining only k-mers whose multiplicities exceed a predefined threshold to remove noise arising from sequencing errors"

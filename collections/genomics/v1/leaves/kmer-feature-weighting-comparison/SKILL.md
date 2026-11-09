---
name: kmer-feature-weighting-comparison
description: Use when benchmarking a k-mer-based sequence similarity pipeline and
  you must decide which feature weighting (TF, IDF, TF-IDF, raw) and distance metric
  (cosine vs Euclidean) to use for overlap detection or ANN search.
when_to_use_negative:
- You only need a production overlap detection run and the weighting choice is already
  fixed (e.g. the default IDF + cosine configuration) — benchmarking all 8 combinations
  is unnecessary
- Input is already an embedded/reduced feature matrix (dimensionality reduction already
  applied) — weighting operates on the raw k-mer count features before SRP
- You are evaluating assembly quality rather than overlap graph accuracy — use assembly
  contiguity/BUSCO metrics instead
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: Fedrann
  role: Executes the feature extraction (k-mer counting, sampling, ICF/TF/IDF/TF-IDF
    weighting), SRP dimensionality reduction, and NNDescent ANN search per weighting/metric
    configuration, producing overlaps.tsv
  repo: https://github.com/jzhang-dev/FEDRANN
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows used to score overlap graphs
    against reference truth and compute error rates
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Jellyfish
  role: Performs k-mer counting and multiplicity filtering prior to feature extraction
    in the Fedrann pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:26a65b6704f071fbd72dfd37d2f2c2c739b1923f962052b8a322ab8ae7d35f5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - Jellyfish
  - kNN-overlap-finder
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# k-mer feature weighting comparison for overlap graphs

## Summary

A benchmarking skill that systematically compares k-mer feature weighting schemes (raw/binary, TF, IDF/ICF, TF-IDF) crossed with similarity metrics (cosine vs Euclidean distance) in the Fedrann feature-extraction + NNDescent pipeline, scoring each overlap graph against truth overlaps to identify the most accurate configuration. IDF weighting with cosine distance yields the lowest overlap error rates.

## When to use

Use when benchmarking a k-mer-based sequence similarity pipeline and you must decide which feature weighting (TF, IDF, TF-IDF, raw) and distance metric (cosine vs Euclidean) to use for overlap detection or ANN search.

## When NOT to use

- You only need a production overlap detection run and the weighting choice is already fixed (e.g. the default IDF + cosine configuration) — benchmarking all 8 combinations is unnecessary
- Input is already an embedded/reduced feature matrix (dimensionality reduction already applied) — weighting operates on the raw k-mer count features before SRP
- You are evaluating assembly quality rather than overlap graph accuracy — use assembly contiguity/BUSCO metrics instead

## Inputs

- Long-read sequencing reads in FASTA or FASTQ format (e.g. test/data/reads.fasta.gz from the chr22 benchmark)
- Reference truth overlap set for the benchmark region
- Fedrann feature weighting options (raw/binary, TF, IDF via inverse collection frequency, TF-IDF)

## Outputs

- overlaps.tsv per configuration (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance)
- Overlap error rate for each of the 8 weighting x distance-metric combinations
- Single comparison table of error rates across all configurations

## How to apply

Run Fedrann once per weighting scheme on the same long-read FASTA/FASTQ input using fixed parameters (k=15, --kmer-sample-fraction 0.05, --kmer-min-multiplicity 2, --seed 602, --threads 32), each time executing the NNDescent approximate nearest neighbor search with cosine distance and collecting overlaps.tsv. Repeat the same weighting-scheme runs with NNDescent configured for Euclidean distance, yielding 8 weighting x metric configurations. Score every resulting overlap graph against reference truth overlaps, defining true overlaps with alignment filters of >= 100 matched bases and >= 10% alignment identity. Compute overlap error rates for all configurations and assemble a comparison table. The rationale is that IDF (approximated by inverse collection frequency, ICF) down-weights repetitive k-mers, and cosine distance is scale-invariant, so the IDF + cosine combination is expected to give the lowest error rate, consistent with the reported IDF + SRP + NNDescent pipeline accuracy.

## Related tools

- **Fedrann** (Executes the feature extraction (k-mer counting, sampling, ICF/TF/IDF/TF-IDF weighting), SRP dimensionality reduction, and NNDescent ANN search per weighting/metric configuration, producing overlaps.tsv) — https://github.com/jzhang-dev/FEDRANN
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows used to score overlap graphs against reference truth and compute error rates) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Jellyfish** (Performs k-mer counting and multiplicity filtering prior to feature extraction in the Fedrann pipeline)

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --threads 32
```

## Evaluation signals

- The IDF + cosine configuration must exhibit the lowest overlap error rate among all 8 weighting/distance combinations
- TF and raw/binary weighting runs should degrade performance relative to IDF, especially with cosine distance
- All 8 runs used identical k-mer parameters (k=15, sample fraction 0.05, min multiplicity 2, seed 602) so error-rate differences are attributable to weighting/metric only
- Truth overlap definition is consistent: alignments with >= 100 matched bases and >= 10% alignment identity
- Each overlaps.tsv is a valid TSV with columns query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance and lower distance values for better matches

## Limitations

- Euclidean-distance configurations cannot be fused with ICF weighting via the pre-multiplied SRP projection matrix as cosine configurations can, potentially confounding the comparison
- Results were established on simulated and chr22-scale benchmarks; scaling to whole-genome feature matrices requires sparse random projection because PCA/UMAP fail to scale
- NNDescent is approximate, so error rates carry sampling variance tied to the random seed

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [discussion] All alignments were filtered using two criteria: (1) >= 100 matched bases and (2) >= 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [discussion] Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"

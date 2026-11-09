---
name: ann-search-error-rate-benchmarking
description: Use when comparing how k-mer feature weighting schemes and ANN distance
  metrics affect overlap graph accuracy on long-read sequencing data, or when validating
  that an IDF + cosine NNDescent configuration minimizes overlap error rates.
when_to_use_negative:
- The goal is assembly quality (contiguity/BUSCO) rather than overlap graph accuracy
  — use the Fedrann-Shasta integration workflow instead.
- Reads lack reference truth overlaps, so error rates against ground truth cannot
  be computed; de novo evaluation signals would be needed.
- The feature weighting and distance metric have already been fixed (e.g. production
  IDF + cosine runs); benchmarking the full grid would be redundant.
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Fedrann
  role: Runs the k-mer feature extraction (with configurable weighting) and NNDescent
    approximate nearest neighbor search per configuration, emitting overlaps.tsv
  repo: https://github.com/jzhang-dev/FEDRANN
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows for scoring overlap graphs
    against truth and computing error rates
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:27667fddcf007e5a8f58bd1214db068f7bd48dde62502544f10238da2a72ec6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - kNN-overlap-finder
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ANN search error rate benchmarking

## Summary

Benchmarks approximate nearest neighbor (NNDescent) overlap detection by systematically varying k-mer feature weighting schemes (raw, TF, IDF/ICF, TF-IDF) and distance metrics (cosine vs Euclidean), then scoring the resulting overlap graphs against reference truth overlaps to compute error rates.

## When to use

Use when comparing how k-mer feature weighting schemes and ANN distance metrics affect overlap graph accuracy on long-read sequencing data, or when validating that an IDF + cosine NNDescent configuration minimizes overlap error rates.

## When NOT to use

- The goal is assembly quality (contiguity/BUSCO) rather than overlap graph accuracy — use the Fedrann-Shasta integration workflow instead.
- Reads lack reference truth overlaps, so error rates against ground truth cannot be computed; de novo evaluation signals would be needed.
- The feature weighting and distance metric have already been fixed (e.g. production IDF + cosine runs); benchmarking the full grid would be redundant.

## Inputs

- Long-read sequencing reads (FASTA/FASTQ, e.g. test/data/reads.fasta.gz)
- Reference truth overlap sets (e.g. alignment-based ground truth with known read positions)
- Fedrann installation (github.com/jzhang-dev/FEDRANN) and kNN-overlap-finder benchmarking workflows (github.com/jzhang-dev/kNN-overlap-finder)

## Outputs

- overlaps.tsv per configuration (query/target names, orientations, neighbor_rank, distance)
- Comparison table of overlap error rates across all 8 weighting x distance-metric configurations

## How to apply

Run Fedrann on the long-read FASTA/FASTQ input once per feature weighting scheme (raw/binary, TF, IDF approximated by inverse collection frequency, TF-IDF), keeping parameters fixed (e.g. k=15, --kmer-sample-fraction 0.05, --kmer-min-multiplicity 2, --seed 602, --threads 32) and collecting each run's overlaps.tsv from the NNDescent search. Repeat the full weighting-scheme grid with the NNDescent distance metric switched from cosine to Euclidean, yielding 8 weighting x metric configurations. Score every resulting overlap graph against reference truth overlaps, defining true overlaps with alignment filters of >= 100 matched bases and >= 10% alignment identity. Compute overlap error rates for all configurations and assemble a comparison table; the IDF + cosine combination should show the lowest error rate, consistent with the reported IDF + SRP + NNDescent pipeline accuracy.

## Related tools

- **Fedrann** (Runs the k-mer feature extraction (with configurable weighting) and NNDescent approximate nearest neighbor search per configuration, emitting overlaps.tsv) — https://github.com/jzhang-dev/FEDRANN
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows for scoring overlap graphs against truth and computing error rates) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
fedrann -i test/data/reads.fasta.gz -o test/output/ -k 15 --kmer-sample-fraction 0.05 --kmer-min-multiplicity 2 --seed 602 --threads 32 --save-feature-matrix
```

## Evaluation signals

- The IDF + cosine configuration exhibits the lowest overlap error rate among all 8 tested weighting/distance combinations.
- TF and raw/unweighted representations degrade overlap performance relative to IDF-weighted runs under cosine distance.
- Each of the 8 runs produces an overlaps.tsv with the expected columns (query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance) and smaller distances indicating higher similarity.
- Truth-overlap scoring applies both alignment filters (>= 100 matched bases AND >= 10% alignment identity); changing either filter changes the error-rate table.
- All runs use identical sampling parameters (--kmer-sample-fraction 0.05, --seed 602, --kmer-min-multiplicity 2) so configuration differences are attributable only to weighting and distance metric.

## Limitations

- IDF is approximated by the total count (collection frequency, CF) of a k-mer in the dataset rather than true document frequency, which introduces approximation error.
- Fedrann's memory consumption is higher than most existing overlap detection tools; memory-efficient representations and alternative ANN implementations remain future work.
- Reproducibility depends solely on the GitHub repositories (e.g. kNN-overlap-finder); no DOI or Zenodo archive is provided.
- NNDescent results depend on the random seed; the documented --seed 602 should be reused for reproducibility.

## Evidence

- [abstract] the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets: "the pipeline combining inverse document frequency (IDF) transformation, sparse random projection (SRP), and NNDescent enables accurate detection of overlaps across diverse datasets"
- [discussion] All alignments were filtered using two criteria: (1) >= 100 matched bases and (2) >= 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [discussion] Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF: "Fedrann therefore adopts the total count (i.e. collection frequency, CF) of a k-mer within the dataset as a computationally efficient approximation of its DF"
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [readme] distance: Measures the dissimilarity between the embedded vectors of the query and target sequences. A smaller value indicates higher similarity: "Measures the dissimilarity between the embedded vectors of the query and target sequences. A smaller value indicates higher similarity between the sequences."

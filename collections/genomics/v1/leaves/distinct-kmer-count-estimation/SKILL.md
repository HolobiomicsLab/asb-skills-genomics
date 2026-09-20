---
name: distinct-kmer-count-estimation
description: Use when short-read FASTQ datasets need a distinct k-mer count to compute
  an optimal Bloom filter size (b) for k-mer-based tools like ntEdit+Sealer at a chosen
  false-positive rate.
when_to_use_negative:
- Reads have not yet been histogrammed by ntCard — the estimation requires k-mer frequency
  histograms (from an upstream ntCard/ntHits step), not raw FASTQ alone.
- The downstream tool does not accept Bloom filter sizing in suffix notation (e.g.
  ABySS-Bloom b=200M), or the analysis uses alignment-based polishing tools like Pilon
  or Racon that do not use Bloom filters.
- K-mer sizes other than the protocol's descending set are used without re-histogramming,
  since n must be recomputed per k.
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0080
tools:
- name: ntHits
  role: Part of the toolchain for creating Bloom filters from short-read k-mers; ntHits
    k-mer frequency counting underpins the distinct k-mer estimation
  repo: https://github.com/BirolLab/nthits
- name: ABySS-Bloom
  role: Consumes the computed Bloom filter size (b) to build Bloom filters from short-read
    k-mers for the ntEdit+Sealer pipeline
  repo: https://github.com/BirolLab/abyss
- name: ntEdit
  role: 'Downstream consumer: the Bloom filter sized by this estimation feeds the
    ntEdit+Sealer Makefile pipeline (b parameter)'
  repo: https://github.com/BirolLab/ntEdit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:f1b4660ffff55547b70f3603f8ce16b693f58d33e0f27a91623f3b621af2fcce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/nthits
  tools:
  - ABySS-Bloom
  - ntEdit
  - ntHits
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# distinct-kmer-count-estimation

## Summary

Estimates the number of distinct (non-erroneous) k-mers in short-read data from ntCard k-mer frequency histograms, by subtracting singleton counts from the estimated total F0 per k-mer size, and uses the largest estimate to size a Bloom filter at a target false-positive rate.

## When to use

Use when short-read FASTQ datasets need a distinct k-mer count to compute an optimal Bloom filter size (b) for k-mer-based tools like ntEdit+Sealer at a chosen false-positive rate.

## When NOT to use

- Reads have not yet been histogrammed by ntCard — the estimation requires k-mer frequency histograms (from an upstream ntCard/ntHits step), not raw FASTQ alone.
- The downstream tool does not accept Bloom filter sizing in suffix notation (e.g. ABySS-Bloom b=200M), or the analysis uses alignment-based polishing tools like Pilon or Racon that do not use Bloom filters.
- K-mer sizes other than the protocol's descending set are used without re-histogramming, since n must be recomputed per k.

## Inputs

- Short-read FASTQ files (e.g. reads_1.fq.gz, reads_2.fq.gz)
- ntCard k-mer count histograms for each k in the protocol's k-mer set (k=80, k=65, k=50)

## Outputs

- Per-k distinct non-erroneous k-mer count n (e.g. n=6,720,950 for k=65)
- Optimal Bloom filter size in bits m (e.g. 1,340,826,718 bits ~ 168 MB)
- bloom_filter_size.txt with a practical suffix-notation value (e.g. 200M) used as the b parameter of the ntEdit-Sealer Makefile pipeline

## How to apply

Run ntCard on the short reads for the k-mer sizes used by the protocol (k=80, k=65, k=50) to produce k-mer count histograms. For each k, compute n as F0 (estimated total distinct k-mers) minus the number of k-mers appearing exactly once, which removes erroneous singleton k-mers. Confirm which k-mer size yields the largest n — in the ntEdit+Sealer protocol k=65 gives the largest n (6,720,950), consistent with its descending k-mer ordering (k='80 65 50'). Apply the Bloom filter sizing formula m = Ceil(-n/ln(1-f)) with false-positive probability f=0.005, giving m=1,340,826,718 bits (~168 MB) for the E. coli NDM5 dataset. Round the computed value up to a practical memory setting in the suffix notation accepted by ABySS-Bloom (e.g. 200M) and write it to bloom_filter_size.txt as the b parameter of the Makefile pipeline.

## Related tools

- **ntHits** (Part of the toolchain for creating Bloom filters from short-read k-mers; ntHits k-mer frequency counting underpins the distinct k-mer estimation) — https://github.com/BirolLab/nthits
- **ABySS-Bloom** (Consumes the computed Bloom filter size (b) to build Bloom filters from short-read k-mers for the ntEdit+Sealer pipeline) — https://github.com/BirolLab/abyss
- **ntEdit** (Downstream consumer: the Bloom filter sized by this estimation feeds the ntEdit+Sealer Makefile pipeline (b parameter)) — https://github.com/BirolLab/ntEdit

## Examples

```
ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=200M
```

## Evaluation signals

- For each k, n equals F0 minus the number of k-mers appearing exactly once in the ntCard histogram (singletons removed as errors).
- The k-mer size with the largest n is confirmed (k=65 in the protocol), consistent with the descending k-mer ordering k='80 65 50'.
- m = Ceil(-n/ln(1-f)) with f=0.005 reproduces ~1,340,826,718 bits (~168 MB) for n=6,720,950.
- The final b value written to bloom_filter_size.txt uses suffix notation accepted by ABySS-Bloom (e.g. 200M) and is >= the computed m in bytes.
- The downstream Makefile invocation accepts the b value without parameter errors (e.g. ntedit-sealer finish ... b=200M).

## Limitations

- Singleton k-mers are assumed to be erroneous; residual sequencing error or true single-copy k-mers affect the n estimate.
- The computed m must be rounded up to a practical memory setting (e.g. 168 MB rounded to 200 MB), so the actual FPR is slightly below the target 0.005.
- n depends on the k-mer size and dataset; the estimate must be recomputed for new datasets or k sets rather than reused.

## Evidence

- [intro] Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers: "Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [intro] This protocol relies on Bloom filters, a probabilistic data structure that stores and tests for elements in a set in constant time (Bloom, 1970).: "This protocol relies on Bloom filters, a probabilistic data structure that stores and tests for elements in a set in constant time (Bloom, 1970)."
- [readme] ntHits is a tool for efficiently counting and filtering k-mers based on their frequencies.: [evidence span withheld — anchor status: external_doc]

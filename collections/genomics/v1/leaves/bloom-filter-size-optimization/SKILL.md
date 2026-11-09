---
name: bloom-filter-size-optimization
description: Use when ntCard k-mer histograms (freq_k80.hist, freq_k65.hist, freq_k50.hist)
  are available for the ntEdit+Sealer protocol and a target false-positive rate (e.g.
  0.005) must be translated into a Bloom filter size b.
when_to_use_negative:
- No ntCard histograms exist for the protocol k-mer sizes — run ntCard first (task_003)
  since n must be derived from F0 and the singleton counts
- The assembly finishing tool does not consume a Bloom filter (e.g. alignment-based
  polishers like Pilon or Racon), where b has no parameter to tune
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntCard
  role: Generates the k-mer frequency histograms (F0/F1 and per-frequency counts)
    from which n is computed
  repo: https://github.com/BirolLab/ntCard
- name: ntHits
  role: Required, together with ABySS-Bloom, for creating the Bloom filters from short-read
    k-mers at the computed size
  repo: https://github.com/bcgsc/nthits
- name: ABySS-Bloom
  role: Consumes the rounded Bloom filter size (e.g. b=200M) as the b parameter of
    the ntEdit-Sealer Makefile pipeline
  repo: https://github.com/bcgsc/abyss
- name: ntEdit+Sealer
  role: Protocol whose Makefile pipeline uses the optimized b value; ntEdit corrects
    errors and Sealer fills gaps via Bloom filters
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:c30a5ac1bcb6260a134e2c7adbf6dc6ad3349ddac6db6f9db86f13a85237b8c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntCard
  tools:
  - ABySS-Bloom
  - ntCard
  - ntEdit
  - ntEdit+Sealer
  - ntHits
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# bloom-filter-size-optimization

## Summary

Computes the optimal Bloom filter size (b) for the ntEdit+Sealer Makefile pipeline from ntCard k-mer frequency histograms using m = Ceil(-n/ln(1-f)), then rounds the result to a practical ABySS-Bloom memory setting (e.g. b=200M).

## When to use

Use when ntCard k-mer histograms (freq_k80.hist, freq_k65.hist, freq_k50.hist) are available for the ntEdit+Sealer protocol and a target false-positive rate (e.g. 0.005) must be translated into a Bloom filter size b.

## When NOT to use

- No ntCard histograms exist for the protocol k-mer sizes — run ntCard first (task_003) since n must be derived from F0 and the singleton counts
- The assembly finishing tool does not consume a Bloom filter (e.g. alignment-based polishers like Pilon or Racon), where b has no parameter to tune

## Inputs

- ntCard k-mer frequency histograms (freq_k80.hist, freq_k65.hist, freq_k50.hist) from task_003, two-column tab-separated files with F0/F1 metric lines in the first two rows
- Target false-positive rate f (protocol uses f = 0.005)
- Protocol k-mer sizes used by the pipeline (k=80, 65, 50)

## Outputs

- Optimal Bloom filter size m in bits (e.g. 1,340,826,718 bits ~168 MB)
- Rounded memory setting in ABySS-Bloom suffix notation (e.g. b=200M) written to bloom_filter_size.txt

## How to apply

Load the ntCard histograms generated with -k80,65,50 on the paired-end FASTQ files; each two-column tab-separated histogram starts with Fk summary lines (F0: distinct k-mers, F1: total k-mers). For each k, compute n as F0 minus the number of k-mers appearing exactly once, removing erroneous singleton k-mers, and take the largest n across tested k values (here k=65 is largest, consistent with the protocol's descending k-mer ordering of 80, 65, 50). Apply the Bloom filter sizing formula m = Ceil(-n/ln(1-f)) with f = 0.005, yielding m = 1,340,826,718 bits (~168 MB) for the demo E. coli NDM5 dataset. Round this up to a practical memory setting accepted by ABySS-Bloom suffix notation and write it to bloom_filter_size.txt for use as the b parameter of the ntEdit-Sealer Makefile pipeline.

## Related tools

- **ntCard** (Generates the k-mer frequency histograms (F0/F1 and per-frequency counts) from which n is computed) — https://github.com/BirolLab/ntCard
- **ntHits** (Required, together with ABySS-Bloom, for creating the Bloom filters from short-read k-mers at the computed size) — https://github.com/bcgsc/nthits
- **ABySS-Bloom** (Consumes the rounded Bloom filter size (e.g. b=200M) as the b parameter of the ntEdit-Sealer Makefile pipeline) — https://github.com/bcgsc/abyss
- **ntEdit+Sealer** (Protocol whose Makefile pipeline uses the optimized b value; ntEdit corrects errors and Sealer fills gaps via Bloom filters) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
ntcard -k80,65,50 -o freq reads_1.fq.gz reads_2.fq.gz
```

## Evaluation signals

- All three histogram files (freq_k80.hist, freq_k65.hist, freq_k50.hist) exist, are two-column tab-separated, and have F0 and F1 metric lines on the first two rows
- k=65 yields the largest n among the tested k-mer sizes after subtracting k-mers appearing exactly once from F0
- Computed m = Ceil(-n * ln(0.995)) reproduces 1,340,826,718 bits (~168 MB) for the demo dataset
- bloom_filter_size.txt contains a value in suffix notation accepted by ABySS-Bloom (e.g. 200M) that is >= the computed m
- The ntEdit-Sealer Makefile pipeline accepts b=<rounded value> without errors at FPR ~0.005

## Limitations

- The optimal b is dataset-specific: it depends on the distinct solid k-mer count of the input reads, so it must be recomputed for each new dataset rather than copied
- Rounding up (168 MB to 200 MB) trades memory for a slightly lower false-positive rate than the 0.005 target
- The formula relies on the ntCard F0 estimate minus singleton k-mers; residual erroneous k-mers not appearing exactly once can inflate n

## Evidence

- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"
- [other] Running ntcard with -k80,65,50 on both FASTQ read files generates a k-mer distribution histogram for each k: [evidence span withheld — anchor status: not_found]
- [intro] This protocol relies on Bloom filters, a probabilistic data structure that stores and tests for elements in a set in constant time: "This protocol relies on Bloom filters, a probabilistic data structure that stores and tests for elements in a set in constant time (Bloom, 1970)."

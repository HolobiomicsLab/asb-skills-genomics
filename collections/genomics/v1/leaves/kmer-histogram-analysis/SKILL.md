---
name: kmer-histogram-analysis
description: Use when you need to size a Bloom filter for k-mer-based assembly finishing
  (e.g., ntEdit+Sealer) from short-read FASTQ data, by measuring the k-mer distribution
  with ntCard histograms.
when_to_use_negative:
- The draft assembly and read data are already accompanied by a validated Bloom filter
  size from a previous run of the same dataset
- The pipeline is an alignment-based polisher (e.g., Pilon or Racon) that does not
  consume a Bloom filter parameter b
- Reads are long reads only — the protocol builds Bloom filters from short-read k-mers
  via ntHits and ABySS-Bloom
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntCard
  role: Generates the k-mer frequency histograms for each k value in a single pass
    over the reads
  repo: https://github.com/BirolLab/ntCard
- name: ntHits
  role: Required, together with ABySS-Bloom, for creating Bloom filters from short-read
    k-mers in the protocol
  repo: https://github.com/bcgsc/nthits
- name: ABySS-Bloom
  role: Builds the Bloom filter using the b size computed here; accepts suffix-notation
    memory settings (e.g., 200M)
  repo: https://github.com/bcgsc/abyss
- name: ntEdit
  role: Consumes the Bloom filter built from the short-read k-mers during error correction
  repo: https://github.com/bcgsc/ntEdit
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:0c3524c2b233dec89f790d0d5043253a80daca0a403efd9a54a75f3dad67d5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntCard
  tools:
  - ABySS-Bloom
  - ntCard
  - ntEdit
  - ntHits
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# kmer-histogram-analysis

## Summary

Generates k-mer frequency histograms with ntCard across multiple k-mer sizes and uses them to compute the optimal Bloom filter size (b) for the ntEdit+Sealer Makefile pipeline via the formula m = Ceil(-n/ln(1-f)).

## When to use

Use when you need to size a Bloom filter for k-mer-based assembly finishing (e.g., ntEdit+Sealer) from short-read FASTQ data, by measuring the k-mer distribution with ntCard histograms.

## When NOT to use

- The draft assembly and read data are already accompanied by a validated Bloom filter size from a previous run of the same dataset
- The pipeline is an alignment-based polisher (e.g., Pilon or Racon) that does not consume a Bloom filter parameter b
- Reads are long reads only — the protocol builds Bloom filters from short-read k-mers via ntHits and ABySS-Bloom

## Inputs

- Paired-end short-read FASTQ files (forward and reverse), gzip-compressed with .fq.gz extensions (e.g., SRR15859208 E. coli NDM5)
- ntCard k-mer frequency histograms (freq_k80.hist, freq_k65.hist, freq_k50.hist)

## Outputs

- Two-column tab-separated k-mer frequency histogram files (freq_k{80,65,50}.hist) with F0/F1 metric lines
- bloom_filter_size.txt containing the Bloom filter size b (e.g., 200M) for the ntEdit-Sealer Makefile pipeline

## How to apply

Run ntCard once on the paired-end FASTQ files (e.g., gzip-compressed .fq.gz from SRR15859208 E. coli NDM5) with -k80,65,50 so k-mer counting is done jointly for all k sizes in a single pass, producing freq_k80.hist, freq_k65.hist, and freq_k50.hist (two-column, tab-separated files whose first two rows hold F0, distinct k-mers, and F1, total k-mers). For each k, compute n as F0 minus the number of k-mers appearing exactly once, which removes erroneous singleton k-mers. Select the k yielding the largest n — here k=65, consistent with the protocol's descending k-mer ordering (80, 65, 50) — then apply m = Ceil(-n/ln(1-f)) with false-positive probability f = 0.005, giving m = 1,340,826,718 bits (~168 MB). Round up to a practical suffix-notation setting accepted by ABySS-Bloom (b=200M) and write it to bloom_filter_size.txt for use as the b parameter of the ntEdit-Sealer Makefile pipeline.

## Related tools

- **ntCard** (Generates the k-mer frequency histograms for each k value in a single pass over the reads) — https://github.com/BirolLab/ntCard
- **ntHits** (Required, together with ABySS-Bloom, for creating Bloom filters from short-read k-mers in the protocol) — https://github.com/bcgsc/nthits
- **ABySS-Bloom** (Builds the Bloom filter using the b size computed here; accepts suffix-notation memory settings (e.g., 200M)) — https://github.com/bcgsc/abyss
- **ntEdit** (Consumes the Bloom filter built from the short-read k-mers during error correction) — https://github.com/bcgsc/ntEdit

## Examples

```
ntcard -k80,65,50 reads_1.fq.gz reads_2.fq.gz -o freq
```

## Evaluation signals

- All three histogram files (freq_k80.hist, freq_k65.hist, freq_k50.hist) exist and are two-column tab-separated files
- The first two rows of each histogram contain the Fk summary metrics F0 (distinct k-mers) and F1 (total k-mers)
- The chosen k (k=65 here) yields the largest n (F0 minus singletons) among the tested k-mer sizes, consistent with the protocol's descending k-mer ordering
- The computed Bloom filter size matches m = Ceil(-n × ln(1/0.995)); for n = 6,720,950 this is 1,340,826,718 bits (~168 MB)
- bloom_filter_size.txt contains a suffix-notation value (e.g., b=200M) accepted by ABySS-Bloom and larger than the computed minimum

## Limitations

- The histogram step must rerun whenever the read set or k-mer sizes change, since n depends on the specific dataset
- The false-positive rate is fixed by the protocol design (f ≈ 0.005); other applications would need to re-derive m for their target FPR
- The computed size (~168 MB) is a minimum; the practical setting (200 MB) is a manual rounding decision, not an automated optimization

## Evidence

- [other] Running ntcard with -k80,65,50 on both FASTQ read files generates a k-mer distribution histogram for each k, printed to a two-column, tab-separated file with the prefix freq (e.g., freq_k80.hist).: [evidence span withheld — anchor status: not_found]
- [other] Using the largest n (k=65 k-mers appearing ≥2 times) and an FPR of 0.005, the optimal size is m=Ceil(−6720950×ln(0.995))=1340826718 bits (~168 MB), which was rounded up to 200 MB.: [evidence span withheld — anchor status: not_found]
- [intro] Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers: "Additionally, ntHits (Mohamadi, Chu, Coombe, Warren, & Birol, 2020) and ABySS-Bloom (Jackman et al., 2017) are required for creating Bloom filters from short read k-mers"
- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"

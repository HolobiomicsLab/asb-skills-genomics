---
name: kmer-size-optimization-for-genome-finishing
description: Use when preparing to run ntEdit+Sealer on a draft long-read assembly
  with short reads and you need to select optimal k-mer lengths and Bloom filter size
  (b) from k-mer distribution histograms.
when_to_use_negative:
- You already have k-mer histograms or Bloom filter parameters for the reads and only
  need to run the ntedit-sealer finish pipeline
- The task is hybrid read-alignment polishing (e.g., Pilon or Racon) rather than the
  alignment-free Bloom-filter-based ntEdit+Sealer workflow that requires k-mer/Bloom
  filter parameter selection
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: ntCard
  role: Generates the k-mer frequency histograms for each requested k in a single
    pass over the paired-end FASTQ reads
  repo: https://github.com/BirolLab/ntCard
- name: ntEdit+Sealer
  role: Downstream Makefile finishing pipeline whose k and Bloom filter size (b) parameters
    are selected using the ntCard histograms
  repo: https://github.com/BirolLab/ntedit_sealer_protocol
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/cpz1.442
    title: 'ntEdit+Sealer: Efficient Targeted Error Resolution and Automated Finishing
      of Long‐Read Genome Assemblies'
  source_content_hash: sha256:b593d64c7a4935ca1cdc6c404b20912dd9f682de68d81b9b9891795f98eafa54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/ntCard
  tools:
  - ntCard
  - ntEdit
  - ntEdit+Sealer
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# kmer-size-optimization-for-genome-finishing

## Summary

Generates k-mer frequency histograms with ntCard across multiple k-mer lengths (e.g., k=80, 65, 50) on paired-end short reads to inform the choice of k-mer lengths and Bloom filter size (b) for the ntEdit+Sealer genome finishing protocol.

## When to use

Use when preparing to run ntEdit+Sealer on a draft long-read assembly with short reads and you need to select optimal k-mer lengths and Bloom filter size (b) from k-mer distribution histograms.

## When NOT to use

- You already have k-mer histograms or Bloom filter parameters for the reads and only need to run the ntedit-sealer finish pipeline
- The task is hybrid read-alignment polishing (e.g., Pilon or Racon) rather than the alignment-free Bloom-filter-based ntEdit+Sealer workflow that requires k-mer/Bloom filter parameter selection

## Inputs

- Paired-end short-read FASTQ files (gzip-compressed, .fq.gz), e.g., forward and reverse reads of the SRR15859208 E. coli NDM5 dataset
- List of k-mer sizes to profile, e.g., 80, 65, 50

## Outputs

- freq_k80.hist, freq_k65.hist, freq_k50.hist — two-column, tab-separated k-mer frequency histograms with F0 (distinct k-mers) and F1 (total k-mers) on the first two lines
- Informed choices of k-mer lengths (k) and Bloom filter size (b) for the ntEdit+Sealer run

## How to apply

Obtain the paired-end short-read FASTQ files (gzip-compressed, .fq.gz) for the dataset, e.g., the SRR15859208 E. coli NDM5 reads. Run ntCard once on both the forward and reverse read files with -k80,65,50 so k-mer counting is performed jointly for all three k-mer sizes in a single pass. ntCard outputs one two-column, tab-separated histogram per k with the prefix freq (freq_k80.hist, freq_k65.hist, freq_k50.hist), where the first two lines contain the Fk summary metrics (F0: distinct k-mers; F1: total k-mers). Inspect the histogram peaks/valleys per k to choose k-mer lengths for the finishing run (e.g., k='80 65 50') and to size the Bloom filter (e.g., b=5G) in the Makefile pipeline invocation: ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G.

## Related tools

- **ntCard** (Generates the k-mer frequency histograms for each requested k in a single pass over the paired-end FASTQ reads) — https://github.com/BirolLab/ntCard
- **ntEdit+Sealer** (Downstream Makefile finishing pipeline whose k and Bloom filter size (b) parameters are selected using the ntCard histograms) — https://github.com/BirolLab/ntedit_sealer_protocol

## Examples

```
ntcard -k80,65,50 reads_1.fq.gz reads_2.fq.gz -p freq
```

## Evaluation signals

- All three histogram files exist: freq_k80.hist, freq_k65.hist, freq_k50.hist
- Each histogram file is a tab-separated, two-column file
- The first two rows of each histogram contain the F0 (distinct k-mers) and F1 (total k-mers) metric lines
- The chosen k values and Bloom filter size passed to ntedit-sealer finish (e.g., k='80 65 50', b=5G) are consistent with the histogram evidence from the reads

## Limitations

- Histograms are only informative for the k values explicitly requested via -k; other k-mer lengths must be re-profiled
- ntCard estimates k-mer distributions but does not itself pick k or b — the selection decision must be made by inspecting the histograms

## Evidence

- [other] Running ntcard with -k80,65,50 on both FASTQ read files generates a k-mer distribution histogram for each k, printed to a two-column, tab-separated file with the prefix freq (e.g., freq_k80.hist).: [evidence span withheld — anchor status: not_found]
- [abstract] Selecting optimal values for k-mer lengths (k) and Bloom filter size (b): "Selecting optimal values for k-mer lengths (k) and Bloom filter size (b)"
- [methods] ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G: "ntedit-sealer finish seqs=draft-assembly.fa reads='reads_1.fq.gz reads_2.fq.gz' k='80 65 50' b=5G"

---
name: wgs-read-filtering-against-transcriptome
description: Use when you have an assembled transcriptome (FASTA) plus raw WGS reads
  (FASTQ, gzipped allowed) and need to cheaply discard reads outside the transcriptome
  before splice-graph or k-mer analyses, without alignment.
when_to_use_negative:
- You only have raw WGS reads with no assembled transcriptome to bait the Bloom filter
  with
- You need mapping locations for the retained reads — biobloomcategorizer is faster
  than alignment but does not provide mapping positions
- The transcriptome is of poor quality or assembled from a single tissue, so filtering
  may discard reads from transcripts absent or fragmented in the bait set
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: biobloommaker
  role: Builds the transcriptome Bloom filter (.bf + .txt) with k-mer length 25 bp
    and specified bloom size/levels
  repo: https://github.com/bcgsc/biobloom
- name: biobloomcategorizer
  role: Classifies WGS reads against the transcriptome Bloom filter, discarding reads
    that share no k-mer with the transcriptome
  repo: https://github.com/bcgsc/biobloom
- name: EXFI
  role: Encapsulates this filtering as the build_baited_bloom_filter module, the initial
    stage of its splice-graph pipeline
  repo: https://github.com/jlanga/exfi
- name: Docker (exfi-docker)
  role: Containerized execution of build_baited_bloom_filter with all dependencies
  repo: https://github.com/jlanga/exfi-docker
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:7e4e867f7d35ab9875b6dae3728baaefc19e86a1a55ed13b82c26a30c281060e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/biobloom
  tools:
  - Docker (exfi-docker)
  - EXFI
  - Python
  - biobloomcategorizer
  - biobloommaker
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# wgs-read-filtering-against-transcriptome

## Summary

Builds a transcriptome-baited Bloom filter (biobloommaker, k=25) and uses biobloomcategorizer to discard WGS reads that do not share at least one k-mer with the assembled transcriptome, as the initial stage of EXFI's build_baited_bloom_filter module.

## When to use

Use when you have an assembled transcriptome (FASTA) plus raw WGS reads (FASTQ, gzipped allowed) and need to cheaply discard reads outside the transcriptome before splice-graph or k-mer analyses, without alignment.

## When NOT to use

- You only have raw WGS reads with no assembled transcriptome to bait the Bloom filter with
- You need mapping locations for the retained reads — biobloomcategorizer is faster than alignment but does not provide mapping positions
- The transcriptome is of poor quality or assembled from a single tissue, so filtering may discard reads from transcripts absent or fragmented in the bait set

## Inputs

- Assembled transcriptome in FASTA format
- Raw whole-genome sequencing (WGS) reads in FASTQ format (gzipped allowed)

## Outputs

- Transcriptome Bloom filter file (.bf) plus companion INI information file (.txt) from biobloommaker
- Filtered FASTQ of retained reads that share at least one k-mer with the transcriptome
- biobloomcategorizer summary.tsv and readStatus.tsv classification summaries

## How to apply

Run the build_baited_bloom_filter module: first load the transcriptome FASTA and raw WGS FASTQ, then build a Bloom filter of the transcriptome with biobloommaker using a k-mer length of 25 bp and a specified bloom size (e.g. 100M) and levels (e.g. 1). Then categorize the WGS reads against the transcriptome Bloom filter with biobloomcategorizer, discarding every read that does not share at least one k-mer with the transcriptome, and write retained (exonic and flanking) reads to an output FASTQ. The rationale is that Bloom filters return no false negatives but tunable false positives (BF FPR), and in EXFI the WGS reads are hashed twice: once for filtering and a second time for the remainder of the computation. Filtering in this setting produced a 68%–75% reduction of the BF FPR while slightly improving all classification metrics.

## Related tools

- **biobloommaker** (Builds the transcriptome Bloom filter (.bf + .txt) with k-mer length 25 bp and specified bloom size/levels) — https://github.com/bcgsc/biobloom
- **biobloomcategorizer** (Classifies WGS reads against the transcriptome Bloom filter, discarding reads that share no k-mer with the transcriptome) — https://github.com/bcgsc/biobloom
- **EXFI** (Encapsulates this filtering as the build_baited_bloom_filter module, the initial stage of its splice-graph pipeline) — https://github.com/jlanga/exfi
- **Docker (exfi-docker)** (Containerized execution of build_baited_bloom_filter with all dependencies) — https://github.com/jlanga/exfi-docker

## Examples

```
docker run --rm exfi:1.5.6 build_baited_bloom_filter --help
```

## Evaluation signals

- The filtering stage should remove a large fraction of the WGS data while retaining reads overlapping the transcriptome
- The reported 68%–75% reduction of the BF FPR when filtering is applied is consistent with correct baiting and filtering
- biobloommaker produces both a .bf binary filter and a .txt INI information file, and the .txt must be kept alongside the .bf for categorization
- Output FASTQ contains only reads sharing at least one k-mer with the transcriptome; readStatus.tsv per-read classifications match the retained/discarded split
- Bloom filter classification should yield no false negatives (reads truly from the transcriptome must not be discarded); false positives are tunable via BF FPR

## Limitations

- Bloom filters return no false negatives but can produce false positives at a tunable BF FPR; without filtering, the FPR is substantially higher
- Biobloomcategorizer does not provide mapping locations, so filtered reads cannot be positioned without a later alignment-like step
- Input transcriptome quality matters: a single-tissue transcriptome leaves lowly expressed transcripts fragmented and other tissues' transcripts missing, biasing the filter bait set
- Bloom filter versions across bbt minor-version increments are incompatible and yield undefined/meaningless behavior

## Evidence

- [methods] build_baited_bloom_filter uses both the transcriptome assembly and the WGS reads: "build_baited_bloom_filter uses both the transcriptome assembly and the WGS reads and performs the task in three steps"
- [methods] reads not sharing a k-mer are discarded: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [results] filtering reduces BF FPR by 68–75%: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] Bloom filters have tunable FPR, no false negatives: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [readme] BBT creates filters and categorizes sequences: [evidence span withheld — anchor status: not_found]

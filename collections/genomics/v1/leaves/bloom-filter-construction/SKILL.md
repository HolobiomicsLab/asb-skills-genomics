---
name: bloom-filter-construction
description: Use when you have a de novo transcriptome assembly (FASTA) and raw WGS
  reads (FASTQ, gzipped allowed) and need a memory-efficient, alignment-free filter
  that keeps only reads sharing k-mers with the transcriptome.
when_to_use_negative:
- When mapping locations are required — BioBloomTools categorizes sequences faster
  than alignment but does not provide mapping positions.
- When the transcriptome is of poor quality or assembled from a single tissue (lowly
  expressed transcripts fragmented, other tissues' transcripts missing), since filtering
  quality depends on the bait set.
- When the goal is full genome assembly rather than restricting analysis to transcriptome-overlapping
  reads.
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: biobloommaker
  role: Builds the Bloom filter of the transcriptome (k-mer length 25 bp, specified
    bloom size and levels) into a .bf file plus .txt info file
  repo: https://github.com/bcgsc/biobloom
- name: biobloomcategorizer
  role: Categorizes WGS reads against the transcriptome Bloom filter and discards
    reads sharing no k-mer with the transcriptome
  repo: https://github.com/bcgsc/biobloom
- name: EXFI (build_baited_bloom_filter)
  role: Python pipeline module that orchestrates filter construction from transcriptome
    and WGS reads in three steps
  repo: https://github.com/jlanga/exfi
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:cd8c8a93ed6781c9cf7e41600681878714764653a237cd26c58fb3c4e25dd05b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/biobloom
  tools:
  - EXFI
  - EXFI (build_baited_bloom_filter)
  - Python
  - biobloomcategorizer
  - biobloommaker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bloom-filter-construction

## Summary

Constructs a baited, transcriptome-seeded Bloom filter with biobloommaker (k-mer length 25 bp, specified bloom size, e.g. 100M, and levels, e.g. 1) and uses biobloomcategorizer to retain only WGS reads sharing at least one k-mer with the assembled transcriptome, discarding all others. In EXFI this is the memory-efficient first stage that reduces the Bloom-filter false-positive rate by 68%–75% before splice-graph construction.

## When to use

Use when you have a de novo transcriptome assembly (FASTA) and raw WGS reads (FASTQ, gzipped allowed) and need a memory-efficient, alignment-free filter that keeps only reads sharing k-mers with the transcriptome.

## When NOT to use

- When mapping locations are required — BioBloomTools categorizes sequences faster than alignment but does not provide mapping positions.
- When the transcriptome is of poor quality or assembled from a single tissue (lowly expressed transcripts fragmented, other tissues' transcripts missing), since filtering quality depends on the bait set.
- When the goal is full genome assembly rather than restricting analysis to transcriptome-overlapping reads.

## Inputs

- Assembled transcriptome in FASTA format (e.g. de novo transcriptome assembly)
- Raw whole-genome sequencing (WGS) reads in FASTQ format (gzipped allowed)
- Bloom filter parameters: k-mer length (25 bp in EXFI), bloom size (e.g. 100M), levels (e.g. 1)

## Outputs

- Bloom filter file (.bf) with corresponding INI-format info file (.txt) from biobloommaker
- Filtered FASTQ file retaining exonic and flanking reads that share at least one k-mer with the transcriptome
- biobloomcategorizer summary.tsv / read status outputs

## How to apply

Load the assembled transcriptome in FASTA format and the raw WGS reads in FASTQ format (gzipped allowed). Build a Bloom filter of the transcriptome with biobloommaker using a k-mer length of 25 bp and the specified bloom size (e.g. 100M) and levels (e.g. 1); biobloommaker writes a binary .bf file plus an INI-format .txt info file that must stay together. Categorize the WGS reads with biobloomcategorizer, discarding every read that does not share at least one k-mer with the transcriptome, and write the retained (exonic and flanking) reads to the output FASTQ. Rationale: Bloom filters return no false negatives but can produce false positives with a tunable BF FPR, so pre-filtering the reads is what lowers the BF FPR by 68%–75% and slightly improves downstream classification metrics; EXFI hashes the WGS reads twice — once for this filtering and once for the remainder of the computation.

## Related tools

- **biobloommaker** (Builds the Bloom filter of the transcriptome (k-mer length 25 bp, specified bloom size and levels) into a .bf file plus .txt info file) — https://github.com/bcgsc/biobloom
- **biobloomcategorizer** (Categorizes WGS reads against the transcriptome Bloom filter and discards reads sharing no k-mer with the transcriptome) — https://github.com/bcgsc/biobloom
- **EXFI (build_baited_bloom_filter)** (Python pipeline module that orchestrates filter construction from transcriptome and WGS reads in three steps) — https://github.com/jlanga/exfi

## Examples

```
./biobloommaker -p transcriptome_bf transcriptome.fasta
```

## Evaluation signals

- The filtering stage should remove a large fraction of the WGS data while retaining reads overlapping the transcriptome.
- The BF FPR after filtering should drop substantially — the article reports a 68%–75% reduction when read filtering is applied.
- A .bf file and its paired .txt info file (same prefix) are both produced and kept together, as required by biobloomcategorizer.
- Every retained read shares at least one 25-bp k-mer with the transcriptome; reads without any shared k-mer are absent from the output FASTQ.
- All classification metrics downstream are expected to improve (or at least not degrade) relative to unfiltered reads, per the article's results.

## Limitations

- Bloom filters produce no false negatives but can produce false positives with a tunable BF FPR (biobloommaker default is 0.075), so retained reads include some non-exonic flanking sequence.
- Bloom filters in biobloomtool versions where the second number of the version is incremented are incompatible, resulting in undefined/meaningless behavior; filters must be used with a compatible categorizer version.
- Special care is required regarding input transcriptome quality: in the herring case the single-tissue transcriptome had fragmented lowly expressed transcripts and missing tissue-specific ones, and mappability scores fell for all methods.

## Evidence

- [methods] Each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [results] FPR reduction from filtering: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] Bloom filter false-positive behavior: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [readme] biobloommaker output files: [evidence span withheld — anchor status: not_found]
- [readme] No mapping locations from BBT: [evidence span withheld — anchor status: not_found]

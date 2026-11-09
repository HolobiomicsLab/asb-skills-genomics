---
name: kmer-based-sequence-matching
description: Use when reads must be quickly matched or filtered against a reference
  without producing alignments — e.g. removing WGS reads that share no 25-mer with
  a de novo transcriptome assembly, or pre-screening reads for contamination.
when_to_use_negative:
- You need mapping coordinates or alignment positions — biobloomcategorizer only classifies
  and does not provide mapping locations
- You need zero false positives in the retained set — Bloom filters have a tunable
  but non-zero false-positive rate (no false negatives, but false positives occur)
- You need full-exon or transcript-level precision from the matching alone — in EXFI
  the filtering is only the first stage before exon calling and splice-graph construction
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: biobloommaker
  role: Builds the Bloom filter (.bf) from the reference FASTA with tunable k-mer
    length, bloom size and false-positive rate
  repo: github.com/bcgsc/biobloom
- name: biobloomcategorizer
  role: Classifies reads against the Bloom filter and discards reads that share no
    k-mer with the transcriptome
  repo: github.com/bcgsc/biobloom
- name: EXFI
  role: Pipeline that orchestrates the baited Bloom filter read-filtering stage before
    exon and splice graph prediction
  repo: https://github.com/jlanga/exfi
- name: ChopStitch
  role: Prior approach cited as using Bloom filters to store frequent k-mers of a
    shotgun WGS dataset
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:b36c3bb69ee9c807d1f2b2242b1e624ec51d8b0e5d38edff6631937ff48f9d5e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ChopStitch
  - EXFI
  - Python
  - biobloomcategorizer
  - biobloommaker
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# kmer-based-sequence-matching

## Summary

Filter or classify sequences by hashing k-mers into a Bloom filter and testing membership, as EXFI does by baiting a Bloom filter from the assembled transcriptome and using biobloomcategorizer to discard WGS reads that share no k-mer with it. This avoids costly alignment and shrinks the false-positive rate (BF FPR) of the downstream analysis.

## When to use

Use when reads must be quickly matched or filtered against a reference without producing alignments — e.g. removing WGS reads that share no 25-mer with a de novo transcriptome assembly, or pre-screening reads for contamination.

## When NOT to use

- You need mapping coordinates or alignment positions — biobloomcategorizer only classifies and does not provide mapping locations
- You need zero false positives in the retained set — Bloom filters have a tunable but non-zero false-positive rate (no false negatives, but false positives occur)
- You need full-exon or transcript-level precision from the matching alone — in EXFI the filtering is only the first stage before exon calling and splice-graph construction

## Inputs

- Reference FASTA file (e.g. assembled transcriptome)
- Raw reads in FASTQ (gzipped allowed) or FASTA (e.g. WGS reads); BAM/bz2 inputs also accepted by biobloomcategorizer
- Bloom filter binary (.bf) plus its INI-format information file (.txt) for categorization

## Outputs

- Bloom filter file (.bf) and information file (.txt) from biobloommaker
- summary.tsv and readStatus.tsv classification summaries
- Categorized FASTA/FASTQ files containing retained (matching) reads, e.g. exonic and flanking reads for downstream splice-graph construction

## How to apply

Build a Bloom filter from the reference FASTA with biobloommaker, choosing the k-mer length (EXFI uses 25 bp), the bloom size (e.g. 100M) and level (e.g. 1); the tool computes the optimal filter size from the target false-positive rate (default FPR 0.075). Then run biobloomcategorizer with the .bf and .txt info files against the FASTQ/FASTA reads, keeping the .bf and .txt in the same directory; any read that does not share at least one k-mer with the reference is discarded and retained reads are written to categorized output FASTQ files. Rationale: Bloom filters return no false negatives but allow a tunable rate of false positives, so membership hits are conservative; in EXFI this baited-filter read pre-filtering reduced the BF FPR by 68%–75% while slightly improving classification metrics. Because the method gives no mapping locations, use it as a fast pipeline pre-processing or QC step rather than as an aligner substitute.

## Related tools

- **biobloommaker** (Builds the Bloom filter (.bf) from the reference FASTA with tunable k-mer length, bloom size and false-positive rate) — github.com/bcgsc/biobloom
- **biobloomcategorizer** (Classifies reads against the Bloom filter and discards reads that share no k-mer with the transcriptome) — github.com/bcgsc/biobloom
- **EXFI** (Pipeline that orchestrates the baited Bloom filter read-filtering stage before exon and splice graph prediction) — https://github.com/jlanga/exfi
- **ChopStitch** (Prior approach cited as using Bloom filters to store frequent k-mers of a shotgun WGS dataset)

## Examples

```
./biobloommaker -p transcriptome transcriptome.fasta
```

## Evaluation signals

- A large fraction of the WGS input is removed while reads overlapping the transcriptome are retained, consistent with the reported 68%–75% BF FPR reduction upon read filtering
- No false negatives: every read that truly shares a k-mer with the reference must be retained (Bloom filter property)
- The .bf file is accompanied by its .txt information file in the same directory; missing .txt makes categorization impossible
- Categorization runs produce summary.tsv and readStatus.tsv with per-read and per-filter classification counts
- Classification metrics (precision/recall of exon/IEB prediction downstream) should improve or hold when the filtered reads are fed forward instead of the full WGS set

## Limitations

- Bloom filters can produce false positives with a tunable BF FPR, so some non-target reads may pass the filter
- No mapping locations are reported — matching does not tell you where a read aligns
- Filter version compatibility matters: filters created across BioBloomTools minor-version increments (e.g. 2.0.x vs 2.1.x) are incompatible and yield undefined/meaningless behavior with no error before v2.3.0
- Results depend on input transcriptome quality; fragmented or single-tissue assemblies underrepresent lowly expressed transcripts

## Evidence

- [methods] k-mer discard rule: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [methods] baited filter construction: "build_baited_bloom_filter uses both the transcriptome assembly and the WGS reads and performs the task in three steps"
- [methods] BF properties: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] FPR reduction benefit: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [readme] BBT purpose: [evidence span withheld — anchor status: not_found]
- [readme] biobloommaker invocation: [evidence span withheld — anchor status: not_found]

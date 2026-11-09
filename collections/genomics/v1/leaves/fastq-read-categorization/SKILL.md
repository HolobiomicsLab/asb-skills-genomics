---
name: fastq-read-categorization
description: Use when raw WGS FASTQ reads must be reduced to only those overlapping
  an assembled transcriptome (exonic/flanking content) before expensive k-mer or graph
  computations.
when_to_use_negative:
- When mapping locations or alignment coordinates are required — BioBloom categorizes
  sequences but 'does not provide mapping locations'
- When the reference is not a transcriptome assembly but many targets (>100) — use
  the multi-index Bloom filter workflow instead
- When the input transcriptome is poor quality or single-tissue, since missing/fragmented
  transcripts will cause their reads to be discarded
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: biobloommaker
  role: Builds the transcriptome Bloom filter (.bf + .txt info file) with k=25 and
    user-specified bloom size/levels
  repo: https://github.com/bcgsc/biobloom
- name: biobloomcategorizer
  role: Categorizes WGS reads against the transcriptome Bloom filter, discarding reads
    sharing no k-mer with it
  repo: https://github.com/bcgsc/biobloom
- name: EXFI
  role: Pipeline whose build_baited_bloom_filter stage wraps this filtering; retained
    reads feed build_splice_graph
  repo: https://github.com/jlanga/exfi
- name: ABySS
  role: Consumes the retained reads downstream to build a cascading Bloom filter
  repo: https://github.com/bcgsc/abyss
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:3b364dca2f86fa55869318c8670c6663c4a17d0338aad09a9f52da0c19519038
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bcgsc/biobloom
  tools:
  - ABySS
  - EXFI
  - Python
  - biobloomcategorizer
  - biobloommaker
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fastq-read-categorization

## Summary

Categorize (filter) raw FASTQ WGS reads against a transcriptome-derived Bloom filter using BioBloom Tools (biobloommaker + biobloomcategorizer), discarding reads that share no k-mer with the transcriptome to reduce downstream false positives and computation.

## When to use

Use when raw WGS FASTQ reads must be reduced to only those overlapping an assembled transcriptome (exonic/flanking content) before expensive k-mer or graph computations.

## When NOT to use

- When mapping locations or alignment coordinates are required — BioBloom categorizes sequences but 'does not provide mapping locations'
- When the reference is not a transcriptome assembly but many targets (>100) — use the multi-index Bloom filter workflow instead
- When the input transcriptome is poor quality or single-tissue, since missing/fragmented transcripts will cause their reads to be discarded

## Inputs

- Assembled transcriptome in FASTA format
- Raw whole-genome sequencing reads in FASTQ format (gzipped allowed)
- Bloom filter parameters: k-mer length 25, bloom size (e.g. 100M), levels (e.g. 1)

## Outputs

- Transcriptome Bloom filter file (.bf) plus information file (.txt) from biobloommaker
- Filtered FASTQ containing only reads sharing at least one k-mer with the transcriptome
- Optional biobloomcategorizer summary.tsv / readStatus.tsv classification reports

## How to apply

First build a Bloom filter of the assembled transcriptome (FASTA) with biobloommaker using k-mer length 25 bp and a specified bloom size (e.g. 100M) and levels (e.g. 1). Then run biobloomcategorizer over the raw WGS reads (FASTQ, gzipped allowed): each read that does not share at least one k-mer with the transcriptome Bloom filter is discarded, and only retained (exonic and flanking) reads are written to the output FASTQ. The rationale is that Bloom filters return no false negatives but have a tunable false-positive rate (BF FPR), so no truly transcriptomic read is lost; filtering reads reduced the BF FPR by 68%–75% while slightly improving all classification metrics. In EXFI, reads are hashed twice — once for this filtering step and once for the remainder of the computation — because the filter removes most WGS data before build_splice_graph runs.

## Related tools

- **biobloommaker** (Builds the transcriptome Bloom filter (.bf + .txt info file) with k=25 and user-specified bloom size/levels) — https://github.com/bcgsc/biobloom
- **biobloomcategorizer** (Categorizes WGS reads against the transcriptome Bloom filter, discarding reads sharing no k-mer with it) — https://github.com/bcgsc/biobloom
- **EXFI** (Pipeline whose build_baited_bloom_filter stage wraps this filtering; retained reads feed build_splice_graph) — https://github.com/jlanga/exfi
- **ABySS** (Consumes the retained reads downstream to build a cascading Bloom filter) — https://github.com/bcgsc/abyss

## Examples

```
biobloommaker -p transcriptome -k 25 --bloom-size 100M --levels 1 transcriptome.fasta && biobloomcategorizer -p filtered -f "transcriptome.bf" wgs_reads.fastq.gz
```

## Evaluation signals

- The filtering stage should remove a large fraction of the WGS data while retaining reads overlapping the transcriptome
- BF FPR should drop by roughly 68%–75% relative to no filtering, with classification metrics slightly improved
- No false negatives expected: Bloom filters return no false negatives, so every genuinely transcriptomic k-mer read must be retained
- Every discarded read must share zero k-mers with the transcriptome filter (reads with ≥1 shared k-mer are kept)
- biobloommaker output must include both the .bf bit array and its matching .txt info file; the .txt must accompany the .bf during categorization

## Limitations

- Bloom filters produce false positives at a tunable BF FPR (biobloommaker default 0.075), so some non-exonic reads may be retained
- Bloom filter compatibility breaks across BioBloom minor versions (e.g. a 2.0.12 filter does not work with 2.1.X), with undefined behavior and no error message prior to 2.3.0
- Filtering efficacy depends on transcriptome quality: single-tissue assemblies fragment lowly expressed transcripts and miss other tissues' transcripts
- Categorization is faster than alignment but provides no mapping locations

## Evidence

- [methods] each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [results] Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR): "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [abstract] The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome: "The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome"
- [readme] BBT provides the means to create filters for a given reference and then to categorize sequences. This methodology is faster than alignment but does not provide mapping locations.: [evidence span withheld — anchor status: not_found]

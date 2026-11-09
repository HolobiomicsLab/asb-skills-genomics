---
name: bloom-filter-based-read-filtering
description: Use when running a low-memory Bloom-filter workflow (e.g. EXFI's build_baited_bloom_filter)
  where the BF FPR is high — e.g. FPR of ~32.6% with a 4 GB filter — and WGS reads
  mostly not from exons contaminate the filter.
when_to_use_negative:
- You need mapping locations or alignment coordinates, not just membership classification
  — biobloomcategorizer categorizes sequences but 'does not provide mapping locations'
- The downstream analysis needs all genomic reads (e.g. whole-genome assembly or variant
  calling outside exonic regions), since filtering discards reads sharing no k-mer
  with the transcriptome
- The input reads are already transcriptomic (RNA-Seq) rather than WGS reads — almost
  all reads would pass the filter and the step adds no benefit
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: EXFI
  role: Pipeline whose build_baited_bloom_filter step applies read filtering with
    biobloomcategorizer before cascading Bloom filter construction
  repo: https://github.com/jlanga/exfi
- name: biobloomtools
  role: Provides biobloomcategorizer to discard WGS reads sharing no k-mer with the
    transcriptome
  repo: https://github.com/bcgsc/biobloom
- name: ABySS
  role: Builds the cascading Bloom filter from the remaining (filtered) reads via
    abyss-bloom
  repo: https://github.com/bcgsc/abyss/
- name: wgsim
  role: Simulates the zebrafish/human WGS reads used to test the filtering step
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:883355aa1fe223de1f6d3fd3af4f3310dd32c195918d73a3703ed9ec6b927e16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - ABySS
  - BioPython
  - EXFI
  - Pandas
  - Python
  - bedtools
  - biobloomtools
  - wgsim
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bloom-filter-based-read-filtering

## Summary

Pre-filters whole-genome sequencing (WGS) reads against a transcriptome using Bloom filters (biobloomcategorizer), discarding reads that share no k-mer with the transcriptome before Bloom-filter construction. This lowers the Bloom-filter false-positive rate (BF FPR) by 68%–75% and slightly improves exon-prediction classification metrics in EXFI.

## When to use

Use when running a low-memory Bloom-filter workflow (e.g. EXFI's build_baited_bloom_filter) where the BF FPR is high — e.g. FPR of ~32.6% with a 4 GB filter — and WGS reads mostly not from exons contaminate the filter.

## When NOT to use

- You need mapping locations or alignment coordinates, not just membership classification — biobloomcategorizer categorizes sequences but 'does not provide mapping locations'
- The downstream analysis needs all genomic reads (e.g. whole-genome assembly or variant calling outside exonic regions), since filtering discards reads sharing no k-mer with the transcriptome
- The input reads are already transcriptomic (RNA-Seq) rather than WGS reads — almost all reads would pass the filter and the step adds no benefit

## Inputs

- Raw WGS reads in FASTA/FASTQ (paired-end supported, e.g. wgsim-simulated reads)
- Assembled transcriptome FASTA (de novo or reference transcriptome)
- Bloom filter (.bf) plus its information (.txt) file built by biobloommaker from the transcriptome

## Outputs

- Filtered WGS reads sharing ≥1 k-mer with the transcriptome (categorized FASTA/FASTQ files)
- summary.tsv / readStatus.tsv from biobloomcategorizer
- Downstream: reduced-FPR baited/cascading Bloom filter, exon predictions, and splice graph in GFA1 format

## How to apply

Build a Bloom filter from the transcriptome assembly, then run biobloomcategorizer over the WGS reads so that each read that does not share at least one k-mer with the transcriptome is discarded. Feed only the retained reads into the cascading Bloom filter construction with ABySS (k = 25 bp in the EXFI benchmark), then proceed to build_splice_graph. The rationale is that Bloom filters return no false negatives but produce false positives with a tunable BF FPR, so removing transcriptome-unrelated reads (most of the genome — only 5.68% of zebrafish bases are exons) sharply reduces the BF FPR. In the zebrafish study, filtering cut the BF FPR from 32.6% to 8.1% at 4 GB and reduced processing time, while improving precision, recall, and F1 (F1: 89.8% → 94.8%, vs. a 95.6% maximum at high memory).

## Related tools

- **EXFI** (Pipeline whose build_baited_bloom_filter step applies read filtering with biobloomcategorizer before cascading Bloom filter construction) — https://github.com/jlanga/exfi
- **biobloomtools** (Provides biobloomcategorizer to discard WGS reads sharing no k-mer with the transcriptome) — https://github.com/bcgsc/biobloom
- **ABySS** (Builds the cascading Bloom filter from the remaining (filtered) reads via abyss-bloom) — https://github.com/bcgsc/abyss/
- **wgsim** (Simulates the zebrafish/human WGS reads used to test the filtering step)

## Examples

```
docker run --rm exfi:1.5.6 build_baited_bloom_filter --help  # filtering via biobloomcategorizer inside EXFI; e.g. biobloomcategorizer -e -p out -f transcriptome.bf reads_1.fq reads_2.fq
```

## Evaluation signals

- BF FPR of the filtered cascading Bloom filter falls by roughly 68%–75% relative to the unfiltered run (e.g. 32.6% → 8.1% for zebrafish at 4 GB)
- Downstream exon-prediction classification metrics do not decrease: precision, recall, and F1 should equal or exceed the unfiltered run (zebrafish F1 rose from 89.8% to 94.8%)
- Processing time of the filtered run is reduced compared to the unfiltered run
- biobloomcategorizer produces summary.tsv and categorized FASTA/FASTQ outputs; the .bf file and its .txt info file must be kept together in the same directory
- Filter compatibility check: biobloom filters are only compatible across versions where the second version number is unchanged (e.g. 2.0.12 works with 2.0.13, not 2.1.X)

## Limitations

- At low memory (4 GB) the filtered BF FPR is still 8.1%; increasing filter memory to 60 GB lowers it to 0.4%, and the filtering step's benefit is largest in memory-constrained settings
- Bloom filters guarantee no false negatives but can produce false positives with a tunable BF FPR, so some transcriptome-absent k-mers will still pass through
- Filter versions with an incremented second version number (e.g. 2.0.x vs 2.1.x) are incompatible and 'will result in undefined/meaningless behavior', with no pre-2.3.0 error message

## Evidence

- [methods] each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [results] Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [methods] It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR): "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [abstract] The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome: "The main algorithm uses Bloom filters to remove reads that are not part of the transcriptome"
- [readme] BioBloom Tools (BBT) provides the means to create filters for a given reference and then to categorize sequences. This methodology is faster than alignment but does not provide mapping locations.: [evidence span withheld — anchor status: not_found]

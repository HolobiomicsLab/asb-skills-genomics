---
name: insplico-bam-based-measurement-extraction
description: Use when you have an RNA-seq alignment (BAM) plus a table of exons of
  interest against hg38, and need AISO counts (upfi/dofi/bus/bos) and Fupfi around
  those exons with short or long reads.
when_to_use_negative:
- When the goal is genome-wide transcript quantification or differential expression
  rather than splicing order of adjacent introns around specified exons
- When exons of interest are longer than the short-read length / paired-end fragment
  size allows, since read length limits the maximum exon length quantifiable with
  short reads
- When you need splicing-order information beyond the introns directly flanking the
  exon of interest; short reads provide almost no information beyond the flanking
  introns
edam_operation: http://edamontology.org/operation_0264
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0085
tools:
- name: Insplico
  role: Standalone software that quantifies AISO from short- and long-read alignments
    and outputs upfi/dofi/bus/bos counts, Fupfi, PSI and PIR; includes the optional
    5'/3' bias-correction module
  repo: gitlab.com/aghr/insplico
- name: R
  role: Used for downstream setup, table handling and statistical testing of the Insplico-derived
    count outputs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:8e3d21d51bb20f8e5656b6b35bc3c4e3e4b9b472bfeb0a0dda8d03c8b7fa9fea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  edam_operation: http://edamontology.org/operation_0264
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# insplico-bam-based-measurement-extraction

## Summary

Use Insplico to quantify adjacent-intron splicing order (AISO) around user-specified exons by extracting upfi, dofi, bus and bos counts, Fupfi, PSI and PIR from short- or long-read RNA-seq alignments (BAM). This is the core measurement step that feeds downstream splicing-order comparisons and bias correction.

## When to use

Use when you have an RNA-seq alignment (BAM) plus a table of exons of interest against hg38, and need AISO counts (upfi/dofi/bus/bos) and Fupfi around those exons with short or long reads.

## When NOT to use

- When the goal is genome-wide transcript quantification or differential expression rather than splicing order of adjacent introns around specified exons
- When exons of interest are longer than the short-read length / paired-end fragment size allows, since read length limits the maximum exon length quantifiable with short reads
- When you need splicing-order information beyond the introns directly flanking the exon of interest; short reads provide almost no information beyond the flanking introns

## Inputs

- BAM alignment of RNA-seq reads (short-read or long-read/direct-RNA, e.g. ONT) against hg38
- Exon-of-interest input table (user-specified exons)
- Genome reference (hg38) and exon/intron annotation derived from GTF (e.g. Hsa38_only_transcript_support_level_1.gtf)

## Outputs

- upfi, dofi, bus and bos counts per exon of interest
- Fupfi (fraction of transcripts with upstream-first intron spliced) measurements per exon
- PSI and PIR estimates around the exons of interest

## How to apply

Run Insplico with a BAM alignment (e.g. Drexler ONT direct-RNA K562 or short-read data) loaded together with the exon-of-interest input table against the hg38 genome reference, using the same exon table across runs to keep read type comparisons consistent. Insplico reports the key pre-mRNA processing measurements around user-specified exons: upfi, dofi, bus and bos counts, plus derived fractions (Fupfi), PSI and PIR. For long-read direct-RNA data, run once with the 5' truncation correction and once with the 3' truncation correction, then a third run with both corrections applied jointly, since uncorrected long-read data carry 5' and 3' biases that distort Fupfi. Compare the resulting Fupfi distributions across runs (uncorrected long-read, corrected long-read, short-read) to confirm the measurements are consistent, and summarize splicing order as upfi/dofi ratios testable with a two-sided Fisher's Exact test.

## Related tools

- **Insplico** (Standalone software that quantifies AISO from short- and long-read alignments and outputs upfi/dofi/bus/bos counts, Fupfi, PSI and PIR; includes the optional 5'/3' bias-correction module) — gitlab.com/aghr/insplico
- **R** (Used for downstream setup, table handling and statistical testing of the Insplico-derived count outputs)

## Examples

```
insplico --bam drexler_ont_k562.bam --exo exons_of_interest.tab --gtf hg38_ref.gtf --biascorr
```

## Evaluation signals

- Output table contains the four expected count columns (upfi, dofi, bus, bos) plus Fupfi per exon of interest
- For long-read --biascorr runs: corrected upfi/dofi ratio should approach the short-read reference (reported as 1.25 vs 1.41 with two-sided Fisher's Exact P = 0.48, i.e. not significantly different)
- Bias-corrected Fupfi distribution should match the short-read Fupfi distribution, unlike the uncorrected long-read distribution
- The same exon-of-interest input table must be used across short-read and long-read runs so Fupfi distributions are directly comparable
- Each of the three long-read correction runs (5'-only, 3'-only, both) must complete and yield count outputs for comparison

## Limitations

- Short reads: read length and paired-end fragment size limit the maximum exon length quantifiable, and provide almost no splicing-order information beyond the directly flanking introns
- Long-read sequencing and library preparation carry 5' and 3' biases that must be corrected (via --biascorr) to harmonize long-read patterns with short-read results
- AISO around individual exons is highly constant across cell and tissue types, so large tissue-condition differences in Fupfi may indicate measurement or bias issues rather than biology

## Evidence

- [discussion] It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [other] Load the Drexler ONT direct-RNA BAM alignment into Insplico together with the same exon-of-interest input table used for the short-read runs, against the hg38 genome reference.: [evidence span withheld — anchor status: not_found]
- [discussion] Long read sequencing and library preparation carry 5' and 3' biases: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [abstract] the first standalone software for quantifying AISO that works with both short and long read sequencing technologies: "we present Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"

---
name: fupfi-distribution-cross-platform-comparison
description: Use when long-read (e.g. ONT direct-RNA) Fupfi/AISO distributions disagree
  with short-read measurements around the same exons and you need to test whether
  truncation bias explains the discrepancy.
when_to_use_negative:
- The comparison is entirely within short-read data (read length/fragment size already
  bound the quantifiable exons; no long-read bias correction applies)
- Splicing-order information beyond the introns directly flanking the exon of interest
  is required — short reads barely provide such information, so the cross-platform
  comparison would be uninformative there
- Exons of interest are longer than what short-read read length/paired-end fragment
  size can quantify, making the short-read reference distribution itself unreliable
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3320
tools:
- name: Insplico
  role: Quantifies AISO (upfi, dofi, bus, bos counts, Fupfi) from both short and long
    reads; its optional --biascorr module corrects 5' and 3' truncation biases in
    long-read data
  repo: gitlab.com/aghr/insplico
- name: R
  role: Session setup and statistical testing (e.g. Fisher's Exact test) of corrected
    vs short-read upfi/dofi ratios
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:672f1d980a55f5295dbd74a2aa8801ec2dc999fb419a53005623003357f6bc63
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# fupfi-distribution-cross-platform-comparison

## Summary

Compares the fraction of upstream-first (Fupfi) splicing-order distributions measured from long-read RNA-seq against short-read RNA-seq, applying Insplico's --biascorr 5' and 3' truncation corrections to harmonize long-read patterns with the short-read reference.

## When to use

Use when long-read (e.g. ONT direct-RNA) Fupfi/AISO distributions disagree with short-read measurements around the same exons and you need to test whether truncation bias explains the discrepancy.

## When NOT to use

- The comparison is entirely within short-read data (read length/fragment size already bound the quantifiable exons; no long-read bias correction applies)
- Splicing-order information beyond the introns directly flanking the exon of interest is required — short reads barely provide such information, so the cross-platform comparison would be uninformative there
- Exons of interest are longer than what short-read read length/paired-end fragment size can quantify, making the short-read reference distribution itself unreliable

## Inputs

- Long-read BAM alignment (e.g. ONT direct-RNA, Drexler et al. K562)
- Exon-of-interest input table (same one used for the short-read Insplico run)
- hg38 genome reference
- Short-read Insplico output with upfi/dofi counts and Fupfi as the reference distribution

## Outputs

- Bias-corrected upfi, dofi, bus, bos counts and Fupfi values per exon (for 5', 3', and joint corrections)
- Fupfi distribution comparison plots (corrected long-read vs uncorrected long-read vs short-read)
- Corrected upfi/dofi ratio with two-sided Fisher's Exact test P-value against the short-read ratio

## How to apply

Load the long-read alignment (e.g. the Drexler et al. ONT direct-RNA BAM for K562) into Insplico together with the same exon-of-interest input table used for the short-read run, against the hg38 genome reference. Run Insplico three times: once with the 5' truncation-bias correction, once with the 3' correction, and once with both corrections applied jointly; each run yields upfi, dofi, bus, bos counts and Fupfi. Plot/overlay the corrected Fupfi distributions against the uncorrected long-read and short-read Fupfi distributions to identify which correction harmonizes the long-read pattern with short reads. Finally, compute the corrected upfi/dofi ratio and test agreement with the short-read ratio using a two-sided Fisher's Exact test (e.g. in R). The article's benchmark: corrected upfi/dofi ratio of 1.25 versus 1.41 for short reads was not significantly different (P = 0.48).

## Related tools

- **Insplico** (Quantifies AISO (upfi, dofi, bus, bos counts, Fupfi) from both short and long reads; its optional --biascorr module corrects 5' and 3' truncation biases in long-read data) — gitlab.com/aghr/insplico
- **R** (Session setup and statistical testing (e.g. Fisher's Exact test) of corrected vs short-read upfi/dofi ratios)

## Examples

```
insplico --bam drexler_k562_ont.bam --exons exons_of_interest.tab --ref hg38 --biascorr
```

## Evaluation signals

- Bias-corrected upfi/dofi ratio ≈ 1.25 vs 1.41 for short reads, with two-sided Fisher's Exact P = 0.48 (no significant difference)
- Corrected Fupfi distribution visually matches the short-read Fupfi distribution, unlike the uncorrected long-read distribution
- Three correction runs (5'-only, 3'-only, joint 5'+3') are compared and the joint correction is the one that harmonizes with short reads
- Same exon-of-interest input table and hg38 reference are used for both platforms so the comparison is not confounded

## Limitations

- Long-read sequencing and library preparation carry 5' and 3' biases that must be explicitly corrected; without --biascorr the long-read AISO pattern is discordant with short reads
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, limiting how far the cross-platform comparison can extend
- Maximum quantifiable exon length with short reads is limited by read length and paired-end fragment size, which constrains the short-read reference distribution

## Evidence

- [discussion] Long read sequencing and library preparation carry 5' and 3' biases that must be corrected to harmonize with short-read results: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [discussion] The optional module to perform these corrections is a unique feature of Insplico: "The optional module to perform these corrections is a unique feature of Insplico"
- [discussion] Measurement of pre-mRNA processing metrics around user-specified exons: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [abstract] Insplico works with both short and long read sequencing technologies: "Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [readme] R session setup is documented in the repository README: "if (!require("BiocManager", quietly = TRUE))
    install.packages("BiocManager")"

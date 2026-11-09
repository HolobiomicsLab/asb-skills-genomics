---
name: long-read-splicing-order-quantification
description: Use when quantifying splicing order (AISO) around user-specified exons
  from long-read (e.g. ONT direct-RNA) alignments, and when long-read upfi/dofi or
  Fupfi distributions must be harmonized with short-read reference patterns.
when_to_use_negative:
- The input is short-read RNA-seq data — the --biascorr module addresses 5'/3' truncation
  biases specific to long-read sequencing and library preparation
- The goal is splicing-order information beyond the introns directly flanking the
  input exon with short reads — short reads can barely provide this and Insplico's
  read/fragment length limits the maximum quantifiable exon length
- No exon-of-interest annotation is available — Insplico requires a user-specified
  exon table to measure pre-mRNA processing metrics
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Insplico
  role: Standalone software that quantifies AISO from long-read data and provides
    the unique --biascorr module to correct 5' and 3' long-read truncation biases
  repo: gitlab.com/aghr/insplico
- name: R
  role: Session setup, GTF import (rtracklayer/plyranges) and statistical testing
    (two-sided Fisher's Exact test) of upfi/dofi ratios
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:e630331536037e825eb5f866ec6f0a40c2e8af3e2276d09828d56732e09a6d7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# long-read-splicing-order-quantification

## Summary

Quantifies adjacent intron splicing order (AISO) from long-read RNA-seq data with Insplico, applying its --biascorr module to correct the 5' and 3' truncation biases that otherwise distort upfi/dofi measurements relative to short-read results.

## When to use

Use when quantifying splicing order (AISO) around user-specified exons from long-read (e.g. ONT direct-RNA) alignments, and when long-read upfi/dofi or Fupfi distributions must be harmonized with short-read reference patterns.

## When NOT to use

- The input is short-read RNA-seq data — the --biascorr module addresses 5'/3' truncation biases specific to long-read sequencing and library preparation
- The goal is splicing-order information beyond the introns directly flanking the input exon with short reads — short reads can barely provide this and Insplico's read/fragment length limits the maximum quantifiable exon length
- No exon-of-interest annotation is available — Insplico requires a user-specified exon table to measure pre-mRNA processing metrics

## Inputs

- Long-read RNA-seq BAM alignment (e.g. ONT direct-RNA K562 data, Drexler et al.)
- Exon-of-interest input table (same as used for short-read Insplico runs)
- hg38 genome reference
- GTF annotation with transcript support level 1 transcripts (e.g. Hsa38_only_transcript_support_level_1.gtf)

## Outputs

- Per-exon upfi, dofi, bus and bos splicing-order counts
- Fupfi (fraction of transcripts with upstream intron spliced first) measurements
- Bias-corrected Fupfi distribution for 5'-only, 3'-only, and joint 5'+3' corrections
- upfi/dofi ratio with two-sided Fisher's Exact test P-value vs the short-read reference

## How to apply

Load the long-read BAM alignment (e.g. Drexler et al. ONT direct-RNA K562) into Insplico together with the same exon-of-interest input table used for the short-read runs, against the hg38 genome reference. Run Insplico to obtain upfi, dofi, bus and bos counts and Fupfi measurements, first with the 5' truncation correction applied separately, then with the 3' truncation correction applied separately, and finally with both corrections applied jointly (--biascorr). The rationale is that long-read sequencing and library preparation carry 5' and 3' biases that, when corrected, harmonize long-read patterns with short reads. Compare the bias-corrected Fupfi distributions against both the uncorrected long-read and the short-read Fupfi distributions. Compute the corrected upfi/dofi ratio and test agreement with the short-read ratio using a two-sided Fisher's Exact test in R.

## Related tools

- **Insplico** (Standalone software that quantifies AISO from long-read data and provides the unique --biascorr module to correct 5' and 3' long-read truncation biases) — gitlab.com/aghr/insplico
- **R** (Session setup, GTF import (rtracklayer/plyranges) and statistical testing (two-sided Fisher's Exact test) of upfi/dofi ratios)

## Examples

```
R: my_obj <- import(gtf_file)  # then run Insplico on the ONT BAM with --biascorr for 5'-only, 3'-only, and joint 5'+3' corrections
```

## Evaluation signals

- With --biascorr applied to both 5' and 3' truncation biases jointly, the corrected upfi/dofi ratio is 1.25 versus 1.41 for short reads (Drexler ONT K562 data)
- A two-sided Fisher's Exact test comparing corrected long-read vs short-read ratios gives P = 0.48 (not significantly different)
- The bias-corrected Fupfi distribution matches the distribution obtained with short reads, while the uncorrected long-read distribution does not
- Output contains the full count schema (upfi, dofi, bus, bos) and Fupfi per input exon-of-interest
- Corrected ratios from 5'-only and 3'-only correction runs can be compared against the joint-correction run to confirm which truncation bias dominates

## Limitations

- Long-read sequencing and library preparation carry 5' and 3' biases that must be corrected to harmonize with short-read results; uncorrected long-read AISO patterns are misleading
- Maximum quantifiable exon length with short reads is limited by read length and paired-end fragment size, so short- and long-read runs are not interchangeable for long exons
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads

## Evidence

- [discussion] The optional module to perform these corrections is a unique feature of Insplico: "The optional module to perform these corrections is a unique feature of Insplico"
- [discussion] 5'/3' biases must be corrected: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [discussion] Insplico outputs upfi, dofi, bus, bos and Fupfi: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [abstract] Insplico works with short and long reads: "the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [methods] R GTF import used in the pipeline: "my_obj <- import(gtf_file)"

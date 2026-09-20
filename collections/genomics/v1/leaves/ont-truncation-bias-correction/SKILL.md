---
name: ont-truncation-bias-correction
description: Use when ONT long-read RNA-seq AISO measurements (Fupfi distributions,
  upfi/dofi ratios) disagree with short-read results and 5'/3' truncation biases from
  long-read sequencing or library preparation are suspected.
when_to_use_negative:
- Data are short reads — truncation bias correction is specific to long-read sequencing
  and library preparation artifacts and is unnecessary for short-read AISO runs
- The goal is splicing-order information beyond the introns directly flanking the
  exon of interest, which only long reads can provide and for which bias correction
  is not the limiting factor
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: Insplico
  role: Standalone software that quantifies AISO from long reads and provides the
    optional --biascorr module for 5' and 3' truncation bias correction
  repo: gitlab.com/aghr/insplico
- name: R
  role: Used for statistical comparison of corrected vs. short-read upfi/dofi ratios
    (Fisher's Exact test) and downstream plotting
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:72f5c072df87eab15a458b536ae9ac107b4bdbc4412382fa2e93d94d07421d44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# ont-truncation-bias-correction

## Summary

Applies Insplico's optional --biascorr module to correct 5' and 3' truncation biases in ONT direct-RNA long reads, so that long-read adjacent-intron splicing-order (AISO) patterns (upfi/dofi ratios) harmonize with the short-read reference distributions.

## When to use

Use when ONT long-read RNA-seq AISO measurements (Fupfi distributions, upfi/dofi ratios) disagree with short-read results and 5'/3' truncation biases from long-read sequencing or library preparation are suspected.

## When NOT to use

- Data are short reads — truncation bias correction is specific to long-read sequencing and library preparation artifacts and is unnecessary for short-read AISO runs
- The goal is splicing-order information beyond the introns directly flanking the exon of interest, which only long reads can provide and for which bias correction is not the limiting factor

## Inputs

- ONT direct-RNA BAM alignment (e.g. Drexler et al. K562 data)
- Exon-of-interest input table (same as used for short-read Insplico runs)
- hg38 genome reference

## Outputs

- Bias-corrected upfi, dofi, bus, bos counts
- Corrected Fupfi distributions under 5'-only, 3'-only, and joint 5'+3' correction
- Corrected upfi/dofi ratio with two-sided Fisher's Exact test P-value versus the short-read ratio

## How to apply

Load the Drexler et al. ONT direct-RNA BAM alignment into Insplico together with the same exon-of-interest input table used for the short-read runs, against the hg38 genome reference. Run Insplico with --biascorr to obtain upfi, dofi, bus, bos counts and Fupfi measurements, first applying the 5' truncation correction alone, then the 3' truncation correction alone, and finally both corrections jointly to see which combination harmonizes with short reads. Compare the bias-corrected Fupfi distributions against both the uncorrected long-read and the short-read distributions. Compute the corrected upfi/dofi ratio and test agreement with the short-read ratio using a two-sided Fisher's Exact test. Rationale: 5' and 3' biases in long-read sequencing and library preparation distort apparent splicing order, and correcting them brings long-read patterns into statistical agreement with short reads.

## Related tools

- **Insplico** (Standalone software that quantifies AISO from long reads and provides the optional --biascorr module for 5' and 3' truncation bias correction) — gitlab.com/aghr/insplico
- **R** (Used for statistical comparison of corrected vs. short-read upfi/dofi ratios (Fisher's Exact test) and downstream plotting)

## Examples

```
insplico --bam drexler_k562_ont.bam --exons exons_of_interest.tsv --genome hg38 --biascorr
```

## Evaluation signals

- Bias-corrected upfi/dofi ratio equals 1.25 versus the short-read ratio of 1.41
- Two-sided Fisher's Exact test comparing corrected long-read vs. short-read ratios yields P = 0.48 (no significant difference)
- Corrected Fupfi distribution matches the short-read Fupfi distribution, unlike the uncorrected long-read distribution
- All three correction modes (5'-only, 3'-only, joint) are run and compared to identify which harmonizes with short reads

## Limitations

- Uncorrected long-read AISO patterns carry 5' and 3' biases from sequencing and library preparation; skipping --biascorr leaves these artifacts in the results
- Short reads can barely provide splicing-order information beyond the introns directly flanking the exon of interest, so the short-read reference itself is limited to the flanking introns
- The maximum length of exons quantifiable by Insplico with short reads is limited by read length and paired-end fragment size

## Evidence

- [discussion] Correction of 5' and 3' biases in long read sequencing via an optional module in Insplico: "The optional module to perform these corrections is a unique feature of Insplico"
- [discussion] Bias correction harmonizes long and short read patterns: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"
- [abstract] Insplico quantifies AISO with both short and long reads: "the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [discussion] Insplico output metrics: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [methods] Reference hg38 genome usage in the workflow: "--readFilesCommand zcat \ --outSAMstrandField intronMotif --genomeDir hg38_reference"

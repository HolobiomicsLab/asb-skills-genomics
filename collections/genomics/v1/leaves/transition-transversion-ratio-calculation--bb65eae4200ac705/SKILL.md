---
name: transition-transversion-ratio-calculation
description: Use when comparing two SNV call sets (e.g. long-read vs short-read) in
  duplicated or low-mappability regions, or when a Ts/Tv ratio is needed to gauge
  call-set quality and reference bias.
when_to_use_negative:
- The variant calls are indels or structural variants — Longshot only calls SNVs (indels
  must be given in an input VCF for genotyping), and the density filter targets SNV
  false-positive clusters.
- The goal is de novo discovery of variant truth rather than quality assessment/benchmarking
  of existing call sets against annotation strata.
- The input regions are non-repetitive, high-mappability regions where short-read
  calling is already reliable — the Ts/Tv contrast between callers is diagnostic specifically
  in poorly mappable duplicated sequence.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: Longshot
  role: Calls and phases SNVs from 30x PacBio SMS reads; supports the --auto_max_cov
    flag implementing the d + 5*sqrt(d) coverage filter used before Ts/Tv stratification
  repo: github.com/pjedge/longshot
- name: FreeBayes
  role: Produces the short-read Illumina SNV call set (fixed GQ 50 cutoff) that is
    compared against the long-read calls via Ts/Tv ratios
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12493-y
    title: Longshot enables accurate variant calling in diploid genomes from single-molecule
      long read sequencing
  source_content_hash: sha256:81e329e0d234b9dff91503d9bc7cd8df053d445e113481632642ee3d7b9238a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FreeBayes
  - Longshot
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Transition-transversion ratio calculation

## Summary

Compute per-stratum transition/transversion (Ts/Tv) ratios for SNV call sets — here, Longshot calls on PacBio SMS reads versus FreeBayes calls on Illumina reads — intersected with segmental-duplication identity strata, to expose mappability-driven false positives and benchmark calling accuracy.

## When to use

Use when comparing two SNV call sets (e.g. long-read vs short-read) in duplicated or low-mappability regions, or when a Ts/Tv ratio is needed to gauge call-set quality and reference bias.

## When NOT to use

- The variant calls are indels or structural variants — Longshot only calls SNVs (indels must be given in an input VCF for genotyping), and the density filter targets SNV false-positive clusters.
- The goal is de novo discovery of variant truth rather than quality assessment/benchmarking of existing call sets against annotation strata.
- The input regions are non-repetitive, high-mappability regions where short-read calling is already reliable — the Ts/Tv contrast between callers is diagnostic specifically in poorly mappable duplicated sequence.

## Inputs

- Longshot VCF of SNVs called from 30x PacBio SMS reads aligned to hs37d5
- FreeBayes VCF of SNVs called from 33x Illumina reads aligned to hs37d5 with BWA-MEM
- GIAB high-confidence region BED and its complement
- Segmental-duplication annotation tracks restricted to >=95% and >=99% identity thresholds

## Outputs

- Per-stratum SNV counts for Longshot and FreeBayes call sets (e.g. Table 2 tallies)
- Per-stratum Ts/Tv ratios, including Illumina-only and SMS-only Ts/Tv within segmental duplications

## How to apply

Call SNVs from both sequencing modalities: run Longshot on the 30x PacBio BAM aligned to hs37d5 with defaults (min_cov 6, max_cov 8000, min_mapq 20, min_alt_count 3, min_alt_frac 0.125) and FreeBayes on the 33x BWA-MEM Illumina BAM. Filter each VCF using a fixed GQ cutoff of 50 for short reads and a variable GQ cutoff matched to the median read coverage for Longshot, then drop false-positive clusters flagged by the density rule (>10 SNVs in a 500 bp window) and excess coverage (d + 5*sqrt(d), or Longshot's --auto_max_cov). Intersect the filtered call sets with strata of interest: GIAB high-confidence regions, their complements, and segmental-duplication tracks at >=95% and >=99% identity. For each stratum and each caller, count transition (A<->G, C<->T) and transversion SNVs and report Ts/Tv; interpret ratios well below the expected ~2.0 as indicating an excess of false positives (e.g. Illumina-only Ts/Tv of 1.55 in >=99% identity duplications versus 1.99 for SMS-only calls). Validate the analysis by checking that the stratified counts and ratios reproduce the expected Table 2 pattern (180,889 vs 116,649 SNVs at >=95%; 78,851 vs 18,684 at >=99%).

## Related tools

- **Longshot** (Calls and phases SNVs from 30x PacBio SMS reads; supports the --auto_max_cov flag implementing the d + 5*sqrt(d) coverage filter used before Ts/Tv stratification) — github.com/pjedge/longshot
- **FreeBayes** (Produces the short-read Illumina SNV call set (fixed GQ 50 cutoff) that is compared against the long-read calls via Ts/Tv ratios)

## Examples

```
longshot --bam pacbio_30x.hs37d5.bam --ref hs37d5.fa --out longshot_snvs.vcf --auto_max_cov
```

## Evaluation signals

- Stratified tallies reproduce Table 2: 180,889 (SMS) vs 116,649 (Illumina) SNVs at >=95% segmental-duplication identity, and 78,851 vs 18,684 at >=99% identity.
- Illumina-only Ts/Tv in >=99% identity duplications equals ~1.55, while SMS-only Ts/Tv is ~1.99; genome-wide Ts/Tv should be near ~2.0 for high-quality call sets.
- All variants counted are genotype-quality filtered (GQ 50 for short reads; variable GQ matched to median coverage for Longshot) with density-filter clusters (>10 SNVs/500 bp) removed.
- Call sets are correctly restricted to the annotation strata (GIAB high-confidence regions, complements, and segmental-duplication tracks at the two identity thresholds) with no double-counting between strata.

## Limitations

- Correctly mapping PacBio reads in highly similar segmental duplications remains a challenge, so even long-read calls in >=99% identity strata are not fully error-free.
- GIAB and Platinum Genomes benchmark sets were generated using short-read datasets and are therefore biased in favor of short-read technologies; some Longshot 'false positives' may be false negatives in the GIAB call sets.
- Allele observations with phred-scaled quality below the default threshold (7.0) are discarded, which reduces the effective read depth for SMS reads and can affect per-stratum counts.

## Evidence

- [results] Illumina-only Ts/Tv in >=99% duplications was 1.55 versus 1.99 for SMS-only calls, consistent with poor Illumina mappability in duplicated regions.: [evidence span withheld — anchor status: not_found]
- [discussion] Variable GQ cutoff for Longshot and fixed GQ 50 for short reads.: "The GQ cutoff was fixed at 50 for short reads."
- [discussion] Density filter removes clustered false variants before ratio computation.: [evidence span withheld — anchor status: not_found]
- [readme] auto_max_cov flag implements the coverage filter used for stratified tallies.: "-A, --auto_max_cov        Automatically calculate mean coverage for region and set max coverage to mean_coverage + 5*sqrt(mean_coverage). (SLOWER)"
- [abstract] Longshot enables variant detection in duplicated regions that cannot be mapped using short reads.: "enables variant detection in duplicated regions of the genome that cannot be mapped using short reads"

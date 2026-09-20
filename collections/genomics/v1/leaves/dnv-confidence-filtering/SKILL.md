---
name: dnv-confidence-filtering
description: Use when joint-genotyped trio VCFs from HAT/Hare (Parabricks GATK 4.2.0
  HaplotypeCaller and DeepVariant 1.4.0 + GLnexus) must be partitioned into high-
  and low-confidence DNV callsets with per-tier validation rates.
when_to_use_negative:
- You only have existing VCFs and no BAM/CRAM files — HAT-FLEX operates directly on
  VCFs instead of the HAT BAM/CRAM-based workflow
- Input is long-read PacBio HiFi data — the Hare v1.2 workflow is effective for WGS
  and WES data only; use Tortoise, which handles WGS, WES and PacBio data
- You need general DNV characterization of already-called variants without confidence
  stratification — acorn functions (extractSNVs, annotateCpG, etc.) work on DNV files
  independent of HAT's confidence tiers
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: HAT (Hare And Tortoise)
  role: Automated DNV detection workflow whose custom de novo variant filter performs
    the high/low confidence stratification
  repo: https://github.com/TNTurnerLab/HAT
- name: Hare v1.2
  role: Workflow variant effective for WGS and WES data (e.g., SPARK WES trios) that
    produces the joint-genotyped files fed to the confidence filter
  repo: https://github.com/TNTurnerLab/Hare
- name: GLnexus
  role: Forms family-level joint-genotyped files that are the direct input to HAT's
    custom de novo variant filter
  repo: https://github.com/TNTurnerLab/HAT
- name: squirrel.py
  role: Preps HAT output (post-filter, confidence-stratified DNVs) into acorn input
    format
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package for follow-up statistics on the filtered, confidence-stratified
    DNV output
  repo: https://github.com/TNTurnerLab/acorn
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:ccd6e8370ea53554b25937325c105da5e2f1020d17f536ac18916fb300808489
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - DeepVariant
  - GATK HaplotypeCaller
  - GLnexus
  - HAT (Hare And Tortoise)
  - Hare
  - Hare v1.2
  - Parabricks
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# dnv-confidence-filtering

## Summary

Stratifies de novo variant (DNV) calls from the HAT workflow into high- versus low-confidence callsets using HAT's custom de novo variant filter applied to family-level joint-genotyped VCFs, enabling confidence-tiered counts and confirmation-rate comparison (e.g., 91.6% vs 70.7% validation on SPARK WES trios).

## When to use

Use when joint-genotyped trio VCFs from HAT/Hare (Parabricks GATK 4.2.0 HaplotypeCaller and DeepVariant 1.4.0 + GLnexus) must be partitioned into high- and low-confidence DNV callsets with per-tier validation rates.

## When NOT to use

- You only have existing VCFs and no BAM/CRAM files — HAT-FLEX operates directly on VCFs instead of the HAT BAM/CRAM-based workflow
- Input is long-read PacBio HiFi data — the Hare v1.2 workflow is effective for WGS and WES data only; use Tortoise, which handles WGS, WES and PacBio data
- You need general DNV characterization of already-called variants without confidence stratification — acorn functions (extractSNVs, annotateCpG, etc.) work on DNV files independent of HAT's confidence tiers

## Inputs

- GLnexus family-level joint-genotyped VCFs per trio (derived from Parabricks v4.0.0-1 GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0 calls)
- Aligned short-read Illumina WES or WGS CRAM/BAM read data for parent-child trios (input to Hare v1.2)

## Outputs

- Per-trio and aggregated DNV callsets stratified into high- and low-confidence tiers (e.g., 282 total = 190 high + 92 low on 100 SPARK WES trios)
- Per-tier confirmation-rate metrics (e.g., 91.6% high-confidence, 70.7% low-confidence)
- acorn-compatible DNV table (sample, chromosome, position, ref, alt) generated via squirrel.py

## How to apply

Run the joint-genotyped family files produced by GLnexus through HAT's custom de novo variant filter, which flags variants present in the offspring but absent from both parents and stratifies them into high- and low-confidence callsets. On WES data such as the 100 SPARK Illumina short-read exome trios, Hare v1.2 is the designated workflow for WGS and WES inputs (CRAM/BAM), and the filter yielded 282 DNVs split into 190 high-confidence and 92 low-confidence calls. Compute confirmation rates per tier (91.6% high-confidence vs 70.7% low-confidence in the SPARK WES validation) to verify the stratification is meaningful. Downstream, prep HAT output with squirrel.py and compute follow-up statistics with the R package acorn. Do not apply this tiering to calls from sources other than HAT/Hare/Tortoise output without revalidation, and use HAT-FLEX only when starting from existing VCFs rather than BAM/CRAM files.

## Related tools

- **HAT (Hare And Tortoise)** (Automated DNV detection workflow whose custom de novo variant filter performs the high/low confidence stratification) — https://github.com/TNTurnerLab/HAT
- **Hare v1.2** (Workflow variant effective for WGS and WES data (e.g., SPARK WES trios) that produces the joint-genotyped files fed to the confidence filter) — https://github.com/TNTurnerLab/Hare
- **GLnexus** (Forms family-level joint-genotyped files that are the direct input to HAT's custom de novo variant filter) — https://github.com/TNTurnerLab/HAT
- **squirrel.py** (Preps HAT output (post-filter, confidence-stratified DNVs) into acorn input format) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package for follow-up statistics on the filtered, confidence-stratified DNV output) — https://github.com/TNTurnerLab/acorn

## Examples

```
R: library('acorn'); dnv <- readDNV('spark_wes_hat_dnvs.txt')  # after squirrel.py converts HAT output to acorn input
```

## Evaluation signals

- Aggregated per-trio counts reproduce the reported totals: 282 DNVs on the 100 SPARK WES trios, partitioned as 190 high confidence and 92 low confidence
- Confirmation-rate split reproduces the reported 91.6% (high confidence) vs 70.7% (low confidence), confirming the tiers carry distinct validation quality
- Every filtered call satisfies the DNV definition: variant present in the offspring but absent from both parents in the GLnexus joint-genotyped trio file
- Output converts cleanly via squirrel.py into an acorn readDNV table (tab-delimited sample, chromosome with 'chr' prefix, position, ref, alt)

## Limitations

- Hare v1.2 is effective for WGS and WES data only; PacBio long-read inputs require the Tortoise workflow
- The high/low confidence split is defined by HAT's custom de novo variant filter; applying it to non-HAT VCFs (e.g., HAT-FLEX output) has not been shown to reproduce these rates
- Counts and rates are cohort-specific (100 SPARK WES trios) and will vary with sequencing depth, capture kit, and trio count

## Evidence

- [methods] GLnexus joint-genotyped input to custom filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] Hare v1.2 scope restriction to WGS/WES: [evidence span withheld — anchor status: not_found]
- [readme] squirrel.py prepping HAT output for acorn: "To prep the output from HAT, you can run squirrel.py found here in this repo."
- [readme] readDNV input format for downstream statistics: [evidence span withheld — anchor status: external_doc]

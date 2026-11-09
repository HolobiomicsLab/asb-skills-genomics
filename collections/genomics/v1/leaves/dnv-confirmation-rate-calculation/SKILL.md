---
name: dnv-confirmation-rate-calculation
description: Use when you need to determine the minimum per-sample coverage (e.g.
  10x/20x/30x) at which a trio DNV caller still recovers a reliable fraction of full-coverage
  DNV calls, e.g. for cost-saving downsampled WGS designs.
when_to_use_negative:
- You only have existing VCFs and no BAM/CRAM access — HAT-FLEX operates directly
  on VCFs but depth downsampling requires aligned reads.
- Your data is noisy long-read (non-HiFi) or the caller arm is Hare, which the paper
  states is effective for WGS and WES data only — use the Tortoise arm for PacBio.
- You need absolute DNV discovery rather than recovery of a known full-coverage callset
  — confirmation rate is a relative metric against a reference callset.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0625
tools:
- name: HAT (Hare And Tortoise)
  role: Automated DNV detection workflow whose Tortoise arm is run on each downsampled
    trio
  repo: https://github.com/TNTurnerLab/HAT
- name: Tortoise
  role: CPU/open-source DNV calling workflow validated for PacBio data; jointly genotypes
    and filters each downsampled trio
  repo: https://github.com/TNTurnerLab/Tortoise
- name: acorn
  role: R package for followup statistics and summarization of HAT DNV output (CpG
    context, variant type breakdowns)
  repo: https://github.com/TNTurnerLab/acorn
- name: squirrel.py
  role: Converts HAT output into acorn input format for downstream DNV statistics
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:ca2cbe57b348c8e336b9caa0ff9b0b9225f6e76575715bfca661ff96db8f2910
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - HAT
  - HAT (Hare And Tortoise)
  - Tortoise
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# dnv-confirmation-rate-calculation

## Summary

Quantifies the fraction of de novo variant (DNV) calls recovered at reduced sequencing depth by comparing downsampled-depth callsets against a full-coverage reference callset, using the HAT/Tortoise workflow on PacBio HiFi trio data. It establishes the minimum per-sample coverage needed for accurate DNV calling.

## When to use

Use when you need to determine the minimum per-sample coverage (e.g. 10x/20x/30x) at which a trio DNV caller still recovers a reliable fraction of full-coverage DNV calls, e.g. for cost-saving downsampled WGS designs.

## When NOT to use

- You only have existing VCFs and no BAM/CRAM access — HAT-FLEX operates directly on VCFs but depth downsampling requires aligned reads.
- Your data is noisy long-read (non-HiFi) or the caller arm is Hare, which the paper states is effective for WGS and WES data only — use the Tortoise arm for PacBio.
- You need absolute DNV discovery rather than recovery of a known full-coverage callset — confirmation rate is a relative metric against a reference callset.

## Inputs

- Full-coverage PacBio HiFi trio aligned reads (BAM/CRAM) for child and both parents
- Full-coverage HAT/Tortoise DNV callset used as the confirmation reference
- Subsampling seed specifications (single seed at 10x; seeds 0–99 at 20x and 30x)
- Unique (non-repetitive) region annotation BED for the unique-region confirmation rate

## Outputs

- Per-replicate downsampled DNV callsets (VCF) from the custom de novo variant filter
- downsampling_depth_summary.csv with per-depth confirmation rates and mean ± SD DNV counts
- Confirmation rates per depth, overall and restricted to unique regions (e.g. 62.4% at 10x, 91.9% at 30x, 96.3% unique)

## How to apply

Take the full-coverage trio BAM/CRAM (e.g. the 9p.100 PacBio HiFi trio) and downsample to target depths (10x, 20x, 30x) using seeded random read subsampling — a single seed at 10x and 100 replicate seeds (0–99) at 20x and 30x to capture stochastic variance. Run the Tortoise arm of HAT (the arm validated for PacBio data) on each downsampled trio to jointly genotype and filter de novo variants, then apply the custom de novo variant filter to each joint-genotyped family file to get per-replicate DNV callsets. For each replicate, compute the confirmation rate as the fraction of downsampled DNVs recovered relative to the full-coverage callset, including a rate restricted to unique (non-repetitive) regions, and aggregate mean ± SD DNV counts across replicates (benchmarks: 105 ± 11 at 20x, 80 ± 2 at 30x). Accept the analysis when per-depth confirmation rates match reported values within their uncertainty ranges (62.4% at 10x, 91.9% at 30x, 96.3% in unique regions), recorded in downsampling_depth_summary.csv.

## Related tools

- **HAT (Hare And Tortoise)** (Automated DNV detection workflow whose Tortoise arm is run on each downsampled trio) — https://github.com/TNTurnerLab/HAT
- **Tortoise** (CPU/open-source DNV calling workflow validated for PacBio data; jointly genotypes and filters each downsampled trio) — https://github.com/TNTurnerLab/Tortoise
- **acorn** (R package for followup statistics and summarization of HAT DNV output (CpG context, variant type breakdowns)) — https://github.com/TNTurnerLab/acorn
- **squirrel.py** (Converts HAT output into acorn input format for downstream DNV statistics) — https://github.com/TNTurnerLab/HAT

## Examples

```
Rscript -e "library('acorn'); dnv <- readDNV('downsampling_depth_summary_DNVs.txt.gz')"  # after running Tortoise on each downsampled trio and applying the custom de novo filter
```

## Evaluation signals

- Confirmation rate at 30x reaches ~91.9% overall and 96.3% when restricted to unique (non-repetitive) regions.
- Confirmation rate at 10x single-seed run matches ~62.4% relative to the full-coverage callset.
- Mean DNV counts across 100 replicates fall within reported uncertainty: 105 ± 11 at 20x and 80 ± 2 at 30x.
- Per-replicate counts at 30x have very low spread (SD ≈ 2), indicating the depth-vs-stability inflection is captured.
- All per-depth confirmation rates and mean counts are recorded in downsampling_depth_summary.csv and match reported values within their ranges.

## Limitations

- Confirmation is defined relative to the full-coverage callset, so biases in that callset propagate to all downsampled estimates.
- Rates are depth- and caller-specific: Tortoise (open-source arm) was the arm validated for PacBio HiFi; Hare is effective for WGS and WES data only.
- At 10x coverage the confirmation rate drops sharply (62.4%), so counts below ~20x are unreliable for DNV calling.
- Unique-region rates are higher than genome-wide rates because repetitive regions lose calls preferentially at lower depth; interpret genome-wide rates accordingly.

## Evidence

- [methods] Tortoise workflow is effective for WGS, WES and PacBio data: "This workflow is effect for WGS, WES and PacBio data."
- [intro] GLnexus joint genotyping and custom de novo filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [abstract] HAT input is aligned trio reads (CRAM or BAM): "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."
- [methods] Hare is WGS/WES only, motivating the Tortoise choice: [evidence span withheld — anchor status: not_found]
- [abstract] HAT detects high-quality DNVs from PacBio HiFi long-read WGS: "HAT detects high-quality DNVs from Illumina short-read whole-exome sequencing, Illumina short-read whole-genome sequencing, and highly accurate PacBio HiFi long-read whole-genome sequencing data."

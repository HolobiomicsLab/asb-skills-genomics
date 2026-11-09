---
name: coverage-depth-benchmark-reproduction
description: Use when a task requires determining the minimum per-sample sequencing
  depth for accurate de novo variant calling, or when reproducing/validating reported
  confirmation rates and DNV counts across downsampled coverage depths for a sequenced
  trio.
when_to_use_negative:
- Input is only existing VCFs without BAM/CRAM access — use HAT-FLEX for trio DNV
  detection on VCFs instead of downsampling reads
- Data are short-read Illumina WGS/WES where the GPU-accelerated Hare workflow is
  the intended arm, or where downsampling benchmarks specific to PacBio HiFi error
  profiles are not applicable
- The goal is post-hoc summarization of already-called DNVs (use acorn) rather than
  depth-dependent re-calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: HAT (Hare And Tortoise)
  role: Overall de novo variant calling workflow applied to each downsampled trio
  repo: https://github.com/TNTurnerLab/HAT
- name: Tortoise
  role: CPU, open-source arm of HAT validated for PacBio HiFi data; runs variant calling
    and joint genotyping on downsampled trios
  repo: https://github.com/TNTurnerLab/Tortoise
- name: GLnexus
  role: Forms family-level joint-genotyped files passed to the custom de novo variant
    filter
- name: acorn
  role: Optional R package for follow-up statistics on HAT DNV output
  repo: https://github.com/TNTurnerLab/acorn
- name: squirrel.py
  role: Converts HAT output to acorn input format for downstream statistics
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:42971fdfd19c7fa57b2004ee1351b9e0a9c325090ee8550418ba993e931a66e5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - GLnexus
  - HAT
  - HAT (Hare And Tortoise)
  - Tortoise
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# coverage-depth-benchmark-reproduction

## Summary

Reproduces coverage-downsampling benchmarks for de novo variant (DNV) calling by seeding subsampling a PacBio HiFi trio to 10x/20x/30x, rerunning the HAT Tortoise workflow at each depth, and validating confirmation rates and DNV counts against reported values (62.4% at 10x, 91.9% at 30x, 105 +/- 11 DNVs at 20x, 80 +/- 2 at 30x).

## When to use

Use when a task requires determining the minimum per-sample sequencing depth for accurate de novo variant calling, or when reproducing/validating reported confirmation rates and DNV counts across downsampled coverage depths for a sequenced trio.

## When NOT to use

- Input is only existing VCFs without BAM/CRAM access — use HAT-FLEX for trio DNV detection on VCFs instead of downsampling reads
- Data are short-read Illumina WGS/WES where the GPU-accelerated Hare workflow is the intended arm, or where downsampling benchmarks specific to PacBio HiFi error profiles are not applicable
- The goal is post-hoc summarization of already-called DNVs (use acorn) rather than depth-dependent re-calling

## Inputs

- Aligned trio reads (BAM or CRAM) for parent-child trio, e.g., 9p.100 PacBio HiFi WGS
- Full-coverage DNV callset (reference against which confirmation rates are computed)
- Subsampling seeds: 1 seed for 10x, seeds 0-99 for 20x and 30x replicates
- Unique (non-repetitive) region definitions for restricted confirmation-rate calculation

## Outputs

- Per-replicate DNV call sets from the custom de novo variant filter at each downsampled depth
- downsampling_depth_summary.csv with per-depth confirmation rates and mean +/- SD DNV counts
- Aggregate statistics: DNV counts per replicate (e.g., 105 +/- 11 at 20x, 80 +/- 2 at 30x) and confirmation rates (62.4% at 10x, 91.9% at 30x, 96.3% in unique regions)

## How to apply

Start from aligned reads (BAM/CRAM) of the target trio (e.g., the 9p.100 PacBio HiFi trio) and downsample to 10x, 20x, and 30x target coverage with seeded random read subsampling — a single seed at 10x and 100 replicate seeds (0-99) at 20x and 30x to capture stochastic variation. Run the Tortoise arm of HAT (the caller validated for PacBio HiFi data; Hare is GPU/Parabricks-based and Hare v1.2 is WGS/WES only) on each downsampled trio to jointly genotype and filter de novo variants. Apply the custom de novo variant filter to each joint-genotyped family file, then compute per-replicate DNV counts and confirmation rates — the fraction of downsampled DNVs recovered relative to the full-coverage callset — including the rate restricted to unique (non-repetitive) regions. Aggregate mean +/- SD DNV counts across the 100 replicates and compare against reported benchmarks (105 +/- 11 at 20x, 80 +/- 2 at 30x; 62.4% at 10x, 91.9% at 30x, 96.3% in unique regions), recording results in downsampling_depth_summary.csv.

## Related tools

- **HAT (Hare And Tortoise)** (Overall de novo variant calling workflow applied to each downsampled trio) — https://github.com/TNTurnerLab/HAT
- **Tortoise** (CPU, open-source arm of HAT validated for PacBio HiFi data; runs variant calling and joint genotyping on downsampled trios) — https://github.com/TNTurnerLab/Tortoise
- **GLnexus** (Forms family-level joint-genotyped files passed to the custom de novo variant filter)
- **acorn** (Optional R package for follow-up statistics on HAT DNV output) — https://github.com/TNTurnerLab/acorn
- **squirrel.py** (Converts HAT output to acorn input format for downstream statistics) — https://github.com/TNTurnerLab/HAT

## Evaluation signals

- Confirmation rate at 10x matches ~62.4% and at 30x matches ~91.9% (rising to 96.3% in unique regions) relative to the full-coverage callset
- Across 100 replicate seeds, mean DNV count at 20x is 105 +/- 11 and at 30x is 80 +/- 2
- Per-replicate DNV call sets are produced for all seeds (1 at 10x, 100 each at 20x and 30x) via the custom de novo variant filter on joint-genotyped family files
- downsampling_depth_summary.csv is produced with per-depth confirmation rates and mean +/- SD DNV counts falling within reported uncertainty ranges

## Limitations

- Tortoise is the only HAT arm validated for PacBio data; Hare v1.2 is effective for WGS and WES data only
- Downsampled DNV counts fluctuate with subsampling seed — single-seed runs are insufficient; the reported SDs (e.g., 80 +/- 2 at 30x) require 100 replicate seeds
- Confirmation rate measured genome-wide (91.9% at 30x) underestimates the rate in unique regions (96.3%), so repeat/repetitive regions should be reported separately

## Evidence

- [methods] Tortoise workflow effective for PacBio: [evidence span withheld — anchor status: not_found]
- [methods] Hare scope restriction: [evidence span withheld — anchor status: not_found]
- [intro] GLnexus joint genotyping and custom filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] acorn for followup statistics: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
- [abstract] HAT input/output definition: "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."

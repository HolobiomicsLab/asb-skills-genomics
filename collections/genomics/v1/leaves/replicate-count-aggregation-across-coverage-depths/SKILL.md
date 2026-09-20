---
name: replicate-count-aggregation-across-coverage-depths
description: Use when quantifying how many DNVs survive and what fraction of a full-coverage
  DNV callset is recovered as sequencing depth is reduced, e.g. 10x/20x/30x downsampling
  of BAM/CRAM trios.
when_to_use_negative:
- Input is already a full-coverage callset and the goal is DNV characterization rather
  than depth sensitivity — use acorn directly instead
- Only existing VCFs are available with no BAM/CRAM access — downsampled reruns require
  aligned reads; use HAT-FLEX for VCF-only trios
- The data are not from a parent-child trio or not from highly accurate reads (Illumina
  short-read or PacBio HiFi)
edam_operation: http://edamontology.org/operation_3227
tools:
- name: Tortoise
  role: Open-source, CPU-only HAT arm used to rerun DNV calling on each downsampled
    trio; effective for WGS, WES and PacBio data
  repo: https://github.com/TNTurnerLab/Tortoise
- name: HAT
  role: Parent workflow (Hare And Tortoise) providing the custom de novo variant filter
    and DNV detection applied per replicate
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package for followup statistics on HAT output; reads DNV files via readDNV
    to summarize replicate call characteristics
  repo: https://github.com/TNTurnerLab/acorn
- name: GLnexus
  role: Forms family-level joint-genotyped files that are run through the custom de
    novo variant filter in each replicate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:46588b6e0c9e4a35f62b7bd2e0af561648068efebfbcc8a47ccbbdccaa235dbc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Tortoise
  tools:
  - GLnexus
  - HAT
  - Tortoise
  - acorn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# Replicate count aggregation across coverage depths

## Summary

Aggregates de novo variant (DNV) counts and confirmation rates across seeded downsampling replicates of a PacBio HiFi trio at multiple depths (10x, 20x, 30x) to determine the minimum per-sample coverage needed for reliable DNV calling with HAT/Tortoise.

## When to use

Use when quantifying how many DNVs survive and what fraction of a full-coverage DNV callset is recovered as sequencing depth is reduced, e.g. 10x/20x/30x downsampling of BAM/CRAM trios.

## When NOT to use

- Input is already a full-coverage callset and the goal is DNV characterization rather than depth sensitivity — use acorn directly instead
- Only existing VCFs are available with no BAM/CRAM access — downsampled reruns require aligned reads; use HAT-FLEX for VCF-only trios
- The data are not from a parent-child trio or not from highly accurate reads (Illumina short-read or PacBio HiFi)

## Inputs

- Aligned PacBio HiFi trio reads (BAM/CRAM) for parent-child trio, e.g. the 9p.100 trio
- Full-coverage DNV callset from the same trio (reference for confirmation-rate calculation)
- Downsampled BAM/CRAM files at 10x, 20x, and 30x with per-replicate seeds

## Outputs

- Per-replicate DNV call sets (.vcf) from each downsampled trio run
- downsampling_depth_summary.csv recording per-depth confirmation rates and mean +/- SD DNV counts
- Mean +/- SD DNV counts across 100 replicates (e.g. 105 +/- 11 at 20x, 80 +/- 2 at 30x)

## How to apply

Downsample the aligned trio reads (BAM/CRAM) to target coverage with seeded random read subsampling — a single seed at 10x and 100 replicate seeds (0-99) at 20x and 30x. Rerun the Tortoise arm of HAT (validated for PacBio HiFi data) on each downsampled trio to jointly genotype and apply the custom de novo variant filter, yielding per-replicate DNV call sets. Aggregate DNV counts per replicate and compute confirmation rates as the fraction of the full-coverage callset recovered, including the rate restricted to unique (non-repetitive) regions. Compute mean +/- SD across the 100 replicates at 20x and 30x, and compare against benchmarks (105 +/- 11 at 20x, 80 +/- 2 at 30x; 62.4% at 10x, 91.9% at 30x, 96.3% in unique regions), accepting when rates and counts fall within reported uncertainty. HAT output can be prepped with squirrel.py and summarized with the acorn R package if followup DNV statistics are needed.

## Related tools

- **Tortoise** (Open-source, CPU-only HAT arm used to rerun DNV calling on each downsampled trio; effective for WGS, WES and PacBio data) — https://github.com/TNTurnerLab/Tortoise
- **HAT** (Parent workflow (Hare And Tortoise) providing the custom de novo variant filter and DNV detection applied per replicate) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package for followup statistics on HAT output; reads DNV files via readDNV to summarize replicate call characteristics) — https://github.com/TNTurnerLab/acorn
- **GLnexus** (Forms family-level joint-genotyped files that are run through the custom de novo variant filter in each replicate)

## Evaluation signals

- Per-depth confirmation rates match reported values: 62.4% at 10x, 91.9% at 30x, 96.3% in unique regions
- Mean DNV counts across 100 replicates match 105 +/- 11 at 20x and 80 +/- 2 at 30x within reported uncertainty
- Confirmation rate increases monotonically with downsampling depth (62.4% -> 91.9% from 10x to 30x)
- downsampling_depth_summary.csv is populated with per-depth confirmation rates and mean +/- SD counts
- Each 20x and 30x replicate uses a distinct seed in the 0-99 range, giving exactly 100 call sets per depth

## Limitations

- Confirmation rates are relative to the full-coverage callset, so errors in the reference callset propagate to all depth estimates
- Results are calibrated for PacBio HiFi long-read trio data via the Tortoise arm; Hare is effective for WGS and WES data only
- The reported error bars are derived from 100 replicates at 20x and 30x only; the 10x point uses a single seed and has no replicate uncertainty

## Evidence

- [methods] Tortoise is the PacBio-validated arm: "Tortoise uses freely available, open source versions of the these variant callers"
- [intro] Custom filter applied to joint-genotyped family files: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [readme] Tortoise accepts CRAM/BAM trio input and reports de novo variants: [evidence span withheld — anchor status: external_doc]
- [abstract] HAT supports PacBio HiFi long-read WGS DNV detection: "HAT detects high-quality DNVs from Illumina short-read whole-exome sequencing, Illumina short-read whole-genome sequencing, and highly accurate PacBio HiFi long-read whole-genome sequencing data."
- [readme] squirrel.py and acorn enable post-HAT statistics on DNV output: [evidence span withheld — anchor status: external_doc]

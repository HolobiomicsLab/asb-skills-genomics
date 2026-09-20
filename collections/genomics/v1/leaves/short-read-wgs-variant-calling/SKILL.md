---
name: short-read-wgs-variant-calling
description: Use when starting from aligned short-read Illumina WGS (or WES) BAM/CRAM
  files for a sequenced parent-child trio and the goal is to produce high-quality
  de novo variant (DNV) calls rather than standard germline variant sets.
when_to_use_negative:
- The input is already called variants (VCFs) with no BAM/CRAM access — HAT-FLEX operates
  directly on existing VCFs instead
- Only GPU-accelerated execution is expected but no GPU resources exist — run Tortoise
  (open-source CPU-only pipeline) rather than Hare
- The data are long-read PacBio HiFi only — Hare v1.2 is effective for WGS and WES
  data only; Tortoise covers PacBio HiFi
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Hare v1.2
  role: GPU-accelerated de novo variant calling workflow for short-read WGS/WES trios
    via NVIDIA Parabricks
  repo: https://github.com/TNTurnerLab/Hare
- name: Tortoise
  role: CPU-only, open-source trio DNV calling pipeline using GATK HaplotypeCaller,
    DeepVariant, and GLnexus on Illumina short-read BAM/CRAM
  repo: https://github.com/TNTurnerLab/Tortoise
- name: Parabricks 4.0.0-1
  role: NVIDIA GPU-accelerated variant calling backend used by Hare for HaplotypeCaller
    GATK 4.2.0 and DeepVariant 1.4.0
- name: GLnexus
  role: Forms family-level joint-genotyped files that are then run through the custom
    de novo variant filter
- name: HAT-FLEX
  role: Caller-agnostic drop-in trio DNV detection tool operating directly on existing
    VCFs when BAM/CRAM access is unavailable
  repo: https://github.com/TNTurnerLab/HAT-FLEX
- name: acorn
  role: R package for follow-up statistics on HAT DNV output (extract SNVs/INDELs/MNVs,
    CpG annotation, Ti/Tv ratios)
  repo: https://github.com/TNTurnerLab/acorn
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:063c72e854be953370b2cfa9b8e791c0401e26156e9355496807f6d4c9275726
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Hare
  tools:
  - DeepVariant
  - GATK
  - GLnexus
  - HAT
  - HAT-FLEX
  - Hare
  - Hare v1.2
  - Parabricks
  - Parabricks 4.0.0-1
  - Tortoise
  - acorn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42105263157894735
n_claims: 19
---

# short-read-wgs-variant-calling

## Summary

Call variants from short-read Illumina WGS trios using the HAT toolchain — Hare (GPU-accelerated via NVIDIA Parabricks 4.0.0-1) or Tortoise (open-source CPU versions of GATK HaplotypeCaller 4.2.0 and DeepVariant 1.4.0) — followed by GLnexus family-level joint genotyping and a custom de novo variant filter to produce de novo variant (DNV) calls.

## When to use

Use when starting from aligned short-read Illumina WGS (or WES) BAM/CRAM files for a sequenced parent-child trio and the goal is to produce high-quality de novo variant (DNV) calls rather than standard germline variant sets.

## When NOT to use

- The input is already called variants (VCFs) with no BAM/CRAM access — HAT-FLEX operates directly on existing VCFs instead
- Only GPU-accelerated execution is expected but no GPU resources exist — run Tortoise (open-source CPU-only pipeline) rather than Hare
- The data are long-read PacBio HiFi only — Hare v1.2 is effective for WGS and WES data only; Tortoise covers PacBio HiFi

## Inputs

- Aligned short-read Illumina BAM/CRAM files for a family trio (father, mother, child), e.g. GIAB HG002 trio 30x Illumina WGS reads
- Human reference genome
- Optional: VCFs from any caller as input to HAT-FLEX when BAM/CRAM files are not available

## Outputs

- Per-sample variant calls from GATK HaplotypeCaller 4.2.0 and DeepVariant 1.4.0 (or Parabricks-accelerated equivalents)
- Family-level joint-genotyped VCFs from GLnexus
- Filtered de novo variant (DNV) callset VCF per child, ready for downstream analysis (e.g. acorn input via squirrel.py)

## How to apply

Begin with aligned read data (CRAM or BAM) from a parent-child sequenced trio and run one of the two HAT callers: Hare v1.2, which uses NVIDIA Parabricks v4.0.0-1 to GPU-accelerate variant calling with GATK HaplotypeCaller 4.2.0 and DeepVariant 1.4.0, or Tortoise, which runs freely available, open-source CPU versions of the same variant callers. Then use GLnexus to form family-level joint-genotyped files, and pass those files through the custom de novo variant filter to identify variants present in the offspring but absent from both parents. Choose Hare for WGS/WES data when GPU resources are available (it is effective for WGS and WES data only); choose Tortoise when CPU-only execution is needed. Downstream, squirrel.py can prep HAT output for the acorn R package for follow-up DNV statistics.

## Related tools

- **Hare v1.2** (GPU-accelerated de novo variant calling workflow for short-read WGS/WES trios via NVIDIA Parabricks) — https://github.com/TNTurnerLab/Hare
- **Tortoise** (CPU-only, open-source trio DNV calling pipeline using GATK HaplotypeCaller, DeepVariant, and GLnexus on Illumina short-read BAM/CRAM) — https://github.com/TNTurnerLab/Tortoise
- **Parabricks 4.0.0-1** (NVIDIA GPU-accelerated variant calling backend used by Hare for HaplotypeCaller GATK 4.2.0 and DeepVariant 1.4.0)
- **GLnexus** (Forms family-level joint-genotyped files that are then run through the custom de novo variant filter)
- **HAT-FLEX** (Caller-agnostic drop-in trio DNV detection tool operating directly on existing VCFs when BAM/CRAM access is unavailable) — https://github.com/TNTurnerLab/HAT-FLEX
- **acorn** (R package for follow-up statistics on HAT DNV output (extract SNVs/INDELs/MNVs, CpG annotation, Ti/Tv ratios)) — https://github.com/TNTurnerLab/acorn

## Evaluation signals

- The output contains de novo variants in the child that are absent from both parents after GLnexus joint genotyping and the custom de novo variant filter
- On the GIAB HG002 trio 30x Illumina WGS benchmark, call quality holds per the paper's metrics: DNVs per individual, percent of DNVs at CpG sites, and percent of DNVs phased to the paternal chromosome of origin
- Hare was tested with output from Parabricks v.3.0.0 and the free version of Parabricks v4.0.0-1 — confirm the Parabricks version matches a tested configuration
- The filtered DNV callset can be converted with squirrel.py and loaded by acorn's readDNV (tab-delimited sample, chr, position, ref, alt format) without schema errors

## Limitations

- Hare v1.2 is effective for WGS and WES data only — it does not cover PacBio HiFi long-read data
- Hare requires GPU resources (Parabricks); without them, the CPU-only Tortoise workflow must be used instead
- HAT as a workflow requires BAM/CRAM access; if only VCFs exist, HAT-FLEX must be substituted (it is beta status)

## Evidence

- [abstract] HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs.: "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."
- [intro] uses the software Parabricks, v4.0.0-1, by NVIDIA, that leverages GPUs to accelerate variant calling: "uses the software Parabricks, v4.0.0-1, by NVIDIA, that leverages GPUs to accelerate variant calling"
- [intro] We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] This workflow is effective for WGS and WES data only.: "This workflow is effective for WGS and WES data only."
- [readme] This pipeline takes in aligned, short-read Illumina data, in the form of .cram or .bam files, for a family trio and reports *de novo* variants in a .vcf file.: [evidence span withheld — anchor status: external_doc]

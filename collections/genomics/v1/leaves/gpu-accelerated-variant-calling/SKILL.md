---
name: gpu-accelerated-variant-calling
description: Use when aligned parent-child trio CRAM/BAM short-read data (WGS or WES)
  needs variant calling and GPU resources are available to accelerate GATK HaplotypeCaller
  and DeepVariant via Parabricks.
when_to_use_negative:
- Input consists of existing VCFs rather than CRAM/BAM reads — use HAT-FLEX, the caller-agnostic
  drop-in trio DNV detection tool, instead
- No GPU access is available — use the Tortoise workflow, which uses freely available
  open-source CPU-only versions of the variant callers
- Data are PacBio HiFi long-read WGS — the Hare workflow is effective for WGS and
  WES data only; use Tortoise, which also supports PacBio data
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Hare
  role: DNV calling workflow that wraps the GPU-accelerated Parabricks variant calling
    step; v1.2 is effective for WGS and WES data only
  repo: https://github.com/TNTurnerLab/Hare
- name: Parabricks 4.0.0-1
  role: NVIDIA software that leverages GPUs to accelerate variant calling with GATK
    4.2.0 HaplotypeCaller and DeepVariant v1.4.0
- name: GLnexus
  role: Forms family-level joint-genotyped files from the per-trio variant calls
- name: Tortoise
  role: CPU-only alternative workflow using open-source versions of the same variant
    callers when GPUs are not available
  repo: https://github.com/TNTurnerLab/Tortoise
- name: HAT (Hare And Tortoise)
  role: Parent automated DNV detection workflow into which the GPU-accelerated variant
    calling step feeds via the custom de novo variant filter
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:4ba21db816263ce14082562d2bd57d68fc5101fd869b6267489f97fb3dabe742
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Hare
  tools:
  - DeepVariant
  - GATK
  - GATK HaplotypeCaller
  - GLnexus
  - HAT
  - HAT (Hare And Tortoise)
  - Hare
  - Parabricks
  - Parabricks 4.0.0-1
  - Tortoise
  - acorn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# gpu-accelerated-variant-calling

## Summary

GPU-accelerated trio variant calling using NVIDIA Parabricks v4.0.0-1 (wrapping GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0) on aligned CRAM/BAM reads, as the compute front-end of the HAT/Hare de novo variant calling workflow.

## When to use

Use when aligned parent-child trio CRAM/BAM short-read data (WGS or WES) needs variant calling and GPU resources are available to accelerate GATK HaplotypeCaller and DeepVariant via Parabricks.

## When NOT to use

- Input consists of existing VCFs rather than CRAM/BAM reads — use HAT-FLEX, the caller-agnostic drop-in trio DNV detection tool, instead
- No GPU access is available — use the Tortoise workflow, which uses freely available open-source CPU-only versions of the variant callers
- Data are PacBio HiFi long-read WGS — the Hare workflow is effective for WGS and WES data only; use Tortoise, which also supports PacBio data

## Inputs

- Aligned CRAM/BAM read data from a parent-child sequenced trio (Illumina short-read WGS or WES)
- Reference genome indexes for Parabricks variant calling

## Outputs

- Per-trio variant calls from GATK HaplotypeCaller and DeepVariant (VCF)
- GLnexus family-level joint-genotyped files ready for HAT's custom de novo variant filter

## How to apply

Take aligned CRAM or BAM read data from a parent-child sequenced trio as input and run the Hare v1.2 workflow, which invokes NVIDIA Parabricks v4.0.0-1 to GPU-accelerate variant calling with GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0. Hare v1.2 is validated against outputs of Parabricks v3.0.0 as well as the free Parabricks v4.0.0.0-1, so version consistency of the Parabricks install should be checked before launch. The per-trio variant calls are then passed to GLnexus to form family-level joint-genotyped files, which feed HAT's custom de novo variant filter to extract variants present in the offspring but absent from both parents. Rationale: GPU acceleration drastically reduces runtime for large cohorts (e.g., 4216 SSC WGS trios or 100 SPARK WES trios) while retaining caller fidelity. If GPUs are unavailable, swap in the Tortoise workflow, which uses freely available open-source CPU-only versions of the same variant callers.

## Related tools

- **Hare** (DNV calling workflow that wraps the GPU-accelerated Parabricks variant calling step; v1.2 is effective for WGS and WES data only) — https://github.com/TNTurnerLab/Hare
- **Parabricks 4.0.0-1** (NVIDIA software that leverages GPUs to accelerate variant calling with GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0)
- **GLnexus** (Forms family-level joint-genotyped files from the per-trio variant calls)
- **Tortoise** (CPU-only alternative workflow using open-source versions of the same variant callers when GPUs are not available) — https://github.com/TNTurnerLab/Tortoise
- **HAT (Hare And Tortoise)** (Parent automated DNV detection workflow into which the GPU-accelerated variant calling step feeds via the custom de novo variant filter) — https://github.com/TNTurnerLab/HAT

## Evaluation signals

- Per-trio VCFs are produced for both HaplotypeCaller (GATK 4.2.0) and DeepVariant v1.4.0 branches before GLnexus joint genotyping
- Downstream HAT filter output is consistent with published cohort totals: 329,589 DNVs across 4216 SSC short-read WGS trios and 282 DNVs (190 high / 92 low confidence) across 100 SPARK WES trios
- Resulting DNV QC metrics (DNVs per individual, percent at CpG sites, percent phased to the paternal chromosome of origin) fall in line with published expectations
- Parabricks version matches a tested release (v3.0.0 or free v4.0.0.0-1) as documented for the Hare workflow

## Limitations

- Requires GPU hardware; the open-source CPU-only Tortoise pipeline is the fallback but with different runtime characteristics
- The Hare v1.2 workflow is effective for WGS and WES data only — it does not cover PacBio long-read data
- Workflow has been tested with specific Parabricks versions (v3.0.0 and free v4.0.0.0-1); other versions are unvalidated

## Evidence

- [readme] uses the software Parabricks, v4.0.0-1, by NVIDIA, that leverages GPUs to accelerate variant calling, specifically for Haplotyecaller GATK 4.2.0 and DeepVariant v1.4.0: "uses the software Parabricks, v4.0.0-1, by NVIDIA, that leverages GPUs to accelerate variant calling, specifically for Haplotyecaller GATK 4.2.0 and DeepVariant v1.4.0."
- [methods] The updated Hare, v1.2, workflow can be found [here]. This workflow is effective for WGS and WES data only.: [evidence span withheld — anchor status: not_found]
- [readme] This version has been tested with the output from Parabricks v.3.0.0, as well as the free version of [Parabricks v4.0.0.0-1]: [evidence span withheld — anchor status: external_doc]
- [readme] This pipeline takes in aligned, short-read Illumina data, in the form of .cram or .bam files, for a family trio and reports *de novo* variants in a .vcf file.: [evidence span withheld — anchor status: external_doc]

---
name: trio-de-novo-variant-detection
description: Use when you have aligned CRAM/BAM reads (Illumina WGS/WES or PacBio
  HiFi) from a parent-child trio and need to identify variants present in the offspring
  but absent from both parents.
when_to_use_negative:
- You have low-quality or inaccurate long reads rather than highly accurate short-read
  or PacBio HiFi data
- You lack access to BAM/CRAM files and only have VCFs — use HAT-FLEX for drop-in
  trio DNV detection on existing VCFs instead
- Input is not a parent-child trio (HAT is designed for trio de novo detection, not
  somatic or population calling)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0625
tools:
- name: HAT
  role: Umbrella de novo variant detection workflow (Hare + Tortoise) from trio CRAM/BAM
    input
  repo: https://github.com/TNTurnerLab/HAT
- name: Hare
  role: GPU-accelerated workflow arm (Parabricks v4.0.0-1, GATK 4.2.0 HaplotypeCaller,
    DeepVariant v1.4.0) for WGS and WES data
  repo: https://github.com/TNTurnerLab/Hare
- name: Tortoise
  role: Open-source, CPU-only workflow arm for WGS, WES, and PacBio data using DeepVariant
    and GATK HaplotypeCaller
  repo: https://github.com/TNTurnerLab/Tortoise
- name: GLnexus
  role: Forms family-level joint-genotyped files passed to the custom de novo variant
    filter
- name: HAT-FLEX
  role: Caller-agnostic drop-in trio DNV detection directly on existing VCFs with
    allele-level intersection and sex/PAR-aware logic
  repo: https://github.com/TNTurnerLab/HAT-FLEX
- name: squirrel.py
  role: Converts HAT output to acorn input format
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package for followup statistics on HAT DNV output (extraction, CpG annotation,
    Ti/Tv, INDEL ratios)
  repo: https://github.com/TNTurnerLab/acorn
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_008
  - task_003
  - task_001
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:b2944b71adf3351bf49aac52408917b9bed7fd549ff4b892b828cb26b9348a86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - DeepVariant
  - GATK
  - GATK HaplotypeCaller
  - GLnexus
  - HAT
  - HAT (Hare And Tortoise)
  - HAT-FLEX
  - Hare
  - Parabricks
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
yes_rate: 0.6956521739130435
n_claims: 23
---

# trio-de-novo-variant-detection

## Summary

Detect de novo variants (DNVs) from a parent-child sequenced trio by running variant calling (GPU-accelerated Hare or open-source Tortoise), GLnexus family-level joint genotyping, and HAT's custom de novo variant filter on aligned CRAM/BAM reads.

## When to use

Use when you have aligned CRAM/BAM reads (Illumina WGS/WES or PacBio HiFi) from a parent-child trio and need to identify variants present in the offspring but absent from both parents.

## When NOT to use

- You have low-quality or inaccurate long reads rather than highly accurate short-read or PacBio HiFi data
- You lack access to BAM/CRAM files and only have VCFs — use HAT-FLEX for drop-in trio DNV detection on existing VCFs instead
- Input is not a parent-child trio (HAT is designed for trio de novo detection, not somatic or population calling)

## Inputs

- Aligned CRAM or BAM read data from a parent-child sequenced trio (Illumina short-read WGS/WES or PacBio HiFi long-read WGS)
- GRCh38 reference genome
- Optionally, existing trio or multi-sample VCFs (for HAT-FLEX)

## Outputs

- Per-trio de novo variant (DNV) callset in VCF format
- HAT output convertible via squirrel.py into acorn input (sample, chromosome, position, ref, alt tab-delimited format)
- Quality metric summaries (DNVs per individual, percent at CpG sites, percent phased to paternal chromosome of origin)

## How to apply

Start with aligned read data (CRAM or BAM) from a parent-child sequenced trio and choose the workflow arm by data type: Hare v1.2 (NVIDIA Parabricks v4.0.0-1 wrapping GATK 4.2.0 HaplotypeCaller and DeepVariant v1.4.0) for WGS/WES, or Tortoise (open-source, CPU-only callers) for WGS, WES, or PacBio HiFi data. Run per-sample variant calling, then use GLnexus to form family-level joint-genotyped files. Pass the joint-genotyped files through HAT's custom de novo variant filter to extract DNVs present in the child but absent from both parents. If you only have existing VCFs and no BAM/CRAM access, use HAT-FLEX instead, which operates directly on VCFs with allele-level intersection, sex/PAR-aware logic, and clustering. Summarize results by prepping HAT output with squirrel.py and computing followup statistics with the R package acorn (e.g., DNVs per individual, percent at CpG sites via annotateCpG).

## Related tools

- **HAT** (Umbrella de novo variant detection workflow (Hare + Tortoise) from trio CRAM/BAM input) — https://github.com/TNTurnerLab/HAT
- **Hare** (GPU-accelerated workflow arm (Parabricks v4.0.0-1, GATK 4.2.0 HaplotypeCaller, DeepVariant v1.4.0) for WGS and WES data) — https://github.com/TNTurnerLab/Hare
- **Tortoise** (Open-source, CPU-only workflow arm for WGS, WES, and PacBio data using DeepVariant and GATK HaplotypeCaller) — https://github.com/TNTurnerLab/Tortoise
- **GLnexus** (Forms family-level joint-genotyped files passed to the custom de novo variant filter)
- **HAT-FLEX** (Caller-agnostic drop-in trio DNV detection directly on existing VCFs with allele-level intersection and sex/PAR-aware logic) — https://github.com/TNTurnerLab/HAT-FLEX
- **squirrel.py** (Converts HAT output to acorn input format) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package for followup statistics on HAT DNV output (extraction, CpG annotation, Ti/Tv, INDEL ratios)) — https://github.com/TNTurnerLab/acorn

## Examples

```
Rscript -e "library(acorn); dnv <- readDNV('hat_output_prepped.txt'); CpGresult <- annotateCpG(DNVobject=dnv, cpggr=load('cpg_b38.rda'))"
```

## Evaluation signals

- DNVs per individual falls in expected range (e.g., ~80 DNVs at 30x downsampled HG002, ~1105 called on the GIAB HG002 Illumina trio)
- Percent of DNVs at CpG sites matches expectations (~16.5-17.0% on HG002 Illumina and PacBio HiFi callsets)
- Percent of DNVs phased to the paternal chromosome of origin is consistent with published values (via unfazed parent-of-origin assignment)
- Confirmation rate against truthset or higher-coverage data is high (e.g., 96% of the 914 overlapping HG002 DNVs confirmed by 300x data; 91.9% confirmation at 30x PacBio HiFi)
- Every emitted DNV satisfies the Mendelian pattern: child heterozygous or homozygous alternate, both parents homozygous reference in the joint-genotyped VCF

## Limitations

- Hare v1.2 is effective for WGS and WES data only; PacBio HiFi data requires the Tortoise workflow
- Requires access to aligned BAM/CRAM files; HAT-FLEX is the alternative when only VCFs are available
- DNV confirmation rates degrade at lower coverage (e.g., 62.4% at 10x PacBio HiFi vs 91.9% at 30x)
- PacBio runs should use the DeepVariant PacBio/Revio model rather than the Illumina model

## Evidence

- [abstract] HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs.: "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."
- [intro] We then use GLnexus to form family level joint-genotyped files: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] This workflow is effective for WGS and WES data only.: [evidence span withheld — anchor status: not_found]
- [methods] HAT-FLEX is a caller-agnostic, drop-in trio DNV detection tool that operates directly on existing VCFs.: "HAT-FLEX is a caller-agnostic, drop-in trio DNV detection tool that operates directly on existing VCFs."
- [readme] There is a code called `squirrel.py` that can convert HAT output to acorn input: [evidence span withheld — anchor status: external_doc]

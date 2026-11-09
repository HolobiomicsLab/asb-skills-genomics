---
name: de-novo-variant-filtering
description: Use when you have trio variant calls (child + two parents) as joint-genotyped
  VCFs and need to extract DNVs present in the child but absent from both parents,
  with allele-level intersection and sex/PAR-aware logic.
when_to_use_negative:
- Input is still aligned reads (BAM/CRAM) with no variant calls — run the full Hare/Tortoise
  workflow including variant calling and GLnexus joint genotyping first, rather than
  filtering alone.
- You want to process single-sample VCFs or cohorts without trio structure — DNV detection
  requires child and both parents; HAT-FLEX operates on trio-level and multi-sample
  VCFs assuming trio relationships.
- You have PacBio HiFi long-read data — use the Tortoise branch (effective for WGS,
  WES and PacBio data), since Hare v1.2 is effective for WGS and WES data only.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0625
tools:
- name: HAT-FLEX
  role: caller-agnostic drop-in trio DNV detection tool operating directly on existing
    VCFs; performs the allele-level intersection, sex/PAR-aware logic, and clustering
    of the filter
  repo: https://github.com/TNTurnerLab/HAT-FLEX
- name: GLnexus
  role: produces the family-level joint-genotyped files that are fed into the custom
    de novo variant filter
- name: Hare
  role: GPU-accelerated (Parabricks) upstream variant calling whose joint-genotyped
    output the filter consumes; WGS/WES only
  repo: https://github.com/TNTurnerLab/Hare
- name: Tortoise
  role: open-source CPU variant-calling branch whose joint-genotyped output the filter
    consumes; supports WGS, WES and PacBio data
  repo: https://github.com/TNTurnerLab/Tortoise
- name: squirrel.py
  role: converts HAT output to acorn input for downstream DNV summarization (CpG fraction,
    variant classes)
  repo: https://github.com/TNTurnerLab/HAT
- name: acorn
  role: R package for follow-up statistics on filtered DNV output, e.g., annotateCpG
    for percent of DNVs at CpG sites
  repo: https://github.com/TNTurnerLab/acorn
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:e5b10adf3941edf9ace491063d7b160d7a82dc9b8b84cc61fff323159831c49f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT-FLEX
  tools:
  - DeepVariant
  - GATK
  - GLnexus
  - HAT
  - HAT-FLEX
  - Hare
  - Parabricks
  - Tortoise
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# de-novo-variant-filtering

## Summary

Extract candidate de novo variants (DNVs) by running family-level joint-genotyped VCFs (produced with GLnexus from Hare/Tortoise variant calls) through HAT's custom de novo variant filter, or apply HAT-FLEX directly to existing VCFs when BAM/CRAM files are not available.

## When to use

Use when you have trio variant calls (child + two parents) as joint-genotyped VCFs and need to extract DNVs present in the child but absent from both parents, with allele-level intersection and sex/PAR-aware logic.

## When NOT to use

- Input is still aligned reads (BAM/CRAM) with no variant calls — run the full Hare/Tortoise workflow including variant calling and GLnexus joint genotyping first, rather than filtering alone.
- You want to process single-sample VCFs or cohorts without trio structure — DNV detection requires child and both parents; HAT-FLEX operates on trio-level and multi-sample VCFs assuming trio relationships.
- You have PacBio HiFi long-read data — use the Tortoise branch (effective for WGS, WES and PacBio data), since Hare v1.2 is effective for WGS and WES data only.

## Inputs

- family-level joint-genotyped VCFs from GLnexus (child, mother, father)
- existing trio or multi-sample VCFs (for HAT-FLEX)
- aligned CRAM/BAM files from a sequenced parent-child trio (upstream of filtering)

## Outputs

- per-child DNV call set (VCF or tidy per-child outputs with provenance/audit files)
- cohort-level DNV count tables (e.g., 329,589 DNVs across 4216 SSC trios)
- acorn-ready DNV file (sample, chromosome, position, ref, alt) prepped via squirrel.py

## How to apply

First call variants per family member (Hare with Parabricks v4.0.0-1 GATK 4.2.0 HaplotypeCaller + DeepVariant v1.4.0, or Tortoise with open-source CPU versions) and joint-genotype the family with GLnexus. Then pass the family-level joint-genotyped files through HAT's custom de novo variant filter to retain variants present in the offspring but absent from both parents. If you already hold VCFs but lack BAM/CRAM access, use HAT-FLEX, a caller-agnostic drop-in tool that operates directly on existing VCFs and applies allele-level intersection, sex/PAR-aware logic, and clustering with comprehensive audit outputs. The filter yields per-child DNV call sets (e.g., .vcf/tidy outputs) whose quality is judged by DNVs per individual, percent at CpG sites, and percent phased to the paternal chromosome of origin. Prep output with squirrel.py and summarize with the acorn R package.

## Related tools

- **HAT-FLEX** (caller-agnostic drop-in trio DNV detection tool operating directly on existing VCFs; performs the allele-level intersection, sex/PAR-aware logic, and clustering of the filter) — https://github.com/TNTurnerLab/HAT-FLEX
- **GLnexus** (produces the family-level joint-genotyped files that are fed into the custom de novo variant filter)
- **Hare** (GPU-accelerated (Parabricks) upstream variant calling whose joint-genotyped output the filter consumes; WGS/WES only) — https://github.com/TNTurnerLab/Hare
- **Tortoise** (open-source CPU variant-calling branch whose joint-genotyped output the filter consumes; supports WGS, WES and PacBio data) — https://github.com/TNTurnerLab/Tortoise
- **squirrel.py** (converts HAT output to acorn input for downstream DNV summarization (CpG fraction, variant classes)) — https://github.com/TNTurnerLab/HAT
- **acorn** (R package for follow-up statistics on filtered DNV output, e.g., annotateCpG for percent of DNVs at CpG sites) — https://github.com/TNTurnerLab/acorn

## Evaluation signals

- Filtered DNV counts reproduce paper values: 1105 DNVs on downsampled ~30x GIAB HG002 Illumina WGS; 1108 and 1106 DNVs on the two PacBio HiFi replicates; 329,589 DNVs across 4216 SSC short-read trios.
- CpG fraction of filtered DNVs lands near reported values (~16.5% for Illumina 30x; ~16.6% and ~17.0% for the PacBio replicates).
- Every retained DNV appears in the child but is absent from both parents (allele-level intersection with sex/PAR-aware logic succeeds).
- QC metrics such as DNVs per individual and percent phased to the paternal chromosome of origin are consistent with published values.
- HAT-FLEX runs emit per-child outputs with full provenance and comprehensive audit outputs.

## Limitations

- Hare's upstream calling requires NVIDIA Parabricks v4.0.0-1 with GPUs and is effective for WGS and WES data only; PacBio HiFi data must go through Tortoise.
- HAT-FLEX is a beta tool and assumes you already have VCFs; it is useful if you do not have access to BAM/CRAM files but cannot itself call variants from reads.
- The custom de novo variant filter expects family-level joint-genotyped files from GLnexus, not independently called per-sample VCFs (use HAT-FLEX for that mode).

## Evidence

- [intro] GLnexus joint genotyping feeding the custom filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] HAT-FLEX filter capabilities: "It introduces allele-level intersection, sex/PAR-aware logic, clustering, comprehensive audit outputs, and streamlined operations."
- [methods] HAT-FLEX VCF-only mode: "HAT-FLEX is a caller-agnostic, drop-in trio DNV detection tool that operates directly on existing VCFs."
- [readme] Tortoise trio input and DNV output format: [evidence span withheld — anchor status: external_doc]

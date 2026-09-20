---
name: joint-genotyped-vcf-parsing
description: Use when input is a trio VCF from GLnexus joint genotyping (child + both
  parents) and the goal is extracting Mendelian de novo variants via HAT-style filtering.
when_to_use_negative:
- Input is raw CRAM/BAM alignment data that has not been variant-called yet — use
  the Hare or Tortoise variant-calling workflow first (Tortoise expects .cram or .bam
  files for a family trio).
- Input is a single-sample VCF with no trio genotypes for the child's parents — the
  Mendelian de novo pattern (child 0/1 or 1/1, parents 0/0) cannot be evaluated.
- Data falls outside the caller's scope, e.g. Hare v1.2 is effective for WGS and WES
  data only and will not handle PacBio HiFi long-read WGS (use Tortoise for WGS, WES
  and PacBio data).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: GLnexus
  role: Produces the family-level joint-genotyped trio VCFs that are the input to
    the custom de novo variant filter
- name: Hare v1.2
  role: GPU-accelerated (Parabricks) variant-calling pipeline whose joint-genotyped
    output feeds the filter; effective for WGS and WES data only
  repo: https://github.com/TNTurnerLab/Hare
- name: Tortoise
  role: Open-source CPU variant-calling pipeline (DeepVariant + GATK HaplotypeCaller
    + GLnexus) whose joint-genotyped output feeds the filter; works for WGS, WES and
    PacBio data
  repo: https://github.com/TNTurnerLab/Tortoise
- name: HAT-FLEX
  role: Caller-agnostic, drop-in implementation of the trio DNV filtering logic that
    operates directly on existing VCFs with allele-level intersection and sex/PAR-aware
    logic
  repo: https://github.com/TNTurnerLab/HAT-FLEX
- name: HAT
  role: Umbrella workflow containing the custom de novo variant filter and squirrel.py
    for converting HAT output to acorn input
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:1dbd185e40e19860094ced5543dfb0b800d018049097f8e6441b4b5b4380bf8e
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
  - Hare v1.2
  - Tortoise
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# joint-genotyped-vcf-parsing

## Summary

Parse GLnexus family-level joint-genotyped trio VCFs and apply HAT's custom de novo variant filter (eight-rule default DNV filtering) to produce a final DNV callset.

## When to use

Use when input is a trio VCF from GLnexus joint genotyping (child + both parents) and the goal is extracting Mendelian de novo variants via HAT-style filtering.

## When NOT to use

- Input is raw CRAM/BAM alignment data that has not been variant-called yet — use the Hare or Tortoise variant-calling workflow first (Tortoise expects .cram or .bam files for a family trio).
- Input is a single-sample VCF with no trio genotypes for the child's parents — the Mendelian de novo pattern (child 0/1 or 1/1, parents 0/0) cannot be evaluated.
- Data falls outside the caller's scope, e.g. Hare v1.2 is effective for WGS and WES data only and will not handle PacBio HiFi long-read WGS (use Tortoise for WGS, WES and PacBio data).

## Inputs

- GLnexus family-level joint-genotyped trio VCF (indexed, e.g. .vcf.gz + .tbi)
- GATK 4.2.0 HaplotypeCaller single-sample VCFs for allele-level intersection
- DeepVariant 1.4.0 single-sample VCFs for allele-level intersection
- BED-style blacklist intervals (repeats, low-complexity sequence, centromeres)

## Outputs

- dnv_filtered_calls.tsv — tidy per-child DNV table (chrom, pos, ref, alt, child GT/DP/GQ/AF, parental GT/DP) with provenance columns
- Per-rule filter audit records (failing records excluded from output)

## How to apply

Load the GLnexus family-level joint-genotyped trio VCF with pysam or cyvcf2, indexing it for random access. Enforce the Mendelian de novo pattern: keep records where the child genotype is 0/1 or 1/1 and both parents are 0/0, with zero alternate-allele reads in either parent. Apply the allele-level intersection rule requiring the alternate allele to be present in both GATK 4.2.0 HaplotypeCaller and DeepVariant 1.4.0 calls, using sex/PAR-aware logic for X/Y chromosome records. Enforce depth, quality, and allele-fraction thresholds: total read depth >=10x, child genotype quality (GQ) >=20, child alternate allele fraction >=25%. Exclude variants overlapping blacklist BED intervals (repeats, low-complexity sequence, centromeres), then emit survivors as a tidy per-child table in dnv_filtered_calls.tsv with full provenance columns.

## Related tools

- **GLnexus** (Produces the family-level joint-genotyped trio VCFs that are the input to the custom de novo variant filter)
- **Hare v1.2** (GPU-accelerated (Parabricks) variant-calling pipeline whose joint-genotyped output feeds the filter; effective for WGS and WES data only) — https://github.com/TNTurnerLab/Hare
- **Tortoise** (Open-source CPU variant-calling pipeline (DeepVariant + GATK HaplotypeCaller + GLnexus) whose joint-genotyped output feeds the filter; works for WGS, WES and PacBio data) — https://github.com/TNTurnerLab/Tortoise
- **HAT-FLEX** (Caller-agnostic, drop-in implementation of the trio DNV filtering logic that operates directly on existing VCFs with allele-level intersection and sex/PAR-aware logic) — https://github.com/TNTurnerLab/HAT-FLEX
- **HAT** (Umbrella workflow containing the custom de novo variant filter and squirrel.py for converting HAT output to acorn input) — https://github.com/TNTurnerLab/HAT

## Examples

```
import pysam; vcf = pysam.VariantFile('trio.glnexus.joint.vcf.gz'); keep = [r for r in vcf.fetch() if r.samples['child']['GT'] in [(0,1),(1,1)] and all(r.samples[p]['GT']==(0,0) for p in ['father','mother'])]
```

## Evaluation signals

- Script-check that every emitted record satisfies child GT in {0/1, 1/1}, both parents 0/0, and zero parental alternate-allele reads.
- Verify dual-caller allele-level intersection: every output DNV's alternate allele is present in both GATK 4.2.0 and DeepVariant 1.4.0 calls.
- Confirm thresholds on every emitted record: total read depth >=10x, child GQ >=20, child alternate allele fraction >=25%.
- Confirm no emitted record overlaps blacklist BED intervals (repeats, low-complexity, centromeres), with sex/PAR-aware handling of X/Y records.
- Check records failing any rule are absent from dnv_filtered_calls.tsv and that the output schema is the tidy per-child table with provenance columns.

## Limitations

- Hare v1.2 is effective for WGS and WES data only; PacBio HiFi long-read WGS requires the Tortoise workflow.
- Blacklisted regions (repeats, low-complexity sequence, centromeres) are excluded by interval overlap, so DNVs inside those intervals are never reported.
- X/Y chromosome records require sex/PAR-aware handling; naive parsing of sex chromosomes or the pseudoautosomal region will misclassify genotypes.
- The classic pipeline needs BAM/CRAM access and dual GATK + DeepVariant calls; if only existing VCFs are available, HAT-FLEX is the drop-in alternative.

## Evidence

- [intro] GLnexus produces the joint-genotyped input the filter consumes: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [methods] HAT-FLEX implements the filter logic directly on VCFs: "HAT-FLEX is a caller-agnostic, drop-in trio DNV detection tool that operates directly on existing VCFs."
- [methods] Filter features include allele-level intersection and sex/PAR-aware logic: "It introduces allele-level intersection, sex/PAR-aware logic, clustering, comprehensive audit outputs, and streamlined operations."
- [abstract] Whole workflow starts from CRAM/BAM trio data: "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."
- [readme] Tortoise input/output format for the CPU pipeline: [evidence span withheld — anchor status: external_doc]
- [methods] Hare v1.2 scope restriction: "This workflow is effective for WGS and WES data only."

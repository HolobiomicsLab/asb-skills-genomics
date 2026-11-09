---
name: genotype-quality-threshold-filtering
description: Use when you have a GLnexus joint-genotyped trio VCF (GATK HaplotypeCaller
  and/or DeepVariant calls) and need to enforce Mendelian de novo pattern plus depth,
  genotype quality, and allele-fraction cutoffs to produce a final DNV callset.
when_to_use_negative:
- You only have existing VCFs and no BAM/CRAM files and need a drop-in tool — HAT-FLEX
  is the caller-agnostic alternative
- Your input is already a called DNV table needing downstream statistics/annotation
  — use the acorn R package (after squirrel.py prep) instead
- You need the variant-calling stage itself (Parabricks/GATK/DeepVariant/Glnexus joint
  genotyping) rather than post-genotyping filtering
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0625
tools:
- name: HAT (Hare And Tortoise)
  role: Contains the custom de novo variant filter that applies these genotype-quality
    thresholds to GLnexus joint-genotyped trio files
  repo: https://github.com/TNTurnerLab/HAT
- name: GLnexus
  role: Produces the family-level joint-genotyped VCFs that are the filter's input
- name: GATK 4.2.0 HaplotypeCaller
  role: One of the two callers whose allele-level intersection the filter requires
- name: DeepVariant 1.4.0
  role: Second caller required for the allele-level intersection rule
- name: HAT-FLEX
  role: Caller-agnostic drop-in alternative for applying the DNV filter directly on
    existing VCFs
  repo: https://github.com/TNTurnerLab/HAT-FLEX
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:161cb347643957170cccd54917abbc2c981de40744ce255d9912266b42b49e9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/HAT
  tools:
  - DeepVariant
  - DeepVariant 1.4.0
  - GATK
  - GATK 4.2.0 HaplotypeCaller
  - GLnexus
  - HAT
  - HAT (Hare And Tortoise)
  - HAT-FLEX
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# genotype-quality-threshold-filtering

## Summary

Apply HAT's genotype- and depth-based quality thresholds to GLnexus family-level joint-genotyped trio VCFs so only high-confidence de novo variants (DNVs) survive. This is the core custom de novo variant filter step of the HAT (Hare And Tortoise) pipeline.

## When to use

Use when you have a GLnexus joint-genotyped trio VCF (GATK HaplotypeCaller and/or DeepVariant calls) and need to enforce Mendelian de novo pattern plus depth, genotype quality, and allele-fraction cutoffs to produce a final DNV callset.

## When NOT to use

- You only have existing VCFs and no BAM/CRAM files and need a drop-in tool — HAT-FLEX is the caller-agnostic alternative
- Your input is already a called DNV table needing downstream statistics/annotation — use the acorn R package (after squirrel.py prep) instead
- You need the variant-calling stage itself (Parabricks/GATK/DeepVariant/Glnexus joint genotyping) rather than post-genotyping filtering

## Inputs

- GLnexus family-level joint-genotyped trio VCF (.vcf/.vcf.gz with index)
- GATK 4.2.0 HaplotypeCaller per-sample calls
- DeepVariant 1.4.0 per-sample calls
- BED-style blacklist intervals (repeats, low-complexity sequence, centromeres)

## Outputs

- dnv_filtered_calls.tsv — tidy per-child DNV table with chrom, pos, ref, alt, child GT/DP/GQ/AF, parental GT/DP, and full provenance columns

## How to apply

Load the GLnexus family-level joint-genotyped trio VCF with pysam or cyvcf2 (indexed for random access) and enforce the Mendelian de novo pattern: child genotype must be 0/1 or 1/1 while both parents are 0/0. Then apply the article's quantitative cutoffs: total read depth >=10x, child genotype quality (GQ) >=20, child alternate allele fraction >=25%, and zero alternate-allele reads in either parent. Require the alternate allele to be present in both GATK 4.2.0 HaplotypeCaller and DeepVariant 1.4.0 calls at the allele level, with sex/PAR-aware handling of X/Y records, and exclude variants overlapping blacklist intervals (repeats, low-complexity sequence, centromeres). Emit surviving records as a tidy per-child table (chrom, pos, ref, alt, child GT/DP/GQ/AF, parental GT/DP) with provenance columns. The rationale is that requiring concordance between two independent callers plus strict child-carrier/parent-reference genotype constraints removes artifacts that pass any single-caller QC check.

## Related tools

- **HAT (Hare And Tortoise)** (Contains the custom de novo variant filter that applies these genotype-quality thresholds to GLnexus joint-genotyped trio files) — https://github.com/TNTurnerLab/HAT
- **GLnexus** (Produces the family-level joint-genotyped VCFs that are the filter's input)
- **GATK 4.2.0 HaplotypeCaller** (One of the two callers whose allele-level intersection the filter requires)
- **DeepVariant 1.4.0** (Second caller required for the allele-level intersection rule)
- **HAT-FLEX** (Caller-agnostic drop-in alternative for applying the DNV filter directly on existing VCFs) — https://github.com/TNTurnerLab/HAT-FLEX

## Examples

```
from cyvcf2 import VCF; for v in VCF('trio_joint_genotyped.vcf.gz'): check child_gt in (0/1,1/1), parents 0/0, DP>=10, GQ>=20, AF>=0.25, parental_alt==0
```

## Evaluation signals

- Every emitted record satisfies each rule: child GT is 0/1 or 1/1, both parents 0/0, DP >=10x, child GQ >=20, child AF >=25%, zero parental alternate reads, and no blacklist overlap
- Records failing any single rule are verified absent from the output (script-check of the inverse set)
- Alternate allele is present at the allele level in both GATK and DeepVariant calls for every kept variant
- Output table schema is complete: chrom, pos, ref, alt, child GT/DP/GQ/AF, parental GT/DP, plus provenance columns
- X/Y chromosome records are handled with sex/PAR-aware logic (no false de novo calls in PAR or sex-chromosome regions)

## Limitations

- Thresholds and rules are calibrated for short-read WGS/WES (Hare) and WGS/WES/PacBio HiFi (Tortoise) trios; validated on Illumina short-read and highly accurate PacBio HiFi data
- Blacklist exclusion (repeats, low-complexity, centromeres) removes variants in those regions regardless of quality, so true DNVs there are lost
- Requires a sequenced parent-child trio; cannot be applied to duos or singletons without modification
- The zero-parental-alt-read and dual-caller intersection rules are strict and can sacrifice sensitivity to maximize precision

## Evidence

- [intro] GLnexus joint genotyping feeds the custom filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [readme] HAT workflow scope (readme, same claim): "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter."
- [methods] Filter sub-features include allele-level intersection and sex/PAR-aware logic: "It introduces allele-level intersection, sex/PAR-aware logic, clustering, comprehensive audit outputs, and streamlined operations."
- [readme] Input data type for the trio pipeline: [evidence span withheld — anchor status: external_doc]
- [readme] Caller-agnostic alternative when thresholds tool does not fit: "HAT-FLEX is a caller-agnostic, drop-in trio DNV detection tool that operates directly on existing VCFs."

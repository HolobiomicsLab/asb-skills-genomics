---
name: repeat-region-blacklist-masking
description: Use when a trio DNV candidate list (e.g., from GLnexus joint-genotyped
  VCFs passing Mendelian, depth, and dual-caller filters) must be pruned of variants
  falling in repeats, low-complexity sequence, or centromeres before emitting final
  calls.
when_to_use_negative:
- Input is raw aligned reads (BAM/CRAM) with no joint-genotyped trio VCF — use the
  Hare or Tortoise variant-calling workflow first
- You only have existing multi-sample VCFs without BAM/CRAM access and want drop-in
  trio DNV detection — HAT-FLEX is the caller-agnostic alternative
- The reference region set already excludes repeats/centromeres (e.g., exome targets
  restricted to high-confidence regions), making a second blacklist pass redundant
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: GLnexus
  role: Produces the family-level joint-genotyped trio VCF that is filtered, including
    the blacklist-overlap rule
  repo: https://github.com/TNTurnerLab/HAT
- name: GATK 4.2.0 HaplotypeCaller
  role: One of the two caller outputs whose allele-level intersection must pass (alongside
    DeepVariant) before blacklist masking is applied
  repo: https://github.com/TNTurnerLab/Hare
- name: DeepVariant 1.4.0
  role: Second caller output required in the allele-level intersection before the
    blacklist-exclusion rule is enforced
  repo: https://github.com/TNTurnerLab/Tortoise
- name: HAT
  role: The custom _de novo_ variant filter workflow in which blacklist exclusion
    of repeats, low-complexity sequence, and centromeres is implemented
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:e7325edbaf8cd2d310907f79c0d19d0216263ed1a4338dcf416910f727282bac
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Repeat-region blacklist masking

## Summary

Excludes candidate de novo variants that overlap a BED-style blacklist of repeat, low-complexity, and centromeric regions, a step in HAT's custom _de novo_ variant filter applied after GLnexus family-level joint genotyping. Masking these error-prone intervals removes false DNVs driven by aligner/variant-caller artifacts.

## When to use

Use when a trio DNV candidate list (e.g., from GLnexus joint-genotyped VCFs passing Mendelian, depth, and dual-caller filters) must be pruned of variants falling in repeats, low-complexity sequence, or centromeres before emitting final calls.

## When NOT to use

- Input is raw aligned reads (BAM/CRAM) with no joint-genotyped trio VCF — use the Hare or Tortoise variant-calling workflow first
- You only have existing multi-sample VCFs without BAM/CRAM access and want drop-in trio DNV detection — HAT-FLEX is the caller-agnostic alternative
- The reference region set already excludes repeats/centromeres (e.g., exome targets restricted to high-confidence regions), making a second blacklist pass redundant

## Inputs

- Family-level joint-genotyped trio VCF from GLnexus (indexed for random access via pysam/cyvcf2)
- Candidate DNV records already passing Mendelian (child 0/1 or 1/1, parents 0/0), dual-caller allele-level intersection, DP >=10x, child GQ >=20, child AF >=25%, and zero-parental-alt-read filters
- BED-style blacklist of repeat, low-complexity, and centromeric regions for the reference genome

## Outputs

- Filtered DNV callset written as a tidy per-child table (chrom, pos, ref, alt, child GT/DP/GQ/AF, parental GT/DP) in dnv_filtered_calls.tsv with full provenance columns
- Blacklist-overlapping variants excluded (optionally auditable via provenance columns)

## How to apply

Load the blacklist as a BED-style interval file containing repeats, low-complexity sequence, and centromere annotations for the reference genome. For each candidate DNV (child 0/1 or 1/1, parents 0/0, dual-caller allele-level intersection between GATK 4.2.0 HaplotypeCaller and DeepVariant 1.4.0, DP >=10x, child GQ >=20, child AF >=25%, zero parental alt reads), test interval overlap between the variant's (chrom, pos) coordinate and blacklist intervals, e.g. with pysam/cyvcf2 indexed VCFs plus a bedtools/pybedtools-style intersect in inversion mode. Drop any variant with nonzero overlap and emit only the survivors to the tidy per-child table dnv_filtered_calls.tsv with full provenance columns. The rationale is that repeats, low-complexity sequence, and centromeres are mappable/artifact-prone regions where apparent Mendelian de novo patterns are frequently spurious.

## Related tools

- **GLnexus** (Produces the family-level joint-genotyped trio VCF that is filtered, including the blacklist-overlap rule) — https://github.com/TNTurnerLab/HAT
- **GATK 4.2.0 HaplotypeCaller** (One of the two caller outputs whose allele-level intersection must pass (alongside DeepVariant) before blacklist masking is applied) — https://github.com/TNTurnerLab/Hare
- **DeepVariant 1.4.0** (Second caller output required in the allele-level intersection before the blacklist-exclusion rule is enforced) — https://github.com/TNTurnerLab/Tortoise
- **HAT** (The custom _de novo_ variant filter workflow in which blacklist exclusion of repeats, low-complexity sequence, and centromeres is implemented) — https://github.com/TNTurnerLab/HAT

## Examples

```
pybedtools.BedTool(dnv_records).intersect(blacklist_bed, v).saveas('dnv_filtered_calls.tsv')
```

## Evaluation signals

- Every record in dnv_filtered_calls.tsv has zero interval overlap with the repeat/low-complexity/centromere blacklist BED
- Re-running the intersect in inversion (keep-only-non-overlapping) mode yields exactly the emitted callset, with no dropped survivors
- All retained records also satisfy the other rules: child GT 0/1 or 1/1, parents 0/0, dual-caller allele-level presence, DP >=10x, child GQ >=20, child AF >=25%, zero parental alt reads
- Records that overlap a blacklist interval appear in the pipeline audit/provenance trail but are absent from the final output
- Output is a tidy per-child table with chrom, pos, ref, alt, child GT/DP/GQ/AF, and parental GT/DP columns

## Limitations

- Blacklist masking depends on reference-matched interval annotations; mis-specified or wrong-genome-build BED files will wrongly drop or retain variants
- Interval overlap alone can discard true DNVs in repeats and centromeres, reducing sensitivity in those regions
- Hare is effective for WGS and WES data only; Tortoise covers WGS, WES, and PacBio HiFi, so long-read blacklist handling should follow the Tortoise workflow

## Evidence

- [other] Enforce Mendelian de novo pattern and blacklist rule: [evidence span withheld — anchor status: not_found]
- [other] Filter thresholds and intersection rule: [evidence span withheld — anchor status: not_found]
- [intro] GLnexus joint genotyping feeding custom filter: "We then use GLnexus to form family level joint-genotyped files to be run through our custom _de novo_ variant filter"
- [readme] HAT-FLEX audit outputs context: "It introduces allele-level intersection, sex/PAR-aware logic, clustering, comprehensive audit outputs, and streamlined operations."

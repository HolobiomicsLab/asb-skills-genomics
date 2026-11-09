---
name: structural-variant-callset-matching
description: Use when you have a VCF of predicted non-reference sequence insertions
  and need precision/recall against a PacBio validation callset, or a confirmation
  rate against a T2T-assembly-derived insertion benchmark.
when_to_use_negative:
- You need de novo comparison of two assemblies rather than matching a callset to
  a trusted benchmark — this skill presumes one side is treated as ground truth
- 'Insertions are shorter than 300 bp: Novel-X output filters and the paper''s analysis
  are restricted to insertions longer than 300 bp'
- Only raw read alignments exist and no benchmark insertion set (PacBio callset or
  T2T assembly) can be constructed for the sample
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3673
tools:
- name: Novel-X
  role: Source of predicted NRS insertion VCFs being benchmarked
  repo: github.com/1dayac/Novel-X
- name: SuperNova
  role: Whole-genome diploid assembler whose output, via Paftools, provides a competing
    insertion callset for NA19240 precision/recall comparison
- name: Paftools
  role: Extracts novel insertion calls from the SuperNova assembly-to-reference alignment
- name: Minimap2
  role: Aligns the T2T-CHM13 v2.0 assembly to GRCh38 to build the benchmark insertion
    set
- name: LongRanger
  role: Produces LongRanger-compatible BAMs of synthetic long reads that feed Novel-X
    and can be used in the Novel-X pipeline directly
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:5eed0ac1a198e36ab0b5b940b270e1cde2c1b4d50f255442239c6483cd824c18
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Minimap2
  - Novel-X
  - Paftools
  - SuperNova
  - stlfr2supernova
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# structural-variant-callset-matching

## Summary

Benchmark a predicted insertion/SV callset (e.g. Novel-X VCFs) against an independent validation set (PacBio long-read calls or a T2T assembly-derived insertion set) using paired breakpoint/length tolerance matching, then compute precision, recall, or confirmation rate.

## When to use

Use when you have a VCF of predicted non-reference sequence insertions and need precision/recall against a PacBio validation callset, or a confirmation rate against a T2T-assembly-derived insertion benchmark.

## When NOT to use

- You need de novo comparison of two assemblies rather than matching a callset to a trusted benchmark — this skill presumes one side is treated as ground truth
- Insertions are shorter than 300 bp: Novel-X output filters and the paper's analysis are restricted to insertions longer than 300 bp
- Only raw read alignments exist and no benchmark insertion set (PacBio callset or T2T assembly) can be constructed for the sample

## Inputs

- Predicted NRS insertion callset as VCF (Novel-X output: insertions > 300 bp with anchor lengths summing over 300 bp)
- PacBio long-read validation insertion callset (e.g. published NA19240 callsets)
- Assembly-to-reference alignment (e.g. T2T-CHM13 v2.0 vs GRCh38 via Minimap2; SuperNova assembly vs GRCh38 via Paftools) from which benchmark insertions are extracted

## Outputs

- Match/mismatch counts per tool or platform
- Precision and recall values (e.g. benchmark_table.csv comparing Novel-X vs SuperNova/Paftools)
- Confirmation rate per platform (e.g. chm13_t2t_validation_summary.csv)

## How to apply

Build the benchmark set from the trusted source: for NA19240 use published PacBio long-read validation callsets; for CHM13 align the T2T-CHM13 v2.0 assembly to GRCh38 with Minimap2 and extract long insertions from that alignment. Match each predicted call to a benchmark call when breakpoint positions differ by no more than 100 bp AND insertion lengths differ by no more than 5%. From match counts compute precision, recall (Novel-X vs SuperNova/Paftools on NA19240-10x: 59%/37.0% vs 56%/37.4%), or the fraction of confirmed calls (CHM13: ~78%). Write results as a numeric table (e.g. benchmark_table.csv or chm13_t2t_validation_summary.csv) with counts and rates per tool/platform.

## Related tools

- **Novel-X** (Source of predicted NRS insertion VCFs being benchmarked) — github.com/1dayac/Novel-X
- **SuperNova** (Whole-genome diploid assembler whose output, via Paftools, provides a competing insertion callset for NA19240 precision/recall comparison)
- **Paftools** (Extracts novel insertion calls from the SuperNova assembly-to-reference alignment)
- **Minimap2** (Aligns the T2T-CHM13 v2.0 assembly to GRCh38 to build the benchmark insertion set)
- **LongRanger** (Produces LongRanger-compatible BAMs of synthetic long reads that feed Novel-X and can be used in the Novel-X pipeline directly)

## Examples

```
minimap2 -c GRCh38.fa chm13v2.0_noY.fa.gz > t2t_vs_grch38.paf  # extract long insertions from this alignment as the benchmark set
```

## Evaluation signals

- Match counting uses both tolerances simultaneously: breakpoint difference ≤ 100 bp and insertion length difference ≤ 5%
- NA19240-10x metrics should approximate Novel-X precision 59% vs SuperNova/Paftools 56%, and recall 37.0% vs 37.4%
- CHM13 confirmation rate should be approximately 78% of Novel-X calls confirmed against T2T-derived insertions
- Input callsets should obey the output filter: only insertions longer than 300 bp with sum of anchor lengths exceeding 300 bp
- Results written as a numeric summary table (benchmark_table.csv / chm13_t2t_validation_summary.csv) with per-platform counts and rates

## Limitations

- Matching tolerances (100 bp breakpoint, 5% length) are coarse; a match is not a base-perfect validation
- Small insertions are underpowered: Novel-X needs a substantial number of distinct barcodes, and for small insertions coverage is not sufficient for anchor assembly, so callsets skew toward large events
- SuperNova offers no user control over assembler parameters (defaults tuned for whole human genome diploid assembly), which constrains how the competing benchmark callset can be optimized

## Evidence

- [other] Match each tool's calls against the corresponding published PacBio long-read validation callset, counting a match when breakpoint positions differ by no more than 100 bp and insertion lengths differ by no more than 5%.: [evidence span withheld — anchor status: not_found]
- [other] Novel-X achieved a slightly better precision (59% versus 56%) but a slightly lower recall (37.0% versus 37.4%) than SuperNova/Paftools.: [evidence span withheld — anchor status: not_found]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [discussion] Novel-X needs a substantial number of distinct barcodes to be associated with it. For small insertions, the number of associated barcodes is usually small and the coverage is not sufficient: "Novel-X needs a substantial number of distinct barcodes to be associated with it. For small insertions, the number of associated barcodes is usually small and the coverage is not sufficient"
- [supplementary] The CHM13 T2T assembly v2.0. used for CHM13 calls validation can be found at https://github.com/nanopore-wgs-consortium/chm13.: "The CHM13 T2T assembly v2.0. used for CHM13 calls validation can be found at https://github.com/nanopore-wgs-consortium/chm13."

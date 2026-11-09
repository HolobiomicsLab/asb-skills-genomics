---
name: copy-number-variant-type-classification
description: Use when a merged nanopore SV call set (VCF per aligner) must be split
  into SV-type classes and only the copy-number classes (DEL/DUP) need coverage-support
  testing and filtering.
when_to_use_negative:
- Input contains only non-copy-number SV types (insertions, inversions, translocations)
  — the article states coverage analysis 'can only be applied to some types, namely
  deletions and duplications'
- No prior SV calls exist — disCoverage is not a variant caller and needs putative
  SV positions from another technique or multi-sample analysis
- Variants are small (<50 kb) point-level differences rather than large CNVs — this
  skill targets large (>50 kb) deletions and duplications
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: disCoverage
  role: Checks whether coverage in a putative SV region agrees with the DEL/DUP type
    and labels support using a 40% coverage-shift threshold with Welch's t-test
  repo: https://github.com/javiercguard/disCoverage
- name: mosdepth
  role: Computes per-base depth over SV coordinates plus 1-Mb flanking regions for
    the coverage-shift calculation
  repo: https://github.com/brentp/mosdepth
- name: SURVIVOR
  role: Merges per-caller SV calls into one VCF per aligner whose records are then
    type-classified
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:87c37001d717baded4d3f8271b44a4d7631de61ece4f1df76a0d52b3a8a29581
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/disCoverage
  repo_url: https://github.com/javiercguard/disCoverage
  tools:
  - CuteSV
  - NGMLR
  - NanoVar
  - SURVIVOR
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  - mosdepth
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# copy-number-variant-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies merged nanopore SV calls into types (deletions, duplications, insertions, inversions, translocations) and applies type-specific downstream handling: only DEL and DUP classes can be validated against per-base coverage with disCoverage, since copy-number changes are the only types that imply coverage shifts.

## When to use

Use when a merged nanopore SV call set (VCF per aligner) must be split into SV-type classes and only the copy-number classes (DEL/DUP) need coverage-support testing and filtering.

## When NOT to use

- Input contains only non-copy-number SV types (insertions, inversions, translocations) — the article states coverage analysis 'can only be applied to some types, namely deletions and duplications'
- No prior SV calls exist — disCoverage is not a variant caller and needs putative SV positions from another technique or multi-sample analysis
- Variants are small (<50 kb) point-level differences rather than large CNVs — this skill targets large (>50 kb) deletions and duplications

## Inputs

- SURVIVOR-merged SV VCF per aligner (lra, minimap2, NGMLR) from CuteSV/Sniffles/SVIM/NanoVar calls
- 4-column BED of putative DEL/DUP coordinates for disCoverage
- NA19240 PromethION nanopore BAM aligned to hg38 (input to mosdepth)
- hg38 human reference genome

## Outputs

- sv_support_records.tsv with one supported/discarded label per input DEL/DUP
- Per-aligner summary table of total SV counts and deletions/duplications/insertions type distribution
- Filtered merged VCF with unsupported DEL/DUP and >10 Mb artifact SVs removed

## How to apply

Take the SURVIVOR-merged VCF produced per aligner (lra, minimap2, NGMLR against hg38) and label each record by SV type; expect a distribution such as 68.8%/66.0%/64.0% deletions, 0.3%/4.9%/6.1% duplications, and 29.9%/24.1%/24.9% insertions (plus inversions and translocations) for lra/minimap2/NGMLR on NA19240. Before classification, discard very large SVs (>10 Mb) that encompass other SVs or centromeres as artifacts. Convert the DEL and DUP entries to a 4-column BED and pass them to disCoverage, which runs mosdepth on the BAM to extract per-base coverage over each SV's coordinates extended by 1-Mb flanks and computes the relative coverage shift between the SV interval and its surroundings. Label a DEL/DUP as supported only when the coverage shift is >=40% and Welch's two-tailed t-test (Student's approximation in R) yields p < 0.05; INS, INV, and TRA records bypass the coverage criterion entirely. Tabulate total counts and the per-type distribution per aligner for comparison against reference values (e.g. Table 3).

## Related tools

- **disCoverage** (Checks whether coverage in a putative SV region agrees with the DEL/DUP type and labels support using a 40% coverage-shift threshold with Welch's t-test) — https://github.com/javiercguard/disCoverage
- **mosdepth** (Computes per-base depth over SV coordinates plus 1-Mb flanking regions for the coverage-shift calculation) — https://github.com/brentp/mosdepth
- **SURVIVOR** (Merges per-caller SV calls into one VCF per aligner whose records are then type-classified)

## Examples

```
disCoverage -h
```

## Evaluation signals

- sv_support_records.tsv contains exactly one labelled row per input SV, and every supported event satisfies coverage shift >=40% and p < 0.05
- Non-DEL/DUP records (INS, INV, TRA) never receive a coverage-support label
- A merged VCF exists for each of the three aligners (.vcf) and the DEL/DUP/INS distribution matches reference values (e.g. 68.8/66.0/64.0% deletions for lra/minimap2/NGMLR)
- All remaining SVs are <=10 Mb and none encompass other SVs or centromeres
- Read-support filtering removed calls below threshold (e.g. the chr8 deletion in P7 class of under-supported events)

## Limitations

- Coverage is not uniform in a sample, and local peaks and valleys frequently occur without pointing to an SV; unresolved genome regions and unclear shift boundaries also constrain the test
- The p-value threshold of 0.05 was not confirmed as optimal; further studies are required
- disCoverage is not a variant caller — it requires putative SV positions obtained from variant callers, another technique, or multi-sample merging

## Evidence

- [readme] disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications).: "disCoverage is a software tool to refine calling for structural variants using coverage analysis (so it can only check deletions and duplications)."
- [discussion] This can only be applied to some types, namely deletions and duplications: "This can only be applied to some types, namely deletions and duplications"
- [abstract] We also used a custom method for the statistical analysis of the coverage value to refine the resulting datasets: "We also used a custom method for the statistical analysis of the coverage value to refine the resulting datasets"
- [discussion] we used a threshold for p-values smaller than 0.05: "we used a threshold for p-values smaller than 0.05"

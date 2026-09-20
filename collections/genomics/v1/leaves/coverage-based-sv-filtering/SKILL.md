---
name: coverage-based-sv-filtering
description: Use when filtering structural variant calls (VCF or 4-column BED of DEL/DUP
  events) from long-read nanopore variant callers by verifying that regional read
  depth, computed from an alignment BAM, supports the claimed copy-number change.
when_to_use_negative:
- The variants of interest are insertions, inversions, or translocations — disCoverage
  can only check deletions and duplications because it relies on copy-number changes
  in coverage
- You need de novo SV discovery — disCoverage is not a variant caller and requires
  putative SV positions as input from another technique or multi-sample analysis
- Regions have highly non-uniform coverage with local peaks/valleys or unresolved
  genome regions where coverage shifts are ambiguous
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_0622
tools:
- name: disCoverage
  role: Coverage-analysis tool that checks whether regional read depth agrees with
    each called DEL/DUP and filters unsupported calls using a p-value < 0.05 plus
    coverage-difference criterion
  repo: https://github.com/javiercguard/disCoverage
- name: Mosdepth
  role: Computes per-region/per-base depth from the nanopore BAM/CRAM, supplying the
    coverage values disCoverage statistically evaluates
  repo: https://github.com/brentp/mosdepth
- name: Sniffles
  role: Upstream variant caller whose DEL/DUP output is one of the call sets fed into
    disCoverage for coverage-based polishing
- name: CuteSV
  role: Upstream variant caller whose DEL/DUP output is filtered by coverage support
- name: SVIM
  role: Upstream variant caller whose DEL/DUP output is filtered by coverage support
- name: NanoVar
  role: Upstream variant caller whose DEL/DUP output is filtered by coverage support
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.jare.2022.10.012
    title: Improvement of large copy number variant detection by whole genome nanopore
      sequencing
  source_content_hash: sha256:79772b470d2541d562937ac52550302ceda42cacdcb8a5f9da0e58c094b577f6
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
  - Mosdepth
  - NGMLR
  - NanoVar
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 22
---

# coverage-based-sv-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Refine nanopore SV call sets by statistically checking read-depth support for each called deletion/duplication using disCoverage (built on Mosdepth coverage), removing calls whose regional coverage does not agree with the SV type. In the source work this improved specificity by up to 62% and sensitivity by 15% for large (>50 kb) CNVs.

## When to use

Use when filtering structural variant calls (VCF or 4-column BED of DEL/DUP events) from long-read nanopore variant callers by verifying that regional read depth, computed from an alignment BAM, supports the claimed copy-number change.

## When NOT to use

- The variants of interest are insertions, inversions, or translocations — disCoverage can only check deletions and duplications because it relies on copy-number changes in coverage
- You need de novo SV discovery — disCoverage is not a variant caller and requires putative SV positions as input from another technique or multi-sample analysis
- Regions have highly non-uniform coverage with local peaks/valleys or unresolved genome regions where coverage shifts are ambiguous

## Inputs

- VCF of structural variant calls from Sniffles/CuteSV/SVIM/NanoVar (converted to 4-column BED: chr, start, end, SV type)
- 4-column BED files of putative DEL/DUP positions (e.g. from aCGH or a truth set such as Chaisson et al. 2019)
- Nanopore alignment BAM/CRAM (e.g. NA19240 PromethION reads aligned to hg38 with lra/minimap2/NGMLR)
- Optionally, precomputed Mosdepth coverage files (reused via --skip-mosdepth with -cov)

## Outputs

- Filtered SV call set with coverage-supported deletions and duplications retained
- Per-region coverage-support assessments (supported/not supported) for each queried DEL/DUP
- Coverage-support labels for truth-set events usable in recall-by-SV-type tables (TSV)

## How to apply

Convert variant caller output (Sniffles, CuteSV, SVIM, NanoVar VCFs) into the 4-column BED format (chr, start, end, SV type) that disCoverage accepts, keeping only deletions and duplications since other SV types are ignored. Run Mosdepth on the nanopore BAM (e.g. NA19240 PromethION alignments against hg38) to obtain per-region coverage; on repeated runs, reuse existing coverage files via disCoverage's --skip-mosdepth option with -cov to save time. Apply disCoverage's coverage-support criterion: a p-value < 0.05 combined with the observed coverage difference must agree with the SV type (depletion for deletions, enrichment for duplications) for the call to be retained. When applying to a truth set instead of a call set, the same procedure marks which truth-set deletions/duplications have coverage support, enabling recall computation per aligner and SV type. The 0.05 p-value threshold was used throughout the study but has not been confirmed as optimal.

## Related tools

- **disCoverage** (Coverage-analysis tool that checks whether regional read depth agrees with each called DEL/DUP and filters unsupported calls using a p-value < 0.05 plus coverage-difference criterion) — https://github.com/javiercguard/disCoverage
- **Mosdepth** (Computes per-region/per-base depth from the nanopore BAM/CRAM, supplying the coverage values disCoverage statistically evaluates) — https://github.com/brentp/mosdepth
- **Sniffles** (Upstream variant caller whose DEL/DUP output is one of the call sets fed into disCoverage for coverage-based polishing)
- **CuteSV** (Upstream variant caller whose DEL/DUP output is filtered by coverage support)
- **SVIM** (Upstream variant caller whose DEL/DUP output is filtered by coverage support)
- **NanoVar** (Upstream variant caller whose DEL/DUP output is filtered by coverage support)

## Examples

```
disCoverage -cov sample.mosdepth.regions.bed.gz --skip-mosdepth variants.bed
```

## Evaluation signals

- Each retained deletion shows a statistically significant coverage reduction and each retained duplication a coverage increase at p-value < 0.05 under disCoverage's criterion
- When applied to the NA19240 BAM against the Chaisson 2019 truth set, deletion recall should fall in ~37–43% and duplication recall in ~3–17% across lra/minimap2/NGMLR (37.93/42.71/40.05% and 3.70/13.81/16.43%)
- Median coordinate differences between matched truth-set and called SVs should be ~12.5/13.0/14.0 bp for lra/minimap2/NGMLR
- disCoverage should improve specificity by up to 62% and sensitivity by up to 15% relative to unfiltered variant-caller output
- Bed-format inputs must be 4-column (chr, start, end, SV type); non-DEL/DUP types in the input are ignored by disCoverage

## Limitations

- Coverage-based filtering applies only to deletions and duplications, not all SV types
- Coverage is not uniform across a sample; local peaks and valleys frequently occur without pointing to an SV, and shift boundaries can be unclear
- The p-value threshold of 0.05 was not confirmed as optimal; further studies are required
- Very large SVs (>10 Mb) encompassing other SVs or centromeres should be discarded as artifacts before/alongside coverage filtering
- disCoverage requires putative SV positions as input; it cannot call variants de novo

## Evidence

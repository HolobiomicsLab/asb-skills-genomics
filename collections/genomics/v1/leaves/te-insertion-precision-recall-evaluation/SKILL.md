---
name: te-insertion-precision-recall-evaluation
description: Use when evaluating the accuracy of a long-read TE insertion callset
  — e.g. after merging cuteSV and Sniffles2 outputs — by computing precision and recall
  against a benchmark set and choosing between strict vs lax filtering criteria.
when_to_use_negative:
- No curated truth/benchmark set exists for the samples — precision and recall cannot
  be computed, so this skill is inapplicable
- Callsets come from a single caller with no strict/lax intersection/union distinction
  to evaluate
- The task is TE detection or annotation itself rather than accuracy assessment of
  existing callsets
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: cuteSV
  role: Long-read structural variant caller whose insertion calls form one side of
    the strict/lax caller agreement
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second long-read SV caller; intersection (strict) or union (lax) with cuteSV
    defines the filtered callsets
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: SURVIVOR
  role: Compares filtered callsets against the HGSVC truth set with a 100 bp distance
    margin to compute precision and recall
- name: RetroInspector
  role: Snakemake pipeline whose default settings (cuteSV + Sniffles2, ≥3 supporting
    reads) produce the callsets being benchmarked
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:c464fd2845e1ad977fff3fc23265490b3bc5e1a45cfc7871a7a94e99ad822371
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/tjiangHIT/cuteSV
  tools:
  - RetroInspector
  - SURVIVOR
  - Sniffles2
  - cuteSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# te-insertion-precision-recall-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks TE insertion callsets from merged cuteSV/Sniffles2 calls against a curated truth set using SURVIVOR, comparing strict (intersection) vs lax (union) caller-agreement criteria to quantify the precision–recall trade-off.

## When to use

Use when evaluating the accuracy of a long-read TE insertion callset — e.g. after merging cuteSV and Sniffles2 outputs — by computing precision and recall against a benchmark set and choosing between strict vs lax filtering criteria.

## When NOT to use

- No curated truth/benchmark set exists for the samples — precision and recall cannot be computed, so this skill is inapplicable
- Callsets come from a single caller with no strict/lax intersection/union distinction to evaluate
- The task is TE detection or annotation itself rather than accuracy assessment of existing callsets

## Inputs

- Per-sample insertion VCF callsets from cuteSV and Sniffles2 after intra-sample and inter-sample merging
- HGSVC TE insertion truth set (5,630 insertions across HG00733, HG00514, NA19240)
- User-set supporting-read threshold (default 3 reads)

## Outputs

- Strict (intersection) filtered callset with precision 80.90% for TE insertions (85.63% all insertions), recall 87.10% (16.34%), F1 83.89%
- Lax (union) filtered callset with higher recall and lower precision than strict
- SURVIVOR benchmark report (100 bp distance margin) with precision/recall per criterion

## How to apply

Load the per-sample insertion VCF callsets produced by cuteSV and Sniffles2 after intra-sample and inter-sample merging, then filter all records by the user-set read-evidence threshold (RetroInspector default and recommended: 3 supporting reads), repairing malformed records such as negative or zero coordinates on decoy/scaffold regions. Build the strict callset as the intersection of insertions reported by BOTH callers on a sample, requiring at least one report to surpass the read-evidence threshold; build the lax callset as the union of insertions from EITHER caller with the same threshold. Benchmark both callsets against the HGSVC TE insertion truth set (5,630 insertions across three samples: HG00733, HG00514, NA19240) using SURVIVOR with a 100 bp distance margin to compute precision and recall. The rationale: the lax criterion recovers more true insertions (higher recall) but admits more non-benchmark insertions (lower precision); in the paper the precision loss was lower than the recall gain, and the strict criterion sat at higher precision / lower recall. Choose the criterion based on whether downstream annotation tolerates false positives or missed insertions.

## Related tools

- **cuteSV** (Long-read structural variant caller whose insertion calls form one side of the strict/lax caller agreement) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second long-read SV caller; intersection (strict) or union (lax) with cuteSV defines the filtered callsets) — https://github.com/fritzsedlazeck/Sniffles
- **SURVIVOR** (Compares filtered callsets against the HGSVC truth set with a 100 bp distance margin to compute precision and recall)
- **RetroInspector** (Snakemake pipeline whose default settings (cuteSV + Sniffles2, ≥3 supporting reads) produce the callsets being benchmarked) — https://github.com/javiercguard/retroinspector

## Evaluation signals

- Strict callset reproduces precision of 80.90% for TE insertions (85.63% for all insertions) with 87.10% recall against the HGSVC truth set
- Strict criterion shows strictly higher precision at lower recall than the lax criterion on the same truth set
- SURVIVOR comparison uses a 100 bp distance margin consistently across criteria
- All variant records pass the ≥3 supporting-read threshold and malformed records (negative/0 coordinates on decoy/scaffold regions) are repaired
- Lax callset size exceeds strict callset size (union ⊇ intersection) with a modest precision loss relative to the recall gain

## Limitations

- Lax criterion admits insertions not present in the benchmark set, reducing precision relative to strict
- HGSVC truth-set data was basecalled with an older basecaller; sequences may be imperfect, particularly for homopolymers
- Truth set incompleteness can distort apparent precision (noted for PALMER's low precision, which 'could stem from an incomplete truth set')

## Evidence

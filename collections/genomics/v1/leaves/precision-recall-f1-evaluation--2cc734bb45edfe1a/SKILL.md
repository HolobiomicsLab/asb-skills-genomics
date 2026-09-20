---
name: precision-recall-f1-evaluation
description: Use when you have variant/insertion callsets (VCF) and a curated truth
  set (e.g., 5,630 HGSVC TE insertions) and need to quantify detection performance
  as precision/recall/F1, or compare tools head-to-head.
when_to_use_negative:
- You lack a curated truth set — precision and recall cannot be computed and only
  consistency checks across callsets are possible
- Insertion coordinates come from different reference assemblies or unaligned representations,
  making SURVIVOR's positional distance-margin matching invalid
- The goal is genotyping accuracy (e.g., concordance with HGSV2 genotypes) rather
  than insertion detection performance
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: Matches predicted insertions against the truth set with a configurable distance
    margin (100 bp strict, 500 bp permissive)
- name: RetroInspector
  role: Pipeline whose cuteSV+Sniffles2 strict callsets are evaluated; benchmark metrics
    computed from its outputs
  repo: https://github.com/javiercguard/retroinspector
- name: PALMER
  role: Comparative benchmark tool; its VCF-converted output scored against the same
    truth set (max F1 60.72%)
- name: GraffiTE
  role: Comparative benchmark tool scored against the same truth set (recall 96.18%,
    precision 32.33%, F1 48.40%)
- name: Sniffles2
  role: Variant caller whose intersection-based callsets feed the precision/recall
    computation
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: cuteSV
  role: Variant caller whose intersection-based callsets feed the precision/recall
    computation
  repo: https://github.com/tjiangHIT/cuteSV
provenance:
  source_task_ids:
  - task_002
  - task_007
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:74e8124fac1020fdb50f1f2b3fa33b6411b12d63945b946dffa6b49bcc068301
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - FamDB toolkit
  - GNU parallel
  - GraffiTE
  - PALMER
  - PanGenIE
  - RapidFuzz
  - RetroInspector
  - SAMtools
  - SURVIVOR
  - SVIM
  - Sniffles2
  - cuteSV
  - minimap2
  - pROC
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 20
---

# precision-recall-f1-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark TE insertion detection callsets by matching predictions against a truth set with SURVIVOR at 100 bp (and a permissive 500 bp) distance margin, then computing precision, recall, and F1-score per tool or caller combination, as done to evaluate RetroInspector against the HGSVC 5,630-insertion truth set.

## When to use

Use when you have variant/insertion callsets (VCF) and a curated truth set (e.g., 5,630 HGSVC TE insertions) and need to quantify detection performance as precision/recall/F1, or compare tools head-to-head.

## When NOT to use

- You lack a curated truth set — precision and recall cannot be computed and only consistency checks across callsets are possible
- Insertion coordinates come from different reference assemblies or unaligned representations, making SURVIVOR's positional distance-margin matching invalid
- The goal is genotyping accuracy (e.g., concordance with HGSV2 genotypes) rather than insertion detection performance

## Inputs

- Predicted insertion/structural-variant callsets in VCF format (e.g., from cuteSV, Sniffles2, SVIM, PALMER, GraffiTE)
- Curated truth set of TE insertions (HGSVC truth set with 5,630 insertions for HG00733, HG00514, NA19240)
- SURVIVOR compare parameters (distance margin, e.g., 100 bp and 500 bp)

## Outputs

- SURVIVOR match files linking predictions to truth-set insertions
- Per-tool/per-threshold precision, recall, and F1-score table (e.g., benchmark_metrics.csv)
- Runtime comparison across tools when benchmarking alternatives

## How to apply

Match predicted insertions against the truth set using SURVIVOR with a default 100 bp distance margin, repeating with a 500 bp margin as the more permissive setting. From the SURVIVOR matches, compute precision (true positives / predicted positives), recall (true positives / truth-set positives), and F1 = 2·P·R/(P+R), stratifying by TE insertions versus all insertions (e.g., RetroInspector's cuteSV+Sniffles2 strict callset: 80.90% precision, 87.10% recall, 83.89% F1 for TE insertions). Apply the same pipeline to each competing method or parameter setting (e.g., read-support thresholds of 3, 5, and 7 supporting reads; lax vs strict caller-intersection criteria) so results are directly comparable. For head-to-head tool comparisons, also record runtime and thread counts so performance is reported fairly (e.g., excluding GraffiTE from timing since it used 96 threads when assigned 32).

## Related tools

- **SURVIVOR** (Matches predicted insertions against the truth set with a configurable distance margin (100 bp strict, 500 bp permissive))
- **RetroInspector** (Pipeline whose cuteSV+Sniffles2 strict callsets are evaluated; benchmark metrics computed from its outputs) — https://github.com/javiercguard/retroinspector
- **PALMER** (Comparative benchmark tool; its VCF-converted output scored against the same truth set (max F1 60.72%))
- **GraffiTE** (Comparative benchmark tool scored against the same truth set (recall 96.18%, precision 32.33%, F1 48.40%))
- **Sniffles2** (Variant caller whose intersection-based callsets feed the precision/recall computation) — https://github.com/fritzsedlazeck/Sniffles
- **cuteSV** (Variant caller whose intersection-based callsets feed the precision/recall computation) — https://github.com/tjiangHIT/cuteSV

## Examples

```
SURVIVOR compare calls.vcf truthset.vcf 100 out.diff && python compute_prf.py --matches out.diff --truth 5630 --out benchmark_metrics.csv
```

## Evaluation signals

- Strict cuteSV+Sniffles2 callset (3 supporting reads, 100 bp margin) reproduces precision ≈80.90%, TE recall ≈87.10%, F1 ≈83.89% against the 5,630-insertion truth set
- The 500 bp margin comparison is reported alongside the 100 bp default so sensitivity to the match distance is quantified
- Precision/recall recomputation at read-support thresholds 5 and 7 yields the expected trade-off (e.g., hallmark filtering raises precision above 90% at 5 and 7 reads)
- Head-to-head ordering is reproduced: RetroInspector (F1 83.89%) > PALMER (60.72%) > GraffiTE (48.40%)
- No malformed records with negative or zero coordinates remain in filtered callsets before matching

## Limitations

- PALMER's low precision may stem from an incomplete truth set, so truth-set incompleteness can bias cross-tool comparisons
- The HGSVC truth-set data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- Match-based scoring depends on the chosen SURVIVOR distance margin (100 bp vs 500 bp gives different results)

## Evidence

- [results] SURVIVOR matching rationale: [evidence span withheld — anchor status: not_found]
- [results] Hallmark filter precision gains: [evidence span withheld — anchor status: not_found]

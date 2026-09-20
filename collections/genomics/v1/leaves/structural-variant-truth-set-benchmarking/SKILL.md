---
name: structural-variant-truth-set-benchmarking
description: Use when you have merged SV or TE insertion VCF callsets and a curated
  truth set, and need precision/recall trade-off numbers to compare caller combinations
  or filtering criteria (strict vs lax).
when_to_use_negative:
- No curated truth set exists for the samples/variant classes under study — precision
  and recall cannot be computed meaningfully
- Callsets are raw single-caller outputs that have not been merged and filtered by
  the read-evidence threshold, making the strict/lax comparison ill-defined
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: SURVIVOR
  role: Compares filtered callsets against the truth set with a 100 bp distance margin
    to compute precision and recall
- name: cuteSV
  role: One of the two SV callers whose outputs form the strict/lax callsets being
    benchmarked
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second SV caller whose outputs are intersected (strict) or unioned (lax) before
    benchmarking
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: RetroInspector
  role: Pipeline whose default settings (cuteSV + Sniffles2, ≥3 supporting reads)
    produce the callsets under evaluation
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:938559b2d263d91a7db84b7d3c517e2279c38a488b730e088b744f0dd89e07fa
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
n_claims: 11
---

# structural-variant-truth-set-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks structural-variant/TE insertion callsets against a curated truth set (e.g. the HGSVC 5,630-insertion TE set) using SURVIVOR with a 100 bp distance margin to compute precision, recall, and F1 for comparing filtering criteria.

## When to use

Use when you have merged SV or TE insertion VCF callsets and a curated truth set, and need precision/recall trade-off numbers to compare caller combinations or filtering criteria (strict vs lax).

## When NOT to use

- No curated truth set exists for the samples/variant classes under study — precision and recall cannot be computed meaningfully
- Callsets are raw single-caller outputs that have not been merged and filtered by the read-evidence threshold, making the strict/lax comparison ill-defined

## Inputs

- Per-sample insertion VCF callsets from cuteSV and Sniffles2 after intra-sample and inter-sample merging
- HGSVC TE insertion truth set (5,630 insertions across three HGSVC samples)
- Read-evidence threshold (default and recommended: 3 supporting reads)

## Outputs

- Strict (intersection) and lax (union) filtered callsets
- Precision, recall, and F1 scores per callset against the truth set (e.g. strict TE precision 80.90%, recall 87.10%, F1 83.89%)

## How to apply

Build the candidate callsets to compare — e.g. a strict callset keeping insertions reported by BOTH callers (cuteSV and Sniffles2) with at least one report above the read-evidence threshold (default 3 supporting reads), and a lax callset keeping insertions from EITHER caller at the same threshold. Benchmark each filtered callset against the HGSVC TE insertion truth set (5,630 insertions across three samples) using SURVIVOR with a 100 bp distance margin to define true positives. Compute precision, recall, and F1 per callset and per variant class (TE insertions vs all insertions). The strict criterion trades some recall for higher precision: for cuteSV & Sniffles2 it reached 80.90% precision for TE insertions (85.63% for all insertions) with 87.10% recall, while the lax criterion found more insertions including more not in the benchmark set, with a precision loss lower than its recall gain.

## Related tools

- **SURVIVOR** (Compares filtered callsets against the truth set with a 100 bp distance margin to compute precision and recall)
- **cuteSV** (One of the two SV callers whose outputs form the strict/lax callsets being benchmarked) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second SV caller whose outputs are intersected (strict) or unioned (lax) before benchmarking) — https://github.com/fritzsedlazeck/Sniffles
- **RetroInspector** (Pipeline whose default settings (cuteSV + Sniffles2, ≥3 supporting reads) produce the callsets under evaluation) — https://github.com/javiercguard/retroinspector

## Examples

```
SURVIVOR compare callsets against the HGSVC truth set with 100 bp margin: `survivor merge truth.vcf query.vcf 100 benchmark_out.vcf`
```

## Evaluation signals

- Strict callset must reach TE-insertion precision of 80.90% (85.63% for all insertions) with 87.10% recall and F1 83.89%
- Strict criterion must show higher precision at lower recall than the lax criterion
- Lax callset should recover more truth-set insertions (higher recall) but include more insertions absent from the benchmark set (lower precision)
- SURVIVOR matching must use a 100 bp distance margin consistently across all comparisons
- All compared callsets must be filtered with the same read-evidence threshold (3 supporting reads) before benchmarking

## Limitations

- The HGSVC truth-set data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- A caller's low precision may partly stem from an incomplete truth set rather than genuine false positives (noted for PALMER)
- Precision/recall figures are specific to the HGSVC truth set and the cuteSV & Sniffles2 combination; other caller combinations performed similarly but were not identical

## Evidence

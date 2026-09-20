---
name: vcf-callset-intersection-union-filtering
description: Use when merging per-sample SV/TE insertion VCF callsets from two variant
  callers and you must choose between caller agreement (strict/intersection) and caller
  union (lax) to tune the precision-recall trade-off.
when_to_use_negative:
- Input is a single-caller VCF with no second callset to intersect/union — there is
  no caller-agreement signal to exploit
- The goal is de novo TE detection or annotation rather than callset-level precision/recall
  tuning after variant calling
- Variants are short-read SNV/small-indel calls rather than long-read structural or
  TE insertion calls suited to SURVIVOR comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: cuteSV
  role: One of the two long-read insertion callers whose per-sample VCFs are intersected
    (strict) or unioned (lax)
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second long-read insertion caller; agreement with cuteSV defines the strict
    callset
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: SURVIVOR
  role: Benchmarks strict vs lax callsets against the HGSVC truth set with a 100 bp
    distance margin to compute precision and recall
- name: RetroInspector
  role: Snakemake pipeline applying the read-evidence threshold filter (default 3
    supporting reads) and producing the callsets under evaluation
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:e4971567908006ea62391d5bce7d13130e973e0ae78dceb65449d15572e7f96f
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

# vcf-callset-intersection-union-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds strict and lax filtered TE-insertion callsets from merged cuteSV and Sniffles2 VCFs by taking, respectively, the intersection (both callers) or union (either caller) of insertions at a shared read-evidence threshold, to trade precision against recall.

## When to use

Use when merging per-sample SV/TE insertion VCF callsets from two variant callers and you must choose between caller agreement (strict/intersection) and caller union (lax) to tune the precision-recall trade-off.

## When NOT to use

- Input is a single-caller VCF with no second callset to intersect/union — there is no caller-agreement signal to exploit
- The goal is de novo TE detection or annotation rather than callset-level precision/recall tuning after variant calling
- Variants are short-read SNV/small-indel calls rather than long-read structural or TE insertion calls suited to SURVIVOR comparison

## Inputs

- Per-sample insertion VCF callsets from cuteSV and Sniffles2 after intra-sample and inter-sample merging
- Read-evidence threshold (integer, default 3 supporting reads)
- HGSVC TE insertion truth set (5,630 insertions across three samples)

## Outputs

- Strict filtered callset (insertions reported by both callers, intersection)
- Lax filtered callset (insertions reported by either caller, union)
- Precision/recall (and F1) metrics from SURVIVOR benchmarking at a 100 bp distance margin

## How to apply

Load the per-sample insertion VCF callsets produced by cuteSV and Sniffles2 after intra-sample and inter-sample merging, then filter all records by the user-set read-evidence threshold (default and recommended: 3 supporting reads) and repair malformed records such as negative or 0 coordinates on decoy/scaffold regions. Build the strict callset by keeping only insertions reported by BOTH callers on a sample, with at least one of the two reports surpassing the read-evidence threshold (intersection). Build the lax callset by keeping insertions reported by EITHER caller under the same threshold (union). Benchmark both callsets against the HGSVC TE insertion truth set (5,630 insertions) with SURVIVOR using a 100 bp distance margin to compute precision and recall; the rationale is that lax yields more insertions (higher recall) at the cost of more non-benchmark insertions, with the precision loss smaller than the recall gain.

## Related tools

- **cuteSV** (One of the two long-read insertion callers whose per-sample VCFs are intersected (strict) or unioned (lax)) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second long-read insertion caller; agreement with cuteSV defines the strict callset) — https://github.com/fritzsedlazeck/Sniffles
- **SURVIVOR** (Benchmarks strict vs lax callsets against the HGSVC truth set with a 100 bp distance margin to compute precision and recall)
- **RetroInspector** (Snakemake pipeline applying the read-evidence threshold filter (default 3 supporting reads) and producing the callsets under evaluation) — https://github.com/javiercguard/retroinspector

## Examples

```
snakemake --use-conda -c 8 --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- Strict callset reaches reported precision of 80.90% for TE insertions (85.63% for all insertions) with 87.10% recall and F1 of 83.89%
- Strict criterion shows higher precision at lower recall than the lax criterion
- Lax callset finds more insertions but also more insertions absent from the benchmark truth set; precision loss should be smaller than the recall gain
- All filtered records satisfy the read-evidence threshold (default 3 supporting reads) and no malformed records (negative/0 coordinates on decoy/scaffold regions) remain

## Limitations

- The HGSVC truth-set data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers, affecting benchmark accuracy
- Truth set may be incomplete, which can distort apparent precision of extra callers (as noted for PALMER)
- Increasing read-evidence threshold trades sensitivity for precision; the default of 3 reads is described as recommended, not optimal for all datasets

## Evidence

- [results] Lax criterion trade-off finding: [evidence span withheld — anchor status: not_found]

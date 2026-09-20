---
name: variant-callset-intersection-filtering
description: Use when you need to merge SV/TE insertion calls from multiple nanopore
  variant callers and quantify callset accuracy against a curated truth set (e.g.,
  the HGSVC 5,630-insertion truth set).
when_to_use_negative:
- The data are short-read sequencing data — the article benchmarks RetroInspector's
  strict/lax criteria specifically on nanopore long-read calls, and short-read TE
  tools (e.g., MELT) are a different regime
- You have only a single caller's output and cannot form the intersection — the strict
  criterion requires both cuteSV and Sniffles2 to report the insertion
- You want the highest recall of all insertions rather than TE-focused precision —
  under the strict criterion overall insertion recall was only 16.34%, so a lax criterion
  is the wrong skill here
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0199
tools:
- name: cuteSV
  role: Long-read structural variant caller providing one half of the intersection;
    calls per-sample insertions from the BAM files
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second long-read structural variant caller whose callset is intersected with
    cuteSV's; its agreement with cuteSV best matched the HGSV dataset
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: SURVIVOR
  role: Matches predicted insertions against the HGSVC truth set using a 100 bp (default)
    or 500 bp distance margin
- name: RetroInspector
  role: Snakemake pipeline implementing the strict/lax filtering criteria, read-support
    thresholds, and benchmark reporting
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:fd08623a2e036c829c624d4391dd0499167ffa54f8a95fb3677c2af5c7d322d5
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
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# variant-callset-intersection-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines structural variant callsets from two long-read callers (cuteSV and Sniffles2) by intersecting insertions and filtering on a user-set supporting-read threshold, then benchmarks the merged callset against a truth set with SURVIVOR to compute precision, recall, and F1-score for TE insertion detection.

## When to use

Use when you need to merge SV/TE insertion calls from multiple nanopore variant callers and quantify callset accuracy against a curated truth set (e.g., the HGSVC 5,630-insertion truth set).

## When NOT to use

- The data are short-read sequencing data — the article benchmarks RetroInspector's strict/lax criteria specifically on nanopore long-read calls, and short-read TE tools (e.g., MELT) are a different regime
- You have only a single caller's output and cannot form the intersection — the strict criterion requires both cuteSV and Sniffles2 to report the insertion
- You want the highest recall of all insertions rather than TE-focused precision — under the strict criterion overall insertion recall was only 16.34%, so a lax criterion is the wrong skill here

## Inputs

- BAM alignment files of basecalled nanopore reads against GRCh38 (produced by minimap2, sorted/indexed with samtools)
- Per-sample VCF callsets from cuteSV and Sniffles2
- HGSVC truth set of TE insertions (5,630 insertions across HG00733, HG00514, NA19240)
- User-set supporting-read threshold (default 3; tested at 3, 5, 7)

## Outputs

- Merged/filtered insertion callset per sample (strict intersection of cuteSV and Sniffles2 calls passing the read threshold)
- benchmark_metrics.csv table of precision, recall, and F1-score at each threshold and distance margin
- SURVIVOR matching results at 100 bp and 500 bp distance margins

## How to apply

Call structural variants per sample from minimap2-derived BAM files using cuteSV and Sniffles2, filtering out malformed records with negative or zero coordinates. Filter each caller's callset by read-evidence thresholds of 3, 5, and 7 supporting reads, then apply RetroInspector's strict (intersection) criterion: keep an insertion only when both callers report it and at least one caller surpasses the user-set read threshold — 3 reads is the default and recommended setting. Match predicted insertions against the HGSVC truth set (5,630 TE insertions) with SURVIVOR using a 100 bp distance margin (repeated at 500 bp for sensitivity to matching tolerance). Compute precision, recall, and F1 from true positives, predicted positives, and truth-set positives at each threshold/margin combination, writing the results to benchmark_metrics.csv. The rationale is that caller agreement (intersection) plus a minimal read-support cutoff maximizes precision while the article's own 'lax' criterion trades precision for recall.

## Related tools

- **cuteSV** (Long-read structural variant caller providing one half of the intersection; calls per-sample insertions from the BAM files) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second long-read structural variant caller whose callset is intersected with cuteSV's; its agreement with cuteSV best matched the HGSV dataset) — https://github.com/fritzsedlazeck/Sniffles
- **SURVIVOR** (Matches predicted insertions against the HGSVC truth set using a 100 bp (default) or 500 bp distance margin)
- **RetroInspector** (Snakemake pipeline implementing the strict/lax filtering criteria, read-support thresholds, and benchmark reporting) — https://github.com/javiercguard/retroinspector

## Examples

```
snakemake --use-conda -np --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- Strict criterion with cuteSV + Sniffles2 at 3 supporting reads and 100 bp margin should reproduce ~80.90% TE precision, 87.10% TE recall (16.34% overall), and F1 of 83.89%
- Precision across all insertions (not just TE-annotated) should be ~85.63% under the same settings
- Callset must contain no records with negative or zero coordinates (malformed records removed before filtering)
- Every retained insertion must be supported by both callers, with at least one caller meeting the read-support threshold
- Re-running with a 500 bp SURVIVOR margin should yield equal or higher matched counts than the 100 bp margin

## Limitations

- Under the strict (intersection) criterion, recall of all insertions drops to 16.34%, so the criterion trades broad insertion recall for TE precision
- No software version numbers were stated for cuteSV, Sniffles2, or other tools in the evaluation; exact settings are only in the pipeline's template configuration file
- The HGSVC data were basecalled with an older basecaller, so sequences (especially homopolymers) may be imperfect, affecting benchmark fidelity

## Evidence

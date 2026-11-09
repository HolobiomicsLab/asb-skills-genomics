---
name: truth-set-matching-distance-margin
description: Use when a TE/SV detection callset must be quantitatively scored against
  a validated truth set (e.g., HGSVC BAM/VCF truth insertions) to report precision/recall/F1
  at a defined bp distance margin.
when_to_use_negative:
- No experimentally validated truth set exists for the samples/genome, so matched
  calls cannot be interpreted as true positives (incomplete truth sets inflate apparent
  false positives)
- The callset contains SNVs or short indels well below the structural-variant scale
  (>50 bp) where coordinate-distance matching is meaningful
- Data are not nanopore/human-genome structural variant calls aligned to a reference
  (e.g., GRCh38) — the thresholds and margins were calibrated for long-read human
  SV benchmarking
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: SURVIVOR
  role: Matches predicted insertions against the truth set VCF at the 100 bp (default)
    and 500 bp (expanded) distance margins to derive true positives
- name: cuteSV
  role: Structural variant caller producing one half of the strict-criterion callset
    being benchmarked (≥3 supporting reads)
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Structural variant caller producing the other half of the strict-criterion
    callset (intersection required for acceptance)
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: RetroInspector
  role: Pipeline whose strict acceptance criterion (caller intersection + read-evidence
    threshold) defines the callset under evaluation
  repo: https://github.com/javiercguard/retroinspector
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:f27756be09eb6e5d7b7b177cc4a7cb1ca109ad1caf64781bc616ba09d651a662
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# truth-set-matching-distance-margin

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark TE insertion detection by matching a pipeline's predicted insertions against a curated truth set (e.g., the 5,630 HGSVC TE insertions) with SURVIVOR using a fixed coordinate distance margin (100 bp default, 500 bp expanded), then computing precision, recall, and F1-score per filtering threshold.

## When to use

Use when a TE/SV detection callset must be quantitatively scored against a validated truth set (e.g., HGSVC BAM/VCF truth insertions) to report precision/recall/F1 at a defined bp distance margin.

## When NOT to use

- No experimentally validated truth set exists for the samples/genome, so matched calls cannot be interpreted as true positives (incomplete truth sets inflate apparent false positives)
- The callset contains SNVs or short indels well below the structural-variant scale (>50 bp) where coordinate-distance matching is meaningful
- Data are not nanopore/human-genome structural variant calls aligned to a reference (e.g., GRCh38) — the thresholds and margins were calibrated for long-read human SV benchmarking

## Inputs

- Per-sample VCF of predicted insertions from cuteSV and Sniffles2 (malformed records with negative/zero coordinates removed)
- HGSVC truth set of 5,630 validated TE insertions (HG00733, HG00514, NA19240) on GRCh38
- Read-evidence threshold setting (e.g., 3 supporting reads) and filter criterion (strict: intersection of callers; lax: union)
- SURVIVOR merge/matching parameters (100 bp default margin; 500 bp expanded margin)

## Outputs

- Matched true-positive/positive/truth counts per threshold and margin
- Precision, recall, and F1-score table (benchmark_metrics.csv)
- TE-only and all-insertion metric breakdowns

## How to apply

Call structural variants per sample with cuteSV and Sniffles2, filtering calls by read-evidence thresholds (e.g., 3, 5, 7 supporting reads) and removing malformed records with negative or zero coordinates. Apply the acceptance criterion under test (RetroInspector's strict criterion keeps an insertion only when both callers report it and at least one surpasses the read-evidence threshold). Match predicted insertions against the HGSVC truth set (5,630 TE insertions for HG00733, HG00514, NA19240 on GRCh38) with SURVIVOR at the default 100 bp distance margin, and repeat with an expanded 500 bp margin to assess margin sensitivity. Count true positives, predicted positives, and truth positives at each threshold/margin and compute precision, recall, and F1, writing results to benchmark_metrics.csv. Sanity-check against the published reference numbers: strict cuteSV & Sniffles2 with 3 supporting reads yields 80.90% TE precision (85.63% over all insertions), 87.10% TE recall (16.34% overall), F1 83.89%.

## Related tools

- **SURVIVOR** (Matches predicted insertions against the truth set VCF at the 100 bp (default) and 500 bp (expanded) distance margins to derive true positives)
- **cuteSV** (Structural variant caller producing one half of the strict-criterion callset being benchmarked (≥3 supporting reads)) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Structural variant caller producing the other half of the strict-criterion callset (intersection required for acceptance)) — https://github.com/fritzsedlazeck/Sniffles
- **RetroInspector** (Pipeline whose strict acceptance criterion (caller intersection + read-evidence threshold) defines the callset under evaluation) — https://github.com/javiercguard/retroinspector

## Examples

```
snakemake --use-conda -c 16 --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- At the strict criterion (cuteSV & Sniffles2, 3 supporting reads, 100 bp margin), reproduce precision 80.90% TE (85.63% all insertions), recall 87.10% TE (16.34% overall), F1 83.89%
- Sanity invariant: recall over all insertions (16.34%) is far below TE recall (87.10%) because the truth set totals 5,630 TE insertions among many more SVs — a large discrepancy in this ratio signals a mismatch in truth-set subsetting
- No matched or unmatched record should carry negative or zero coordinates after preprocessing; such malformed records must be removed before SURVIVOR matching
- The 500 bp expanded margin should yield counts no worse than the 100 bp margin for the same threshold; large sensitivity to the margin indicates coordinate instability in the callset
- Metrics must be recomputed separately for each read-evidence threshold (3, 5, 7) and both strict/lax criteria, and written per row into benchmark_metrics.csv

## Limitations

- The HGSVC truth set is itself possibly incomplete — low precision of a competitor (PALMER) 'could stem from an incomplete truth set', so precision deficits may not reflect caller error
- The public HGSVC data were basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers, affecting reassembly-based matching
- Metric values are sensitive to the distance margin (100 bp vs 500 bp) and to the caller combination chosen; cuteSV and Sniffles2 agreed best with the HGSV dataset among the three tested combinations

## Evidence

---
name: structural-variant-calling-long-reads
description: Use when nanopore/long-read FASTQ or FASTA data must be turned into TE
  insertion/deletion callsets, or when caller combinations and supporting-read thresholds
  must be compared against a truth set.
when_to_use_negative:
- Input is short-read sequencing data — tools like MELT or xTea cover short-read TE
  detection; this skill relies on long-read SV callers (cuteSV, Sniffles2, SVIM)
- The goal is annotation or enrichment of already-called TE insertions rather than
  detection — RetroInspector's annotation/enrichment stages, not SV calling, are the
  relevant step
- A single caller suffices and no benchmarking is needed — the strict-criterion value
  here comes from intersecting two callers, which discards single-caller-only calls
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: RetroInspector
  role: Snakemake pipeline orchestrating SV detection, read-evidence filtering (strict/lax
    criteria, 3/5/7-read thresholds), and benchmarking of TE insertion callsets
  repo: https://github.com/javiercguard/retroinspector
- name: minimap2
  role: Aligns basecalled nanopore reads against the GRCh38 reference genome
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Sorts and indexes the minimap2 BAM alignment files before variant calling
- name: cuteSV
  role: First SV caller producing insertion VCFs; combined with Sniffles2 under the
    strict criterion for the default (recommended) RetroInspector settings
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second SV caller producing insertion VCFs; intersected with cuteSV (or SVIM)
    under the strict criterion
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: SVIM
  role: Alternative SV caller substituted for cuteSV in caller-combination comparisons
    (Sniffles2+SVIM reached F1 84.46% at a 5-read threshold)
  repo: https://github.com/eldariont/svim
- name: SURVIVOR
  role: Matches called TE insertions against the HGSVC truth set with a 100 bp (supplementary
    500 bp) distance margin for precision/recall/F1 computation
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:29f67ca071d2a8f0c49141cce933d24077535c3cb355d6e5a3854a5a43ca7015
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
  - RetroInspector
  - SAMtools
  - SURVIVOR
  - SVIM
  - Sniffles2
  - cuteSV
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# structural-variant-calling-long-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect transposable element (TE) insertions and deletions from basecalled nanopore reads by aligning to GRCh38 with minimap2, calling structural variants with multiple SV callers (cuteSV, Sniffles2, optionally SVIM), and filtering the callsets by supporting-read thresholds under strict (intersection) or lax criteria before benchmarking against a truth set.

## When to use

Use when nanopore/long-read FASTQ or FASTA data must be turned into TE insertion/deletion callsets, or when caller combinations and supporting-read thresholds must be compared against a truth set.

## When NOT to use

- Input is short-read sequencing data — tools like MELT or xTea cover short-read TE detection; this skill relies on long-read SV callers (cuteSV, Sniffles2, SVIM)
- The goal is annotation or enrichment of already-called TE insertions rather than detection — RetroInspector's annotation/enrichment stages, not SV calling, are the relevant step
- A single caller suffices and no benchmarking is needed — the strict-criterion value here comes from intersecting two callers, which discards single-caller-only calls

## Inputs

- Basecalled nanopore FASTA/FASTQ files (one per sample, e.g., HGSV samples HG00733, HG00514, NA19240)
- Reference human genome FASTA (GRCh38/hg38)
- TE insertion truth set VCF (e.g., HGSVC truth set with 5,630 TE insertions) for benchmarking

## Outputs

- Sorted, indexed BAM alignment files per sample
- Per-caller structural variant VCFs (cuteSV, Sniffles2, or SVIM)
- Filtered TE insertion callsets under strict/lax criteria at chosen supporting-read thresholds
- Benchmark metrics table (precision, recall, F1-score per caller combination, threshold, and margin, e.g., benchmark_metrics.csv)
- SURVIVOR matching results at 100 bp and 500 bp distance margins

## How to apply

Align basecalled nanopore reads per sample against the GRCh38 reference with minimap2, then sort and index the BAMs with samtools. Run two SV callers per sample — cuteSV and Sniffles2 by default (SVIM can be substituted to test alternative combinations) — producing one VCF per caller. Apply the acceptance criterion: the strict (default, recommended) mode keeps an insertion only when both callers report it (intersection) and at least one caller surpasses the user-set read-evidence threshold (default and tested thresholds of 3, 5, and 7 supporting reads), while malformed records with negative or zero coordinates are removed. To benchmark, match the called insertions against a truth set (e.g., the HGSVC truth set of 5,630 TE insertions for HG00733, HG00514, NA19240) with SURVIVOR using a 100 bp distance margin (repeat with 500 bp), then compute precision, recall, and F1-score. Rationale: cuteSV+Sniffles2 agreed best with the HGSV dataset among tested combinations (strict: precision 80.90%, TE recall 87.10%, F1 83.89%), though at a 5-read threshold Sniffles2+SVIM is marginally better (F1 84.46% vs 84.28%).

## Related tools

- **RetroInspector** (Snakemake pipeline orchestrating SV detection, read-evidence filtering (strict/lax criteria, 3/5/7-read thresholds), and benchmarking of TE insertion callsets) — https://github.com/javiercguard/retroinspector
- **minimap2** (Aligns basecalled nanopore reads against the GRCh38 reference genome) — https://github.com/lh3/minimap2
- **SAMtools** (Sorts and indexes the minimap2 BAM alignment files before variant calling)
- **cuteSV** (First SV caller producing insertion VCFs; combined with Sniffles2 under the strict criterion for the default (recommended) RetroInspector settings) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second SV caller producing insertion VCFs; intersected with cuteSV (or SVIM) under the strict criterion) — https://github.com/fritzsedlazeck/Sniffles
- **SVIM** (Alternative SV caller substituted for cuteSV in caller-combination comparisons (Sniffles2+SVIM reached F1 84.46% at a 5-read threshold)) — https://github.com/eldariont/svim
- **SURVIVOR** (Matches called TE insertions against the HGSVC truth set with a 100 bp (supplementary 500 bp) distance margin for precision/recall/F1 computation)

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Strict-criterion cuteSV+Sniffles2 at the default 3-read threshold should reproduce precision 80.90% (85.63% over all insertions), TE recall 87.10% (16.34% overall), and F1 83.89%
- At a 5-supporting-read threshold, Sniffles2+SVIM should reach ~84.46% F1 vs ~84.28% for cuteSV+Sniffles2 (precision 82.25%, recall 86.41%), matching reported Table 1
- Callsets must contain no malformed records with negative or zero coordinates after filtering
- With the recommended settings, pipeline precision should exceed 80% against the HGSV truth set
- SURVIVOR matching must be run at both the 100 bp default margin and the 500 bp expanded margin to check sensitivity to margin choice

## Limitations

- At the 3-read default, overall (all-insertion) recall is only 16.34% — the high TE recall (87.10%) is relative to annotated TE insertions, not all insertions
- The comparative evaluation could not include xTea, whose literature-reported performance is promising
- Publicly available HGSVC data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers, affecting truth-set fidelity
- PALMER's low precision in comparisons may stem from an incomplete truth set rather than the caller itself

## Evidence

---
name: te-insertion-benchmark-evaluation
description: Use when evaluating TE insertion caller performance on nanopore data
  against a curated truth set — i.e., when callset VCFs must be matched to known insertions
  to compute precision, recall, and F1 at defined read-support thresholds and distance
  margins.
when_to_use_negative:
- Input is a single-sample detection/annotation task without a truth set — use RetroInspector's
  detection/annotation/genotyping mode instead of benchmarking
- Data is short-read sequencing (e.g., for MELT-style TE analysis) rather than nanopore
  long reads aligned to GRCh38
- The reference genome or truth set coordinates are not on the same assembly (GRCh38)
  as the predictions, since matching depends on SURVIVOR coordinate comparison at
  100/500 bp margins
edam_operation: http://edamontology.org/operation_3225
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: RetroInspector
  role: Snakemake pipeline whose strict/lax filtering criteria and default 3-read
    threshold are being benchmarked
  repo: https://github.com/javiercguard/retroinspector
- name: cuteSV
  role: First variant caller in the benchmarked combination; produces insertion VCFs
    per sample
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Second variant caller in the benchmarked combination; also paired with SVIM
    in the caller-comparison arm
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: SVIM
  role: Alternative caller substituted for cuteSV in the Sniffles2+SVIM comparison
    arm at the 5-read threshold
  repo: https://github.com/eldariont/svim
- name: minimap2
  role: Aligns basecalled nanopore reads to GRCh38 to produce the BAM inputs for variant
    calling
  repo: https://github.com/lh3/minimap2
- name: SURVIVOR
  role: Matches predicted insertions against the HGSVC truth set at 100 bp (default)
    and 500 bp distance margins
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:605987b517e3829f30eb13207c1730b42044ceda2d99b7df976be127ba565af8
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
  edam_operation: http://edamontology.org/operation_3225
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# te-insertion-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking protocol for quantifying precision, recall, and F1-score of TE insertion detection pipelines (RetroInspector, cuteSV, Sniffles2, SVIM combinations) against the HGSVC truth set of 5,630 TE insertions, matching predicted insertions with SURVIVOR at 100 bp and 500 bp distance margins.

## When to use

Use when evaluating TE insertion caller performance on nanopore data against a curated truth set — i.e., when callset VCFs must be matched to known insertions to compute precision, recall, and F1 at defined read-support thresholds and distance margins.

## When NOT to use

- Input is a single-sample detection/annotation task without a truth set — use RetroInspector's detection/annotation/genotyping mode instead of benchmarking
- Data is short-read sequencing (e.g., for MELT-style TE analysis) rather than nanopore long reads aligned to GRCh38
- The reference genome or truth set coordinates are not on the same assembly (GRCh38) as the predictions, since matching depends on SURVIVOR coordinate comparison at 100/500 bp margins

## Inputs

- Basecalled nanopore FASTQ/FASTA files per sample (HGSV samples HG00733, HG00514, NA19240)
- GRCh38 human reference genome FASTA
- BAM alignments sorted and indexed with samtools
- VCF callsets from cuteSV, Sniffles2, and optionally SVIM, filtered at 3/5/7 supporting reads
- HGSVC TE insertion truth set (5,630 insertions across the three samples)

## Outputs

- benchmark_metrics.csv with precision, recall, and F1-score per caller combination, read-evidence threshold, and distance margin (100 bp and 500 bp)
- Matched/unmatched insertion callsets produced by the SURVIVOR comparison against the truth set

## How to apply

Align basecalled nanopore reads of the HGSV samples (HG00733, HG00514, NA19240) against GRCh38 with minimap2 and sort/index with samtools. Call structural variants per sample with cuteSV and Sniffles2 (or substitute SVIM for caller-comparison runs), filtering callsets at supporting-read thresholds of 3, 5, and 7 and removing malformed records with negative or zero coordinates. Apply the strict acceptance criterion — keep an insertion only when both callers report it and at least one surpasses the user-set read threshold (default and recommended: 3 reads) — versus the lax criterion, and evaluate both. Match predictions against the HGSVC truth set with SURVIVOR using the default 100 bp distance margin, and repeat at 500 bp as a sensitivity check. Compute precision, recall, and F1 from true positives, predicted positives, and truth-set positives per caller combination, threshold, and margin, writing results to benchmark_metrics.csv. The rationale for the strict intersection criterion is that cuteSV and Sniffles2's combined callset agreed best with the HGSV dataset among the tested combinations.

## Related tools

- **RetroInspector** (Snakemake pipeline whose strict/lax filtering criteria and default 3-read threshold are being benchmarked) — https://github.com/javiercguard/retroinspector
- **cuteSV** (First variant caller in the benchmarked combination; produces insertion VCFs per sample) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Second variant caller in the benchmarked combination; also paired with SVIM in the caller-comparison arm) — https://github.com/fritzsedlazeck/Sniffles
- **SVIM** (Alternative caller substituted for cuteSV in the Sniffles2+SVIM comparison arm at the 5-read threshold) — https://github.com/eldariont/svim
- **minimap2** (Aligns basecalled nanopore reads to GRCh38 to produce the BAM inputs for variant calling) — https://github.com/lh3/minimap2
- **SURVIVOR** (Matches predicted insertions against the HGSVC truth set at 100 bp (default) and 500 bp distance margins)

## Examples

```
snakemake --use-conda -np --config referenceGenome="/home/yourname/hg38.fa" outputPath="/home/yourname/experiment_results" fastq_directory="/home/yourname/data/samples"
```

## Evaluation signals

- Strict-criterion results with cuteSV & Sniffles2 at the default 3-read threshold should reproduce precision 80.90% (85.63% over all insertions), TE recall 87.10% (16.34% overall), and F1-score 83.89%
- At the 5-read threshold, the Sniffles2+SVIM arm should reach F1 ≈ 84.46% versus 84.28% for cuteSV+Sniffles2 (precision 82.25%, recall 86.41%), matching the reported caller comparison
- Malformed variant records with negative or zero coordinates must be removed before matching
- Benchmark table must include both the 100 bp default margin and the supplementary 500 bp margin comparison for sensitivity
- Metric computation should reconcile true positives, predicted positives, and truth-set positives so precision and recall are consistent at each threshold

## Limitations

- xTea was not used in the comparative evaluation, despite promising literature performance
- The publicly available HGSVC data was basecalled with an older basecaller, so sequences may be imperfect, particularly for homopolymers
- Truth sets may be incomplete — e.g., PALMER's low precision in the same comparison could stem from an incomplete truth set, inflating false positives

## Evidence

---
name: insertion-callset-matching-to-ground-truth
description: Use when comparing insertion callsets from Novel-X, PopIns2, NUI or similar
  linked-read NRS callers against a known ground truth (e.g. simulated 50-6000 bp
  insertions embedded in GRCh38) to compute per-tool recall and precision.
when_to_use_negative:
- Evaluating real (non-simulated) samples where no curated ground-truth insertion
  callset exists — use orthogonal validation (e.g. T2T CHM13 assembly for haploid
  genomes) instead
- Short-read insertion callers on ultra-short-read data where assembly of >300 bp
  insertions is known to fail
- Comparing callsets from different variant types (e.g. deletions or SNVs) — the 100
  bp/5% matching rule is defined for NRS insertions only
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3489
tools:
- name: Novel-X
  role: Linked-read NRS insertion caller whose output VCF is matched to ground truth
    in the benchmark
  repo: https://github.com/1dayac/Novel-X
- name: PopIns2
  role: Short-read insertion-calling baseline benchmarked against the same ground-truth
    callset
- name: NUI
  role: Synthetic long-read insertion-calling baseline benchmarked against the same
    ground-truth callset
- name: LRSIM
  role: Simulates the 65x synthetic long-read dataset with 2000 embedded insertions
    used as the benchmark's ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:5b5dd4f5a6637455b241eea52cc49fb2e06bdc759b47dc1729162b6c180eef55
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Novel-X
  tools:
  - LRSIM
  - NUI
  - Novel-X
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3489
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Insertion callset matching to ground truth

## Summary

Benchmarks non-reference sequence (NRS) insertion calls from linked-read methods against a ground-truth callset by matching breakpoint positions within 100 bp and insertion lengths within 5%, then tabulating recall and precision per tool, as done for Novel-X, PopIns2 and NUI in the downsampled 13x/26x/39x/52x simulated 10X-like dataset.

## When to use

Use when comparing insertion callsets from Novel-X, PopIns2, NUI or similar linked-read NRS callers against a known ground truth (e.g. simulated 50-6000 bp insertions embedded in GRCh38) to compute per-tool recall and precision.

## When NOT to use

- Evaluating real (non-simulated) samples where no curated ground-truth insertion callset exists — use orthogonal validation (e.g. T2T CHM13 assembly for haploid genomes) instead
- Short-read insertion callers on ultra-short-read data where assembly of >300 bp insertions is known to fail
- Comparing callsets from different variant types (e.g. deletions or SNVs) — the 100 bp/5% matching rule is defined for NRS insertions only

## Inputs

- Ground-truth insertion callset (VCF) for the simulated dataset (2000 insertions of 50-6000 bp embedded in GRCh38)
- Per-tool insertion callset VCFs (Novel-X, PopIns2, NUI) at each coverage level (65x/52x/39x/26x/13x)
- Downsampled linked-read datasets (10X Chromium-like FASTQ/BAM with barcodes) at ratios 0.8, 0.6, 0.4, 0.2

## Outputs

- Per-tool, per-coverage recall and precision table (mirroring Supplementary Table S1)
- TP/FP/FN counts under the 100 bp breakpoint / 5% length matching rule
- Matched/unmatched insertion events per caller

## How to apply

Run each caller (Novel-X, PopIns2, NUI) with default recommended parameters on the dataset(s) of interest — e.g. the simulated 65x dataset from LRSIM with 2000 random 50-6000 bp insertions in GRCh38, downsampled at ratios 0.8/0.6/0.4/0.2 to 52x/39x/26x/13x. Convert each tool's output VCF to a comparable set of insertion events (locus + sequence length). Match each call to the ground truth by counting a true positive only when the breakpoint positions differ by no more than 100 bp and the insertion lengths differ by no more than 5%. Compute recall (TP / ground truth) and precision (TP / calls) per tool per coverage level and tabulate them, mirroring Supplementary Table S1. Reuse the precomputed VCFs in the 1dayac/novel_insertions_supplementary repository (folder 'simulated', files with 80/60/40/20 postfixes for downsampled sets) to reproduce the benchmark.

## Related tools

- **Novel-X** (Linked-read NRS insertion caller whose output VCF is matched to ground truth in the benchmark) — https://github.com/1dayac/Novel-X
- **PopIns2** (Short-read insertion-calling baseline benchmarked against the same ground-truth callset)
- **NUI** (Synthetic long-read insertion-calling baseline benchmarked against the same ground-truth callset)
- **LRSIM** (Simulates the 65x synthetic long-read dataset with 2000 embedded insertions used as the benchmark's ground truth)

## Examples

```
git clone https://github.com/1dayac/novel_insertions_supplementary && ls Results/simulated/  # VCFs: plain (65x), 80/60/40/20 (52x/39x/26x/13x) for recall-precision matching
```

## Evaluation signals

- Every reported true positive satisfies both constraints: breakpoint positions differ by ≤ 100 bp and insertion lengths differ by ≤ 5%
- Recall/precision values reproduce Supplementary Table S1 trends: Novel-X and NUI stable at 26x coverage and above, with a drop only at 13x
- PopIns2's performance worsens gradually with each downsample step (65x → 52x → 39x → 26x → 13x)
- Output VCFs are schema-consistent with those in the novel_insertions_supplementary repository ('simulated' folder, 80/60/40/20 downsample postfixes)

## Limitations

- Matching tolerances (100 bp breakpoint, 5% length) can conflate nearly-identical events near repeats; borderline calls may flip TP/FP status
- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions often lack sufficient barcode coverage for anchor assembly, so benchmark results are dominated by >300 bp events (only insertions longer than 300 bp with anchor sum >300 bp are output)
- Ground truth derives from simulated data (LRSIM, GRCh38); conclusions may not transfer to real haploid/diploid genomes with unmodeled biases

## Evidence

- [other] counting an event as a true positive when breakpoint positions differ by no more than 100 bp and lengths differ by no more than 5%: [evidence span withheld — anchor status: not_found]
- [other] Compute recall and precision per tool per coverage level and tabulate them, mirroring Supplementary Table S1: [evidence span withheld — anchor status: not_found]
- [readme] 80, 60, 40, 20 postfix - datasets that were created by downsampling original simulated removing 80/60/40/20 percent of reads: [evidence span withheld — anchor status: external_doc]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [other] The study demonstrates high sensitivity of Novel-X on simulated data as well as real data, including well-characterized haploid and diploid human genomes.: [evidence span withheld — anchor status: not_found]

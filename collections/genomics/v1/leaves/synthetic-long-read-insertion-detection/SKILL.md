---
name: synthetic-long-read-insertion-detection
description: Use when evaluating or comparing non-reference insertion callers (e.g.
  Novel-X, PopIns2, NUI) on linked-read data, or when quantifying how coverage level
  affects NRS detection performance.
when_to_use_negative:
- The goal is detection of insertions shorter than 300 bp — Novel-X only outputs insertions
  longer than 300 bp with anchor lengths exceeding 300 bp, and needs a substantial
  number of distinct barcodes that small insertions usually lack
- Input data are ultra-short reads without barcode information — this skill requires
  linked-reads (10x Genomics, TELL-Seq or stLFR) and barcode-aware callers
- The goal is whole-genome de novo assembly-based detection — this skill benchmarks
  targeted/local-assembly callers and does not cover expensive whole-genome assembly
  approaches
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Novel-X
  role: Primary NRS insertion caller under evaluation; integrated alignment-based
    and local assembly-based algorithm using linked-read barcode information
  repo: github.com/1dayac/Novel-X
- name: PopIns2
  role: Baseline short-read-based insertion caller benchmarked for coverage sensitivity
- name: NUI
  role: Baseline synthetic long-read insertion caller benchmarked for coverage sensitivity
- name: LRSIM
  role: Simulator used to generate the ground-truth 65x synthetic long-read dataset
    with 2000 embedded insertions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:55a36e47bdde0da71caa5206202cd40d9762509931726fd9220fe81a68b8494b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LRSIM
  - NUI
  - Novel-X
  - PopIns2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# synthetic-long-read-insertion-detection

## Summary

Benchmarking protocol for detecting non-reference sequence (NRS) insertions with linked-read (synthetic long-read) tools, based on the Novel-X paper's coverage-sensitivity assessment: downsample a simulated 65x synthetic long-read dataset to 52x/39x/26x/13x, run competing callers, and score recall/precision against a ground-truth insertion callset.

## When to use

Use when evaluating or comparing non-reference insertion callers (e.g. Novel-X, PopIns2, NUI) on linked-read data, or when quantifying how coverage level affects NRS detection performance.

## When NOT to use

- The goal is detection of insertions shorter than 300 bp — Novel-X only outputs insertions longer than 300 bp with anchor lengths exceeding 300 bp, and needs a substantial number of distinct barcodes that small insertions usually lack
- Input data are ultra-short reads without barcode information — this skill requires linked-reads (10x Genomics, TELL-Seq or stLFR) and barcode-aware callers
- The goal is whole-genome de novo assembly-based detection — this skill benchmarks targeted/local-assembly callers and does not cover expensive whole-genome assembly approaches

## Inputs

- Simulated 65x synthetic long-read dataset (10X Chromium linked-reads, simulated with LRSIM) with 2000 random insertions of 50-6000 bp embedded in GRCh38
- Ground-truth insertion callset (VCF) for the simulated dataset
- Downsampled read sets at ratios 0.8/0.6/0.4/0.2 (52x/39x/26x/13x coverage)
- Reference genome GRCh38

## Outputs

- Per-tool, per-coverage VCF files of called NRS insertions (available in the simulated/ folder of 1dayac/novel_insertions_supplementary)
- Recall and precision table per tool per coverage level (mirroring Supplementary Table S1)

## How to apply

Load the simulated 65x synthetic long-read dataset (2000 random insertions of 50-6000 bp embedded in GRCh38, reads simulated with LRSIM) together with the ground-truth insertion callset from the novel_insertions_supplementary repository. Downsample the read set at ratios 0.8, 0.6, 0.4 and 0.2 to yield 52x, 39x, 26x and 13x coverage datasets. Run Novel-X, PopIns2 and NUI on each downsampled dataset with default recommended parameters, then match each method's insertion calls to the ground truth, counting an event as a true positive when breakpoint positions differ by no more than 100 bp and lengths differ by no more than 5%. Compute recall and precision per tool per coverage level and tabulate them, mirroring Supplementary Table S1; expect Novel-X and NUI to remain stable at 26x and above with a drop only at 13x, while PopIns2 degrades gradually at each downsample step.

## Related tools

- **Novel-X** (Primary NRS insertion caller under evaluation; integrated alignment-based and local assembly-based algorithm using linked-read barcode information) — github.com/1dayac/Novel-X
- **PopIns2** (Baseline short-read-based insertion caller benchmarked for coverage sensitivity)
- **NUI** (Baseline synthetic long-read insertion caller benchmarked for coverage sensitivity)
- **LRSIM** (Simulator used to generate the ground-truth 65x synthetic long-read dataset with 2000 embedded insertions)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- True positives are defined as calls whose breakpoint positions differ by no more than 100 bp and whose lengths differ by no more than 5% from the ground truth — verify the matching script enforces both thresholds
- Novel-X and NUI recall/precision should remain stable at 26x, 39x and 52x coverage, with a visible drop only at 13x
- PopIns2 recall and precision should worsen gradually with each downsample step (52x -> 39x -> 26x -> 13x)
- Recomputed metrics should match Supplementary Table S1 values reproduced from the VCFs in the simulated/ folder of the supplementary repository

## Limitations

- Performance at 13x coverage drops even for Novel-X and NUI, so conclusions from this skill are only reliable at 26x coverage and above
- Novel-X needs a substantial number of distinct barcodes; for small insertions the number of associated barcodes is usually small and coverage is insufficient for anchor assembly
- The sensitivity curve is established on a simulated LRSIM dataset with 2000 insertions of 50-6000 bp in GRCh38 and may not transfer to all real cohorts or platforms

## Evidence

- [abstract] The study demonstrates high sensitivity of Novel-X on simulated data as well as real data, including well-characterized haploid and diploid human genomes.: [evidence span withheld — anchor status: not_found]
- [intro] Here we evaluated our method on several cohorts of synthetic long reads sequenced on 10x Genomics, TELL-Seq and stLFR platforms.: "Here we evaluated our method on several cohorts of synthetic long reads sequenced on 10x Genomics, TELL-Seq and stLFR platforms."
- [readme] 80, 60, 40, 20 postfix - datasets that were created by downsampling original simulated removing 80/60/40/20 percent of reads: [evidence span withheld — anchor status: external_doc]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]

---
name: simulation-based-benchmark-evaluation
description: 'Use when comparing insertion/NRS-calling tools for depth robustness:
  a simulated linked-read dataset with ground truth exists and the goal is to quantify
  how recall and precision degrade as coverage drops from 65x toward 13x.'
when_to_use_negative:
- The goal is to characterize real biological samples (e.g. the 68-sample Polaris
  PGx cohort) rather than measure tool depth-sensitivity — use real-data validation
  instead.
- No ground-truth insertion callset exists for the dataset, making TP/FP assignment
  at the 100 bp breakpoint / 5% length tolerance impossible.
- Evaluating methods on data types without barcode/linked-read structure (e.g. ultra
  short-read only), where linked-read-based callers like Novel-X do not apply.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: Novel-X
  role: Primary NRS-insertion caller benchmarked across downsampled coverage levels;
    alignment + local assembly using linked-read barcodes
  repo: https://github.com/1dayac/Novel-X
- name: PopIns2
  role: Short-read insertion-calling baseline compared against Novel-X in the coverage-sensitivity
    analysis
- name: NUI
  role: Synthetic long-read insertion-calling baseline compared against Novel-X in
    the coverage-sensitivity analysis
- name: LRSIM
  role: Simulator used to generate the 65x ground-truth synthetic long-read dataset
    with 2000 embedded insertions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:b66949fabfd9cbd5eec1546bfac30575802d6299377c4982972c7fe446f1a7c4
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# simulation-based-benchmark-evaluation

## Summary

Assess NRS-insertion detection performance under controlled sequencing depth by benchmarking tools (Novel-X, PopIns2, NUI) on an LRSIM-simulated 65x linked-read dataset downsampled to 52x/39x/26x/13x, scoring recall and precision against a known ground-truth insertion callset (Supplementary Table S1 of the Novel-X paper).

## When to use

Use when comparing insertion/NRS-calling tools for depth robustness: a simulated linked-read dataset with ground truth exists and the goal is to quantify how recall and precision degrade as coverage drops from 65x toward 13x.

## When NOT to use

- The goal is to characterize real biological samples (e.g. the 68-sample Polaris PGx cohort) rather than measure tool depth-sensitivity — use real-data validation instead.
- No ground-truth insertion callset exists for the dataset, making TP/FP assignment at the 100 bp breakpoint / 5% length tolerance impossible.
- Evaluating methods on data types without barcode/linked-read structure (e.g. ultra short-read only), where linked-read-based callers like Novel-X do not apply.

## Inputs

- Simulated 65x synthetic long-read dataset (2000 random insertions of 50-6000 bp in GRCh38, simulated with LRSIM)
- Ground-truth insertion callset (VCF) for the simulated data
- Read BAM/FASTQ files downsampled at ratios 0.8, 0.6, 0.4, 0.2 (52x/39x/26x/13x)

## Outputs

- Per-tool, per-coverage recall and precision table (Supplementary Table S1 equivalent)
- Called insertion VCFs per tool per coverage level (as provided in the novel_insertions_supplementary repository under the 'simulated' folder with 80/60/40/20 postfixes)
- Sensitivity/robustness verdict per method across coverage levels

## How to apply

Simulate or load a 65x synthetic long-read dataset containing 2000 random insertions of 50-6000 bp embedded in GRCh38, generated with LRSIM, plus its ground-truth callset. Downsample the read set at ratios 0.8, 0.6, 0.4 and 0.2 to yield 52x, 39x, 26x and 13x coverage datasets. Run Novel-X, PopIns2 and NUI on each downsampled dataset with default recommended parameters, then match each method's calls to the ground truth, counting a true positive when breakpoint positions differ by no more than 100 bp and lengths differ by no more than 5%. Compute recall and precision per tool per coverage level and tabulate them (mirroring Supplementary Table S1). Correct execution is judged by the expected pattern: Novel-X and NUI remain stable at 26x and above, dropping only at 13x, while PopIns2's performance worsens gradually with each downsample step.

## Related tools

- **Novel-X** (Primary NRS-insertion caller benchmarked across downsampled coverage levels; alignment + local assembly using linked-read barcodes) — https://github.com/1dayac/Novel-X
- **PopIns2** (Short-read insertion-calling baseline compared against Novel-X in the coverage-sensitivity analysis)
- **NUI** (Synthetic long-read insertion-calling baseline compared against Novel-X in the coverage-sensitivity analysis)
- **LRSIM** (Simulator used to generate the 65x ground-truth synthetic long-read dataset with 2000 embedded insertions)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- Novel-X and NUI recall/precision are stable at 26x coverage and above, with a drop appearing only at 13x.
- PopIns2's performance worsens gradually with each downsample step (52x -> 39x -> 26x -> 13x).
- TP counts match calls whose breakpoints differ by <= 100 bp and lengths differ by <= 5% from ground truth; recompute recall/precision and check reproducibility against Supplementary Table S1.
- Called VCFs per downsampled dataset correspond to the supplementary files with 80/60/40/20 postfixes in the novel_insertions_supplementary repository.

## Limitations

- Performance drops at 13x coverage for Novel-X and NUI; below ~26x results are not reliable.
- Novel-X needs a substantial number of distinct barcodes per insertion; small insertions usually have too few associated barcodes and insufficient coverage, biasing benchmarks toward insertions longer than 300 bp.
- Only insertions longer than 300 bp with anchor sums exceeding 300 bp are output, so the benchmark does not assess small-insertion sensitivity.

## Evidence

- [readme] Downsampled simulated VCFs: [evidence span withheld — anchor status: external_doc]
- [methods] Coverage-sensitivity workflow: [evidence span withheld — anchor status: not_found]
- [methods] True positive definition: [evidence span withheld — anchor status: not_found]
- [abstract] High sensitivity on simulated and real data: [evidence span withheld — anchor status: not_found]
- [methods] Minimum insertion size filter: [evidence span withheld — anchor status: not_found]

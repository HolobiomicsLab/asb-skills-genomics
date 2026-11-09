---
name: bs-seq-benchmark-comparison
description: Use when evaluating Dorado basecalling/mod-calling output on ONT R10.4
  plant data and you need the adaptive P_th threshold (minimum-population bin of methylation_rate)
  plus Pearson correlation against BS-seq to quantify improvement over Dorado defaults.
when_to_use_negative:
- When the goal is DeepPlant model training or CHH-positive sample screening rather
  than benchmarking Dorado calls against BS-seq truth data
- When working with R9.4-platform tools (Tombo, Megalodon, DeepSignal-Plant) — this
  workflow targets R10.4 Dorado calls
- When no BS-seq methylation frequencies are available for the species, since the
  Pearson correlation benchmark cannot be computed
edam_operation: http://edamontology.org/operation_0564
tools:
- name: Dorado
  role: Produces the per-site methylation_rate calls (call_mods-style TSV) that are
    thresholded and benchmarked against BS-seq
  repo: nanoporetech/dorado
- name: DeepPlant
  role: Source of the adaptive threshold-selection method (minimizing the sum of false-positive
    and false-negative calls) applied here to Dorado calls
  repo: github.com/xiaochuanle/DeepPlant
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-58576-x
    title: Accurate cross-species 5mC detection for Oxford Nanopore sequencing in
      plants with DeepPlant
  source_content_hash: sha256:e9aa69a64a4eabf8c39a7cbf6b2831650720828364d3a3388a8adbe4824899dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - DeepPlant
  - Dorado
  edam_operation: http://edamontology.org/operation_0564
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bs-seq-benchmark-comparison

## Summary

Applies the adaptive methylation score threshold selection (P_th from the probability-distribution trough, borrowed from DeepPlant) to Dorado methylation calls, then benchmarks thresholded calls against BS-seq methylation frequencies via whole-genome Pearson correlation to reproduce the Dorado* rows of the comparison table.

## When to use

Use when evaluating Dorado basecalling/mod-calling output on ONT R10.4 plant data and you need the adaptive P_th threshold (minimum-population bin of methylation_rate) plus Pearson correlation against BS-seq to quantify improvement over Dorado defaults.

## When NOT to use

- When the goal is DeepPlant model training or CHH-positive sample screening rather than benchmarking Dorado calls against BS-seq truth data
- When working with R9.4-platform tools (Tombo, Megalodon, DeepSignal-Plant) — this workflow targets R10.4 Dorado calls
- When no BS-seq methylation frequencies are available for the species, since the Pearson correlation benchmark cannot be computed

## Inputs

- Dorado call_mods-style TSV output with per-site methylation_rate values for cytosine sites
- Reference genome / genomic window definitions for aggregating methylation frequency
- BS-seq methylation frequency calls for the same species and sequence context (CpG/CHG/CHH)

## Outputs

- Adaptive confidence threshold P_th (left endpoint of the minimum-population bin in 0.2–0.9, 70 bins of step 0.01)
- Thresholded site-level methylation calls (methylation_rate >= P_th)
- Per-window, per-context methylation frequency table
- Dorado* correlation table: whole-genome Pearson correlations vs BS-seq per species/context, with improvement over Dorado defaults

## How to apply

Load the Dorado methylation call output (a TSV with per-site methylation_rate values from a call_mods-style process) and bin all called cytosine sites by methylation_rate across the range 0.2–0.9 into 70 equal bins of step 0.01, counting sites per bin. Identify the minimum-population bin and set the confidence threshold P_th to its left endpoint — the rationale is that this trough separates false-positive from false-negative calls (minimizing their sum), the same strategy DeepPlant uses. Retain only calls with methylation_rate >= P_th and aggregate methylation frequency per genomic window and per CHH context. Compute whole-genome Pearson correlations between the thresholded Dorado calls and BS-seq methylation frequencies for each species/context combination, tabulating the resulting Dorado* correlations.

## Related tools

- **Dorado** (Produces the per-site methylation_rate calls (call_mods-style TSV) that are thresholded and benchmarked against BS-seq) — nanoporetech/dorado
- **DeepPlant** (Source of the adaptive threshold-selection method (minimizing the sum of false-positive and false-negative calls) applied here to Dorado calls) — github.com/xiaochuanle/DeepPlant

## Evaluation signals

- Computed Dorado* Pearson correlation values must match the published Dorado rows of the benchmark comparison table within rounding tolerance
- Derived improvement over Dorado must fall within the reported band (correlations 0.705 to 0.881; improvement 14.0% to 117.6%)
- P_th must equal the left endpoint of the minimum-population bin among the 70 bins spanning methylation_rate 0.2–0.9
- All retained calls must satisfy methylation_rate >= P_th; per-window/CHH frequencies aggregated only from retained sites
- Correlation computed per species/context combination and tabulated as the Dorado* condition alongside Dorado defaults

## Limitations

- The trough-based threshold assumes the methylation_rate distribution has a detectable minimum-population bin in 0.2–0.9; a flat or shifted score distribution can misplace P_th
- Whole-genome correlation with BS-seq is bounded by BS-seq's short-read limitations in complex and repetitive regions (centromeres, TEs), so even ideal nanopore calls may not reach perfect agreement
- Results are species- and context-dependent: DeepPlant/Dorado* correlations were reported across nine species for CHH (0.705–0.838 range for DeepPlant), and Dorado* performance varies by species/context

## Evidence

- [abstract] Dorado* improvement band: "Evaluated across nine species, DeepPlant achieves high whole-genome methylation frequency correlations (0.705-0.838) with BS-seq data on CHH, improved by 23.4- 117.6% compared to Dorado"
- [intro] Dorado as R10.4 baseline: "Dorado, the only tool for plant 5mC detection on the R10.4 platform, lacks extensive species testing"
- [intro] BS-seq correlation benchmark framing: "In parallel, we develop DeepPlant, which outperforms Dorado, achieving a correlation improvement ranging from 0.135 to 0.381 with BS-seq in whole-genome CHH methylation frequency quantification."
- [discussion] Dorado benchmark limitation: "The computational benchmark showed that DeepPlant is less computationally efficient than Dorado as it took much longer to call methylation on the same dataset (Supplementary Data 10)"
- [intro] BS-seq short-read bias rationale: [evidence span withheld — anchor status: not_found]

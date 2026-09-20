---
name: multi-tool-results-tabulation
description: 'Use when benchmarking several overlap-detection or sequence-search tools
  on the same read sets and you need a unified comparison table of error rate, mean
  overlap size, #COC, #CC, runtime, and peak memory per tool-dataset pair.'
when_to_use_negative:
- You have only a single tool's output on a single dataset — a per-tool report, not
  a cross-tool tabulation, is appropriate
- The tools produce incomparable output types with no shared ground-truth alignment
  evaluation against the same reference assembly
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0196
tools:
- name: Fedrann
  role: Benchmarked overlap-detection tool whose per-dataset accuracy/runtime metrics
    are tabulated
  repo: https://github.com/jzhang-dev/FEDRANN
- name: minimap2
  role: Baseline overlap-detection tool; its PAF outputs are evaluated and its metrics
    aggregated into the comparison table
- name: MECAT2
  role: Baseline overlap-detection tool benchmarked alongside Fedrann
- name: MHAP
  role: Baseline overlap-detection tool benchmarked alongside Fedrann
- name: wtdbg2
  role: Baseline overlap-detection tool benchmarked alongside Fedrann
- name: xRead
  role: Baseline overlap-detection tool benchmarked alongside Fedrann
- name: BLEND
  role: Baseline overlap-detection tool benchmarked alongside Fedrann
- name: kNN-overlap-finder workflows
  role: Reproducible benchmarking and evaluation workflows that generate the metrics
    to tabulate
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: Python
  role: Implementation language used for metric aggregation into the CSV table
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:d7dbc82395b8fca0810604d68d62ab89c1520fec1083801c6e169cd84751ba41
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - BLEND
  - Fedrann
  - MECAT2
  - MHAP
  - Python
  - kNN-overlap-finder workflows
  - minimap2
  - wtdbg2
  - xRead
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# multi-tool-results-tabulation

## Summary

Aggregate overlap-detection quality and efficiency metrics from multiple tools (Fedrann, minimap2, MECAT2, MHAP, wtdbg2, xRead, BLEND) across datasets into a single CSV with one row per tool-dataset pair, enabling direct accuracy and runtime comparison.

## When to use

Use when benchmarking several overlap-detection or sequence-search tools on the same read sets and you need a unified comparison table of error rate, mean overlap size, #COC, #CC, runtime, and peak memory per tool-dataset pair.

## When NOT to use

- You have only a single tool's output on a single dataset — a per-tool report, not a cross-tool tabulation, is appropriate
- The tools produce incomparable output types with no shared ground-truth alignment evaluation against the same reference assembly

## Inputs

- Overlap output files from each tool (e.g. Fedrann overlaps.tsv, minimap2 PAF)
- Reference-based ground-truth alignments (filtered PAF/alignment files)
- Wall-clock runtime and peak memory measurements per tool run
- Long-read sequencing datasets (FASTQ/FASTA): ONT SRR10028111, SRR13070625, PacBio HiFi/CycloneSEQ HG002, H4C, H4O

## Outputs

- Single CSV table with one row per tool-dataset combination
- Columns: error rate, mean overlap size, #COC, #CC, runtime, peak memory

## How to apply

Run each tool (Fedrann, minimap2, MHAP, MECAT2, wtdbg2, xRead, BLEND) on the identical read sets — e.g. C. elegans ONT SRR10028111, D. melanogaster ONT SRR13070625, and human H4C/H4O/H4P benchmarks — recording wall-clock runtime and peak memory for every run. Evaluate each tool's overlap output against reference-based ground-truth alignments filtered by read length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30, and alignments requiring >= 100 matched bases and >= 10% identity; note that for minimap2 PAF files on human WGS the identity threshold is raised to 30% due to >4TB outputs. Compute error rate, mean overlap size, number of connected components (#COC), and #CC per tool-dataset pair. Aggregate all metrics into one CSV table with one row per tool-dataset combination (7 tools x 5 datasets), with columns for error rate, mean overlap size, #COC, #CC, runtime, and peak memory. Validate the table by checking every tool-dataset cell is populated with all required metric fields before drawing conclusions (e.g. Fedrann most accurate on 4/5 datasets, MECAT2 slightly better on H4C).

## Related tools

- **Fedrann** (Benchmarked overlap-detection tool whose per-dataset accuracy/runtime metrics are tabulated) — https://github.com/jzhang-dev/FEDRANN
- **minimap2** (Baseline overlap-detection tool; its PAF outputs are evaluated and its metrics aggregated into the comparison table)
- **MECAT2** (Baseline overlap-detection tool benchmarked alongside Fedrann)
- **MHAP** (Baseline overlap-detection tool benchmarked alongside Fedrann)
- **wtdbg2** (Baseline overlap-detection tool benchmarked alongside Fedrann)
- **xRead** (Baseline overlap-detection tool benchmarked alongside Fedrann)
- **BLEND** (Baseline overlap-detection tool benchmarked alongside Fedrann)
- **kNN-overlap-finder workflows** (Reproducible benchmarking and evaluation workflows that generate the metrics to tabulate) — https://github.com/jzhang-dev/kNN-overlap-finder
- **Python** (Implementation language used for metric aggregation into the CSV table)

## Examples

```
python -c "import pandas as pd; df = pd.DataFrame(rows, columns=['tool','dataset','error_rate','mean_overlap_size','COC','CC','runtime_s','peak_mem_GB']); df.to_csv('benchmark_results.csv', index=False)"
```

## Evaluation signals

- CSV table exists with all required metric columns (error rate, mean overlap size, #COC, #CC, runtime, peak memory) populated for all 7 tools x 5 datasets
- Each tool-dataset row has runtime and peak memory recorded from the same execution environment
- Ground-truth evaluation applied uniform filters (reads >= 5 kb, aligned fraction >= 50%, MAPQ >= 30; alignments >= 100 matched bases and >= 10% identity) across all tools
- Reproduced comparison matches the paper's finding: Fedrann most accurate on 4/5 datasets, with MECAT2 slightly better on H4C, and Fedrann fastest on H4C/H4O

## Limitations

- minimap2 PAF files on human whole-genome alignments produced exceptionally large outputs exceeding 4TB, requiring a raised 30% identity threshold — a per-tool caveat that must be recorded in the tabulation
- No DOI or Zenodo archive is provided; reproducibility of the tabulated results depends solely on the GitHub repositories
- Memory consumption figures should be interpreted carefully since Fedrann's memory use is higher than most existing overlap detection tools

## Evidence

- [readme] This repository contains all testing, data processing, benchmarking, and evaluation workflows for the development and publication of FEDRANN software: [evidence span withheld — anchor status: external_doc]
- [abstract] Fedrann comparable or better than state-of-the-art: "Fedrann produces overlap graphs comparable to or better than those generated by existing state-of-the-art tools, including MECAT2, minimap2, and wtdbg2, while maintaining competitive runtime"
- [intro] Benchmarked against six established tools: "We benchmarked Fedrann against established tools, including minimap2, MHAP, MECAT2, wtdbg2, xRead, and BLEND."
- [discussion] Ground-truth read filter thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Alignment filter criteria: [evidence span withheld — anchor status: not_found]
- [discussion] minimap2 human PAF identity threshold exception: "for minimap2 PAF files (which produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%"

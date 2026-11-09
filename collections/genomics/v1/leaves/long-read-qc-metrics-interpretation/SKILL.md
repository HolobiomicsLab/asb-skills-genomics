---
name: long-read-qc-metrics-interpretation
description: Use when raw nanopore FASTQ reads need QC before AMR or assembly analysis
  in NanoGalaxy, or when per-read length, Phred quality, and yield summary statistics
  must be reported.
when_to_use_negative:
- The input is short-read (150–300 bp) sequencing data rather than nanopore long-read
  data — the NanoGalaxy toolkit targets Nanopore long-read data.
- A downstream analysis goal other than QC, such as de novo genome assembly or AMR
  determination, is the immediate task and QC has already been completed.
- No FASTQ read-level input exists (e.g., input is already an assembled genome or
  a variant/AMR report).
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: NanoPlot
  role: Computes per-read length and Phred quality statistics and renders QC plots
    and an HTML report from the nanopore FASTQ input
- name: Galaxy
  role: Platform hosting the NanoGalaxy toolkit; provides the user-friendly interface
    and dependency management for running the NanoPlot QC step
  repo: https://nanopore.usegalaxy.eu
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:9dc06bdd6e28655e38d9dcffedcd0807fd88c1d6d4057eb02e3a32ceaa1fdc10
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - NanoPlot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-qc-metrics-interpretation

## Summary

Quality assessment of Oxford Nanopore long-read FASTQ data using NanoPlot within the NanoGalaxy Galaxy toolkit, producing per-read length and Phred-quality statistics and plots prior to downstream AMR analysis or genome assembly.

## When to use

Use when raw nanopore FASTQ reads need QC before AMR or assembly analysis in NanoGalaxy, or when per-read length, Phred quality, and yield summary statistics must be reported.

## When NOT to use

- The input is short-read (150–300 bp) sequencing data rather than nanopore long-read data — the NanoGalaxy toolkit targets Nanopore long-read data.
- A downstream analysis goal other than QC, such as de novo genome assembly or AMR determination, is the immediate task and QC has already been completed.
- No FASTQ read-level input exists (e.g., input is already an assembled genome or a variant/AMR report).

## Inputs

- Raw Oxford Nanopore FASTQ read files (e.g., from the GigaDB dataset supporting the NanoGalaxy AMR use case)

## Outputs

- NanoPlot interactive HTML report with summary statistics table (read count, read length, quality metrics)
- Log-scaled read-length distribution plots
- Quality-vs-length scatter plots
- Aggregate QC metrics (read count, N50, yield)

## How to apply

Obtain the raw nanopore FASTQ read files (for the AMR use case, from the GigaDB dataset referenced by the paper) and run NanoPlot on the FASTQ input with default reporting settings via the NanoGalaxy Galaxy interface at https://nanopore.usegalaxy.eu. NanoPlot computes per-read length, per-read Phred quality, and aggregate summary statistics such as read count, N50, and yield. Generate the standard NanoPlot artifacts: log-scaled read-length distribution plots, quality-vs-length scatter plots, and the interactive HTML report containing the summary statistics table. Save the NanoPlot HTML report and plot files as the QC output artifacts. Per the source workflow, acceptance is judged by presence and structural completeness of the report rather than a numeric threshold, since the paper reports no QC cutoff values.

## Related tools

- **NanoPlot** (Computes per-read length and Phred quality statistics and renders QC plots and an HTML report from the nanopore FASTQ input)
- **Galaxy** (Platform hosting the NanoGalaxy toolkit; provides the user-friendly interface and dependency management for running the NanoPlot QC step) — https://nanopore.usegalaxy.eu

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The NanoPlot HTML report file exists and is valid HTML.
- The report contains a summary statistics table including read count, read-length, and quality metrics.
- Standard NanoPlot plots (log-scaled read-length distribution, quality-vs-length scatter) are present among the output artifacts.
- Aggregate statistics such as read count, N50, and yield are populated with values consistent with the input FASTQ.
- Acceptance is judged by presence and structural completeness of the report — no numeric QC threshold is expected, as the paper reports no QC cutoff values.

## Limitations

- The paper reports no QC cutoff values, so this step cannot enforce pass/fail quality filtering against published thresholds.
- Other long-read analysis tools exist but were not integrated into NanoGalaxy; the toolkit focused on the most established and widely used tools.
- NanoGalaxy's integrated tools were intensively tested for ONT data; support for other platforms (e.g., Pacific Biosciences) is untested ('should also work').

## Evidence

- [abstract] The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows.: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [intro] NanoGalaxy toolkit introduction: "Here we introduce the NanoGalaxy toolkit for analysing Nanopore long-read data."
- [discussion] Training manual availability: "An online training manual for the AMR use case described in this publication"
- [discussion] Supporting data location: "Additional supporting data are available from the GigaScience GigaDB database [53]"
- [readme] README description: "NanoGalaxy: Galaxy toolkit and workflows for long-read Oxford Nanopore data"

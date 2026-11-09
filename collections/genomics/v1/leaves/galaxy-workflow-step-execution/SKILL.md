---
name: galaxy-workflow-step-execution
description: Use when a nanopore long-read analysis step (e.g., NanoPlot QC on raw
  FASTQ reads from the GigaDB AMR dataset) must be executed as a step of a Galaxy/NanoGalaxy
  workflow.
when_to_use_negative:
- The analysis goal is de novo assembly or metagenomic analysis with dedicated long-read
  assemblers rather than a per-step QC/analysis execution — use the corresponding
  NanoGalaxy assembly workflows instead
- The input data are not Oxford Nanopore long reads (the integrated tools were intensively
  tested for ONT data; Pacific Biosciences support is only expected, not tested)
- A strict numeric QC cutoff must be enforced — the paper specifies acceptance by
  report presence and structural completeness, not by thresholds
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Galaxy
  role: Platform on which the workflow step is executed; provides the user-friendly
    interface, handles software dependencies, and manages the Conda dependency framework
  repo: https://nanopore.usegalaxy.eu
- name: NanoGalaxy
  role: Toolkit providing the wrapped, best-practice tools and workflows (e.g., NanoPlot
    QC) for analysing Nanopore long-read data
  repo: mmiladi/galaxy-nanopore
- name: NanoPlot
  role: 'Tool invoked as the quality-assessment workflow step: computes per-read length,
    per-read Phred quality, and summary statistics and emits HTML/plots as QC artifacts'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:4c80fe8ca0406ed71481dbbb228a18afffed398a887b43ef9153897a0c6935a1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - NanoGalaxy
  - NanoPlot
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# galaxy-workflow-step-execution

## Summary

Executes individual analysis steps of the NanoGalaxy long-read Oxford Nanopore toolkit (e.g., NanoPlot quality assessment of nanopore FASTQ data) as Galaxy workflow steps on the Galaxy platform, which handles software dependencies so no command-line coding is required.

## When to use

Use when a nanopore long-read analysis step (e.g., NanoPlot QC on raw FASTQ reads from the GigaDB AMR dataset) must be executed as a step of a Galaxy/NanoGalaxy workflow.

## When NOT to use

- The analysis goal is de novo assembly or metagenomic analysis with dedicated long-read assemblers rather than a per-step QC/analysis execution — use the corresponding NanoGalaxy assembly workflows instead
- The input data are not Oxford Nanopore long reads (the integrated tools were intensively tested for ONT data; Pacific Biosciences support is only expected, not tested)
- A strict numeric QC cutoff must be enforced — the paper specifies acceptance by report presence and structural completeness, not by thresholds

## Inputs

- Raw nanopore FASTQ read files (e.g., AMR use case reads from the GigaDB dataset referenced in the paper)
- Analysis workflow definition (best-practice NanoGalaxy workflows integrated in the Galaxy platform)

## Outputs

- NanoPlot HTML report with interactive summary statistics table (read count, read-length, quality metrics)
- Log-scaled read-length distribution plots
- Quality-vs-length scatter plots
- Galaxy history datasets for downstream workflow steps

## How to apply

Run each step as a wrapped tool inside the Galaxy platform, starting a NanoGalaxy instance via Docker (docker run -d -p 8080:80 mmiladi/galaxy-nanopore, then use http://localhost:8080) or the European Galaxy server at https://nanopore.usegalaxy.eu. Upload the input (e.g., raw nanopore FASTQ reads for the AMR use case obtained from the deposited GigaDB dataset) and invoke the step's tool with default reporting settings; for the NanoPlot QC step this computes per-read length, per-read Phred quality, and aggregate summary statistics (read count, N50, yield) and produces log-scaled read-length plots, quality-vs-length scatter plots, and an HTML report. Dependencies are managed automatically by the Conda framework on the Galaxy platform, so steps are reproducible without manual installation; wrappers come from the Galaxy Tool Shed and are tested via Travis continuous integration on GitHub. Acceptance of a step's output (e.g., the NanoPlot report) is judged by presence and structural completeness — a valid HTML report containing the summary statistics table — rather than a numeric cutoff, since the paper reports no QC threshold values.

## Related tools

- **Galaxy** (Platform on which the workflow step is executed; provides the user-friendly interface, handles software dependencies, and manages the Conda dependency framework) — https://nanopore.usegalaxy.eu
- **NanoGalaxy** (Toolkit providing the wrapped, best-practice tools and workflows (e.g., NanoPlot QC) for analysing Nanopore long-read data) — mmiladi/galaxy-nanopore
- **NanoPlot** (Tool invoked as the quality-assessment workflow step: computes per-read length, per-read Phred quality, and summary statistics and emits HTML/plots as QC artifacts)

## Examples

```
docker run -d -p 8080:80 mmiladi/galaxy-nanopore
```

## Evaluation signals

- The NanoPlot HTML report file exists and is valid HTML
- The report contains the summary statistics table with read count, read-length, and quality metrics
- Standard NanoPlot artifacts (log-scaled length distributions, quality-vs-length scatter plots) are present as QC outputs
- The Galaxy history shows the step completed under the platform's Conda-managed tool environment (no manual dependency installation was needed)
- No numeric QC threshold is applied — acceptance is by structural completeness of the report, matching the paper's stated validation

## Limitations

- The paper reports no QC cutoff values, so step validation is limited to presence/structure checks rather than numeric thresholds
- Support for other technologies such as Pacific Biosciences inside Galaxy is expected but untested; only ONT tools were intensively tested
- The toolkit is expected, but not yet guaranteed, to be extended by the Galaxy community

## Evidence

- [abstract] Galaxy interface for long-read steps: "The Galaxy platform provides a user-friendly interface to computational command line–based tools, handles the software dependencies, and provides refined workflows."
- [intro] NanoGalaxy toolkit introduction: "Here we introduce the NanoGalaxy toolkit for analysing Nanopore long-read data."
- [discussion] Dependency management for steps: "All tools and their dependencies are installed on the Galaxy platform and are managed by the Conda framework for dependency management"
- [readme] Docker invocation of the NanoGalaxy instance: "docker run -d -p 8080:80 mmiladi/galaxy-nanopore"

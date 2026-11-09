---
name: sequencing-fastq-input-handling
description: Use when raw nanopore sequencing data in FASTQ format must be imported
  into Galaxy (e.g., from a GigaDB dataset) and characterized before downstream long-read
  assembly or AMR analysis.
when_to_use_negative:
- Input data is already basecalled, filtered, or provided as alignment files (e.g.,
  BAM) rather than raw nanopore FASTQ reads
- The analysis goal is short-read data handling — NanoGalaxy focuses on long-read
  Oxford Nanopore data
- 'A numeric QC threshold decision is required: the paper reports no QC cutoff values,
  so acceptance is structural completeness of the report, not threshold passing'
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoPlot
  role: 'Quality assessment of the nanopore FASTQ input: computes per-read length
    and Phred quality and produces the HTML QC report'
- name: Galaxy
  role: Platform in which the FASTQ files are uploaded, managed, and fed to NanoPlot
    and downstream workflows
  repo: https://nanopore.usegalaxy.eu
- name: NanoGalaxy
  role: Galaxy-based toolkit providing the tool wrappers and workflows that consume
    the FASTQ input
  repo: mmiladi/galaxy-nanopore
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa105
    title: 'NanoGalaxy: Nanopore long-read sequencing data analysis in Galaxy'
  source_content_hash: sha256:d8baf0c69010515de5525a2b9172d70c2bb074779f861dec2db2b46597c24efe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Galaxy
  - NanoGalaxy
  - NanoPlot
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sequencing-fastq-input-handling

## Summary

Handles raw Oxford Nanopore FASTQ read files as the entry point for NanoGalaxy long-read analyses, covering upload into Galaxy and initial quality assessment (e.g., with NanoPlot) before assembly or AMR workflows.

## When to use

Use when raw nanopore sequencing data in FASTQ format must be imported into Galaxy (e.g., from a GigaDB dataset) and characterized before downstream long-read assembly or AMR analysis.

## When NOT to use

- Input data is already basecalled, filtered, or provided as alignment files (e.g., BAM) rather than raw nanopore FASTQ reads
- The analysis goal is short-read data handling — NanoGalaxy focuses on long-read Oxford Nanopore data
- A numeric QC threshold decision is required: the paper reports no QC cutoff values, so acceptance is structural completeness of the report, not threshold passing

## Inputs

- Raw Oxford Nanopore sequencing reads in FASTQ format
- Deposited dataset files from GigaDB (GigaScience database) referenced in the paper

## Outputs

- NanoPlot HTML report with summary statistics table (read count, read-length, quality metrics)
- Log-scaled read-length distribution plots
- Quality-vs-length scatter plots
- Galaxy history containing imported FASTQ datasets ready for downstream workflows

## How to apply

Obtain the raw nanopore FASTQ read files, e.g., for the AMR use case from the GigaDB dataset deposited alongside the NanoGalaxy paper. Upload the FASTQ files into a Galaxy instance (NanoGalaxy, available at https://nanopore.usegalaxy.eu or via the docker image mmiladi/galaxy-nanopore). Run NanoPlot on the FASTQ input with default reporting settings to compute per-read length, per-read Phred quality, and aggregate summary statistics such as read count, N50, and yield. Inspect the generated log-scaled read-length distribution plots, quality-vs-length scatter plots, and the interactive HTML report before proceeding to assembly or AMR steps; validation is judged by presence and structural completeness of the report, since the paper reports no numeric QC cutoff values.

## Related tools

- **NanoPlot** (Quality assessment of the nanopore FASTQ input: computes per-read length and Phred quality and produces the HTML QC report)
- **Galaxy** (Platform in which the FASTQ files are uploaded, managed, and fed to NanoPlot and downstream workflows) — https://nanopore.usegalaxy.eu
- **NanoGalaxy** (Galaxy-based toolkit providing the tool wrappers and workflows that consume the FASTQ input) — mmiladi/galaxy-nanopore

## Examples

```
NanoPlot --fastq nanopore_reads.fastq --outdir nanoplots/ --N50
```

## Evaluation signals

- The NanoPlot report file exists and is valid HTML
- The NanoPlot HTML report contains a summary statistics table with read count, read-length, and quality metrics
- Log-scaled read-length distribution and quality-vs-length scatter plots are present among the output artifacts
- The imported FASTQ dataset is recognized by Galaxy (dataset state 'ok', not failed/empty) before downstream steps run
- No numeric QC threshold is asserted — acceptance is judged by structural completeness of the report, consistent with the paper's lack of QC cutoff values

## Limitations

- The paper reports no QC cutoff values, so this step cannot gate the pipeline on numeric quality thresholds
- Other long-read analysis tools were not integrated — only the most established and widely used tools are covered, so exotic nanopore FASTQ variants may lack wrappers
- Pacific Biosciences tools inside Galaxy are untested ('should also work'), so non-ONT FASTQ inputs are not validated

## Evidence

- [other] NanoPlot is the tool designated for quality assessment of the sequencing data.: [evidence span withheld — anchor status: not_found]
- [other] Obtain the raw nanopore FASTQ read files for the AMR use case from the deposited GigaDB dataset: [evidence span withheld — anchor status: not_found]
- [abstract] The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data: "The newly developed “NanoGalaxy" is a Galaxy-based toolkit for analysing long-read sequencing data, which is suitable for diverse applications, including de novo genome assembly"
- [readme] docker run -d -p 8080:80 mmiladi/galaxy-nanopore: "``docker run -d -p 8080:80 mmiladi/galaxy-nanopore``"
- [discussion] here we have performed intensive testing of the integrated tools for ONT data.: "However, here we have performed intensive testing of the integrated tools for ONT data."

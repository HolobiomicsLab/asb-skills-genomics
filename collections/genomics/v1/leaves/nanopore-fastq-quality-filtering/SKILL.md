---
name: nanopore-fastq-quality-filtering
description: Use when new basecalled FASTQ batches from an Oxford Nanopore MinION/Flongle
  run must be quality-filtered per batch, and per-batch QC statistics (before/after
  filtering read and base counts) are needed before taxonomic classification.
when_to_use_negative:
- Input reads are already filtered FASTQ (post-FastP output) — re-filtering would
  discard additional reads and distort per-batch QC totals
- The goal is taxonomic classification or BLAST validation itself — those are handled
  by the separate Kraken2 classification and BLAST validation steps that run after
  this filtering stage
- Data are not Oxford Nanopore FASTQ batches (e.g. short-read or non-FASTQ formats)
  — the nanometanf pipeline is specific to Oxford Nanopore long-read data
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: FastP
  role: Performs the initial quality filtering of each nanopore FASTQ batch and emits
    the statistics JSON with before/after filtering totals
- name: nanometanf
  role: Nextflow pipeline whose 'Quality control (FASTP)' process orchestrates FastP
    execution per sample/batch in live or batch mode
  repo: github.com/FOI-Bioinformatics/nanometanf
- name: Nanometa Live
  role: Dashboard that consumes the per-batch FastP filtering statistics for its quality
    control panel and real-time batch tracking
  repo: github.com/FOI-Bioinformatics/nanometa_live
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:16781304b62615dc1b8cfb11a0da3e57b552fe776c1e5a76fd755a82c165c458
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastP
  - Nanometa Live
  - nanometanf
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# nanopore-fastq-quality-filtering

## Summary

Reconstructs the initial read-filtering stage of the Nanometa Live workflow, in which each newly generated nanopore FASTQ batch is passed through FastP (the 'Quality control (FASTP)' process of the nanometanf Nextflow pipeline) before downstream Kraken2 taxonomic classification.

## When to use

Use when new basecalled FASTQ batches from an Oxford Nanopore MinION/Flongle run must be quality-filtered per batch, and per-batch QC statistics (before/after filtering read and base counts) are needed before taxonomic classification.

## When NOT to use

- Input reads are already filtered FASTQ (post-FastP output) — re-filtering would discard additional reads and distort per-batch QC totals
- The goal is taxonomic classification or BLAST validation itself — those are handled by the separate Kraken2 classification and BLAST validation steps that run after this filtering stage
- Data are not Oxford Nanopore FASTQ batches (e.g. short-read or non-FASTQ formats) — the nanometanf pipeline is specific to Oxford Nanopore long-read data

## Inputs

- Basecalled FASTQ batches from an Oxford Nanopore MinION/Flongle flow cell (per-sample or per-batch files)
- nanometanf Nextflow pipeline configuration (samplesheet CSV, nanopore output directory, execution profile such as conda/docker/singularity)

## Outputs

- Per-sample FastP filtered FASTQ files ready for Kraken2 classification
- FastP statistics JSON reports in results/fastp/ (e.g. sample1.fastp.) exposing fastp_data['summary']['after_filtering']['total_reads'] and base-count fields
- Per-batch QC statistics (read counts and base pairs sequenced) used by the Nanometa Live QC panel and reporting

## How to apply

Locate the basecalled FASTQ input batches in the configured nanopore output directory (live or batch mode), then run FastP on each batch as the 'Quality control (FASTP)' process of the nanometanf Nextflow pipeline, writing per-sample reports into results/fastp/. For each batch, record the FASTP statistics JSON (e.g. sample1.fastp.) containing before/after filtering totals such as fastp_data['summary']['after_filtering']['total_reads']. The rationale is that each new FASTQ batch must be filtered before Kraken2 classification, and these after_filtering totals feed the downstream QC loaders and reporting. Validate that results/fastp/ contains one *.fastp. report per processed sample and that each report exposes after_filtering total_reads and base-count fields.

## Related tools

- **FastP** (Performs the initial quality filtering of each nanopore FASTQ batch and emits the statistics JSON with before/after filtering totals)
- **nanometanf** (Nextflow pipeline whose 'Quality control (FASTP)' process orchestrates FastP execution per sample/batch in live or batch mode) — github.com/FOI-Bioinformatics/nanometanf
- **Nanometa Live** (Dashboard that consumes the per-batch FastP filtering statistics for its quality control panel and real-time batch tracking) — github.com/FOI-Bioinformatics/nanometa_live

## Examples

```
nextflow run foi-bioinformatics/nanometanf --input samplesheet.csv --outdir results -profile conda
```

## Evaluation signals

- results/fastp/ contains exactly one *.fastp. report per processed sample/batch
- Each FastP JSON report exposes fastp_data['summary']['after_filtering']['total_reads'] and after_filtering base-count fields readable by the downstream QC loaders
- after_filtering total_reads is <= before_filtering total_reads, and the per-batch read counts match the number of reads entering Kraken2 for that batch
- Filtered reads for each batch are successfully consumed by the subsequent Kraken2 classification step without missing-input errors
- Per-batch QC statistics (read counts, base pairs sequenced) appear in the Nanometa Live quality control panel and realtime_batch_stats reporting

## Limitations

- FastP is one of several QC tools in nanometanf (alongside Chopper and NanoPlot); the article/README do not specify FastP parameter values (e.g. quality or length cutoffs), so defaults are implied but not documented in the provided context
- No tool version for FastP is specified in the provided context
- Filtering is per-batch: statistics reflect each individual FASTQ batch, not a cumulative per-sample total until later aggregation

## Evidence

- [methods] FastP runs as the QC step per batch: "| Quality control (FASTP) | yes | yes | Same |"
- [methods] Each batch is FastP-filtered before Kraken2: [evidence span withheld — anchor status: not_found]
- [readme] QC panel surfaces per-sample filtering statistics: [evidence span withheld — anchor status: external_doc]
- [readme] Real-time monitoring of FASTQ batches: [evidence span withheld — anchor status: external_doc]

---
name: cumulative-qc-metrics-accumulation
description: Use when reconstructing or reproducing real-time per-batch QC tracking
  (reads, base pairs, classified vs unclassified counts) for a nanopore sequencing
  run where FASTQ batches land incrementally and QC metrics must accumulate.
when_to_use_negative:
- The run is a completed batch-mode analysis with a single static FASTQ set rather
  than incrementally landing batches — use end-of-run QC aggregation instead.
- The goal is taxonomic visualization (Sankey/sunburst) or pathogen watchlist alerting
  rather than per-batch QC metrics accumulation.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: FASTP
  role: Extract per-batch read counts and base pairs from the summary/after_filtering
    section of the fastp JSON report
- name: Kraken2
  role: Classify each FASTQ batch and provide the report from which classified versus
    unclassified reads (rank code 'U') are tallied
- name: Nanometa Live
  role: Hosts the QC tab that displays the accumulated per-batch and cumulative QC
    metrics via bar charts
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Dash
  role: Web framework whose 10-second interval component sets the refresh cadence
    for the QC aggregation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:4f1994879601ac8c3fe6305dd81d7f40e0abd6c2f4548290426daa69a0a1189d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - Dash
  - FASTP
  - Kraken2
  - Nanometa Live
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cumulative-qc-metrics-accumulation

## Summary

Reconstructs the per-batch and cumulative quality-control statistics layer of Nanometa Live, in which FASTP read/base-pair metrics and Kraken2 classified/unclassified counts are extracted for each newly landing FASTQ batch during an Oxford Nanopore run and accumulated into running QC totals surfaced on the Dash dashboard.

## When to use

Use when reconstructing or reproducing real-time per-batch QC tracking (reads, base pairs, classified vs unclassified counts) for a nanopore sequencing run where FASTQ batches land incrementally and QC metrics must accumulate.

## When NOT to use

- The run is a completed batch-mode analysis with a single static FASTQ set rather than incrementally landing batches — use end-of-run QC aggregation instead.
- The goal is taxonomic visualization (Sankey/sunburst) or pathogen watchlist alerting rather than per-batch QC metrics accumulation.

## Inputs

- Basecalled FASTQ batches landing incrementally in the nanopore output directory (batch_1, batch_2, ...)
- Per-sample FASTP JSON reports (summary.after_filtering section)
- Kraken2 report files with columns [%, cumul_reads, reads, rank, taxid, name]

## Outputs

- results/realtime_batch_stats/batch_N. files with reads_in_batch, timestamp, base pairs, classified/unclassified counts
- cumulative_qc_stats accumulation of per-batch read counts, base pairs, and classification rates
- QC tab bar charts showing per-batch (noncumulative) and cumulative metrics

## How to apply

Monitor the nanopore sequencer output directory and ingest each newly integrated basecalled FASTQ batch (batch_1, batch_2, ...) as it lands. For each batch, run FASTP and read fastp_data['summary']['after_filtering']['total_reads'] (plus base pairs) from the per-sample fastp JSON report, then run Kraken2 and compute unclassified reads by summing reads with rank code 'U' from the concatenated report columns [%, cumul_reads, reads, rank, taxid, name]. Write each batch's statistics (reads_in_batch, timestamp, base pairs, classified/unclassified counts) to results/realtime_batch_stats/batch_N.. Accumulate per-batch values into cumulative_qc_stats, keeping both noncumulative per-batch and running cumulative read counts, base pairs, and classification rates. Refresh the aggregation on a 10-second cadence matching the Dash interval component so new batches appear in the QC tab bar charts.

## Related tools

- **FASTP** (Extract per-batch read counts and base pairs from the summary/after_filtering section of the fastp JSON report)
- **Kraken2** (Classify each FASTQ batch and provide the report from which classified versus unclassified reads (rank code 'U') are tallied)
- **Nanometa Live** (Hosts the QC tab that displays the accumulated per-batch and cumulative QC metrics via bar charts) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Dash** (Web framework whose 10-second interval component sets the refresh cadence for the QC aggregation)

## Examples

```
unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum(); stats = fastp_data['summary']['after_filtering']['total_reads']
```

## Evaluation signals

- The per-sample QC table row count matches the actual barcode count.
- Cumulative read totals equal the sum of individual realtime_batch_stats/batch_* reads_in_batch values.
- Unclassified counts equal the sum of reads with rank code 'U' in each Kraken2 report.
- Each batch file in results/realtime_batch_stats/ contains reads_in_batch, timestamp, base pairs, and classified/unclassified counts.
- Newly integrated batches appear in the QC tab bar charts within one 10-second dashboard refresh.

## Limitations

- Metrics are only as timely as the 10-second dashboard interval; very rapid batch integration may briefly lag the display.
- The unclassified count is derived from Kraken2 rank-code tallies, so accuracy depends on the classification report format being preserved.
- FASTP read counts come from the after_filtering section, so raw (pre-filter) read counts are not what is tracked.

## Evidence

- [methods] Per-batch FASTP/Kraken2 QC extraction: [evidence span withheld — anchor status: not_found]
- [results] Unclassified tally: "unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum()"
- [methods] realtime_batch_stats directory: "├── realtime_batch_stats/       # NEW: Real-time batch tracking"
- [results] 10-second dashboard refresh: "interval=10*1000,  # Update every 10 seconds"
- [readme] QC panel role: [evidence span withheld — anchor status: external_doc]

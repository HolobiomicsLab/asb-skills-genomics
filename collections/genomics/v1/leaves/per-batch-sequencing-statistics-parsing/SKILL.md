---
name: per-batch-sequencing-statistics-parsing
description: Use when monitoring a live nanopore sequencing run where FASTQ batches
  land incrementally and you need per-batch and cumulative QC metrics (reads, base
  pairs, classification rates) for a dashboard or QC table.
when_to_use_negative:
- The run is a completed batch analysis with a single final FASTQ/per-sample output
  rather than incrementally landing batches during sequencing
- You need taxonomic composition or pathogen watchlist alerts rather than read/base-pair
  QC statistics
- Input is already aggregated QC output (e.g. a MultiQC report) rather than raw per-batch
  FASTP JSON or Kraken2 reports
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Nanometa Live
  role: Front-end application that orchestrates per-batch statistics extraction, accumulation
    into cumulative_qc_stats, and QC tab visualisation
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: FASTP
  role: Computes per-batch read counts and base pairs sequenced, parsed from the after_filtering
    section of its JSON reports
- name: Kraken2
  role: Classifies reads per batch; rank code 'U' lines are summed to derive unclassified
    read counts
- name: Dash
  role: Web framework whose 10-second interval component sets the refresh cadence
    for the per-batch QC bar charts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:8c4a4f83a093a114381b53961783d534905a8e721e169e36a49fc5b22e2251ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - Dash
  - FASTP
  - Kraken2
  - Nanometa Live
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# per-batch-sequencing-statistics-parsing

## Summary

Extracts and accumulates per-batch quality control statistics (read counts, base pairs, classified vs unclassified reads) from each newly landing FASTQ batch during an Oxford Nanopore sequencing run, as implemented in Nanometa Live's realtime_batch_stats module.

## When to use

Use when monitoring a live nanopore sequencing run where FASTQ batches land incrementally and you need per-batch and cumulative QC metrics (reads, base pairs, classification rates) for a dashboard or QC table.

## When NOT to use

- The run is a completed batch analysis with a single final FASTQ/per-sample output rather than incrementally landing batches during sequencing
- You need taxonomic composition or pathogen watchlist alerts rather than read/base-pair QC statistics
- Input is already aggregated QC output (e.g. a MultiQC report) rather than raw per-batch FASTP JSON or Kraken2 reports

## Inputs

- Newly landing basecalled FASTQ batch files (batch_1, batch_2, ...) from a nanopore sequencer output directory
- Per-sample FASTP JSON reports (fastp_data['summary']['after_filtering'])
- Kraken2 classification reports with columns [%, cumul_reads, reads, rank, taxid, name]

## Outputs

- Per-batch statistics files written to results/realtime_batch_stats/batch_N (reads_in_batch, timestamp, base pairs, classified/unclassified counts)
- cumulative_qc_stats with running cumulative read counts, base pairs, and classification rates
- Per-sample QC table and bar charts rendered in the dashboard QC tab

## How to apply

Monitor the sequencer output directory and ingest each newly integrated basecalled FASTQ batch (batch_1, batch_2, ...). Run FASTP on each batch and read reads and base pairs from fastp_data['summary']['after_filtering']['total_reads'] in the per-sample fastp JSON reports. Run Kraken2 per batch and compute classified versus unclassified counts, deriving the unclassified count as the sum of reads with rank code 'U' from the concatenated report columns [%, cumul_reads, reads, rank, taxid, name]. Write each batch's statistics (reads_in_batch, timestamp, base pairs, classified/unclassified counts) to results/realtime_batch_stats/batch_N and accumulate them into cumulative_qc_stats, tracking both per-batch (noncumulative) and running cumulative values. Refresh the aggregation on a 10-second cadence to match the Dash interval component so newly integrated batches appear in the QC tab bar charts.

## Related tools

- **Nanometa Live** (Front-end application that orchestrates per-batch statistics extraction, accumulation into cumulative_qc_stats, and QC tab visualisation) — https://github.com/FOI-Bioinformatics/nanometa_live
- **FASTP** (Computes per-batch read counts and base pairs sequenced, parsed from the after_filtering section of its JSON reports)
- **Kraken2** (Classifies reads per batch; rank code 'U' lines are summed to derive unclassified read counts)
- **Dash** (Web framework whose 10-second interval component sets the refresh cadence for the per-batch QC bar charts)

## Examples

```
unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum(); total_reads = fastp_data['summary']['after_filtering']['total_reads']
```

## Evaluation signals

- The per-sample QC table row count matches the actual barcode count
- Cumulative read totals equal the sum of the individual realtime_batch_stats/batch_* read_in_batch values
- Each batch_N file contains reads_in_batch, timestamp, base pairs, and classified/unclassified counts
- Unclassified count equals the sum of reads_clade for rows where rank == 'U' in the Kraken2 report
- Newly integrated batches appear in the QC tab bar charts within one 10-second dashboard refresh

## Limitations

- Statistics reflect only batches integrated so far; metrics are provisional until the run ends
- Unclassified rate depends on the Kraken2 database choice (e.g. MiniKraken2 8 GB at k=12 default vs GTDB Bac120 ~25 GB at k=30), so classification rates are not comparable across databases
- Noncumulative per-batch values must be kept distinct from cumulative_qc_stats running totals to avoid double counting

## Evidence

- [methods] During sequencing, quality control data such as read counts and base pairs sequenced are extracted and processed for each new FASTQ batch integrated into the workflow: [evidence span withheld — anchor status: not_found]
- [results] unclassified rank code sum: "unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum()"
- [methods] realtime_batch_stats directory: "├── realtime_batch_stats/       # NEW: Real-time batch tracking"
- [results] Dash interval 10 seconds: "interval=10*1000,  # Update every 10 seconds"
- [other] fastp after_filtering parsing: [evidence span withheld — anchor status: not_found]

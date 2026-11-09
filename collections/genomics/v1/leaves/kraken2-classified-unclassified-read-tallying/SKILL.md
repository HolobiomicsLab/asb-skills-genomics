---
name: kraken2-classified-unclassified-read-tallying
description: Use when a sequencing run produces per-batch Kraken2 report output and
  you must compute per-batch and cumulative classified vs. unclassified read counts
  for real-time quality control.
when_to_use_negative:
- Input is a single finished sample rather than a live stream of FASTQ batches — a
  one-shot Kraken2 run with end-of-run report aggregation is simpler
- The task is taxonomic composition visualization (Sankey/sunburst) rather than classified-vs-unclassified
  read counting
- Reports come from a classifier whose output lacks a rank code column (e.g. raw Kraken2
  classifications without the report format), since the 'U' rank-code tally depends
  on Kraken2 report format
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: Kraken2
  role: Classifies reads per FASTQ batch; its report rank codes ('U', 'R') are the
    basis for tallying unclassified vs. classified reads
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: FASTP
  role: Provides per-batch read and base-pair counts from the summary/after_filtering
    section of its JSON reports to accompany the Kraken2 tallies
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: Nanometa Live
  role: Dash front end that aggregates the realtime_batch_stats output and renders
    per-batch and cumulative QC bar charts on a 10-second refresh
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
- name: nanometanf
  role: Nextflow backend performing real-time FASTQ monitoring via watchPath and per-batch
    Kraken2 classification with append-only batch storage
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: Dash
  role: Framework hosting the local web application whose interval component (10*1000
    ms) drives the QC tab refresh
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:692b175bccb98bda8a5b49f3e2bde9b22c8e47469697ede6d4e5ff5e3b4a58e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometa_live
  tools:
  - Dash
  - FASTP
  - Kraken2
  - Nanometa Live
  - nanometanf
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Kraken2 classified/unclassified read tallying

## Summary

Tallies Kraken2-classified versus unclassified reads from per-batch Kraken2 reports during an Oxford Nanopore sequencing run, feeding real-time batch QC statistics (reads, base pairs, classification rates) into the Nanometa Live QC dashboard.

## When to use

Use when a sequencing run produces per-batch Kraken2 report output and you must compute per-batch and cumulative classified vs. unclassified read counts for real-time quality control.

## When NOT to use

- Input is a single finished sample rather than a live stream of FASTQ batches — a one-shot Kraken2 run with end-of-run report aggregation is simpler
- The task is taxonomic composition visualization (Sankey/sunburst) rather than classified-vs-unclassified read counting
- Reports come from a classifier whose output lacks a rank code column (e.g. raw Kraken2 classifications without the report format), since the 'U' rank-code tally depends on Kraken2 report format

## Inputs

- Per-batch Kraken2 report files (columns: %, cumul_reads, reads, rank, taxid, name) generated during a Nanopore sequencing run
- Newly landed basecalled FASTQ batches (batch_1, batch_2, ...) from the sequencer output directory
- Per-sample FASTP JSON reports (summary.after_filtering.total_reads and total_bases)

## Outputs

- Per-batch statistics files at results/realtime_batch_stats/batch_N containing reads_in_batch, timestamp, base pairs, and classified/unclassified counts
- Cumulative QC statistics (cumulative_qc_stats) tracking running read counts, base pairs, and classification rates
- Per-sample QC bar charts in the Nanometa Live quality control dashboard tab

## How to apply

Run Kraken2 on each newly landing basecalled FASTQ batch (batch_1, batch_2, ...) ingested from the nanopore sequencer output directory during the run. Parse the concatenated Kraken2 report columns (%, cumul_reads, reads, rank, taxid, name) and compute the unclassified count as the sum of reads carrying rank code 'U'; classified reads are those with ranked codes (e.g. 'R'), giving per-batch classification rates. Pair these tallies with per-batch read and base-pair counts extracted from FASTP JSON reports (fastp_data['summary']['after_filtering']['total_reads']) and write each batch's statistics to results/realtime_batch_stats/batch_N. Accumulate per-batch rows into cumulative_qc_stats so both noncumulative and running totals are tracked, and refresh the aggregation on the dashboard's 10-second interval component so newly integrated batches appear in the QC tab bar charts. Rationale: per-batch (append-only) storage with cumulative aggregation mirrors nanometanf's streaming architecture, where each batch is written to its own file and cumulative output is rebuilt only at end of session, keeping per-batch accounting O(1).

## Related tools

- **Kraken2** (Classifies reads per FASTQ batch; its report rank codes ('U', 'R') are the basis for tallying unclassified vs. classified reads) — https://github.com/FOI-Bioinformatics/nanometa_live
- **FASTP** (Provides per-batch read and base-pair counts from the summary/after_filtering section of its JSON reports to accompany the Kraken2 tallies) — https://github.com/FOI-Bioinformatics/nanometa_live
- **Nanometa Live** (Dash front end that aggregates the realtime_batch_stats output and renders per-batch and cumulative QC bar charts on a 10-second refresh) — https://github.com/FOI-Bioinformatics/nanometa_live
- **nanometanf** (Nextflow backend performing real-time FASTQ monitoring via watchPath and per-batch Kraken2 classification with append-only batch storage) — https://github.com/FOI-Bioinformatics/nanometanf
- **Dash** (Framework hosting the local web application whose interval component (10*1000 ms) drives the QC tab refresh) — https://github.com/FOI-Bioinformatics/nanometa_live

## Examples

```
unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum()
```

## Evaluation signals

- Per-sample QC table row count matches the actual barcode count
- Cumulative read totals equal the sum of the individual results/realtime_batch_stats/batch_* reads_in_batch values
- unclassified + classified reads equals total reads in the batch for every batch
- Unclassified count computed from rank 'U' rows in the Kraken2 report matches the unclassified percentage column (first column) when re-multiplied by total reads
- QC tab bar charts reflect newly integrated batches within one 10-second dashboard refresh

## Limitations

- Unclassified-rate logic assumes Kraken2 report format with a rank column; the sum over rank 'U' rows is used rather than parsing a single unclassified line
- Cumulative outputs in nanometanf's streaming architecture are rebuilt only at end of session, so mid-run cumulative views depend on the per-batch accumulation layer
- Memory limits of the configured Kraken2 database (e.g. MiniKraken2 at 8 GB with k=12 default) affect classification rate and hence the classified/unclassified split

## Evidence

- [results] Unclassified rate computed from reads with rank 'U': "unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum()"
- [methods] Kraken2 classification is a core pipeline step in both pipeline versions: "| Kraken2 classification | yes | yes | Same |"
- [readme] Per-batch, append-only storage underpins the batch statistics layer: [evidence span withheld — anchor status: not_found]
- [readme] Real-time Kraken2 report monitoring is an advertised feature of the dashboard: [evidence span withheld — anchor status: external_doc]
- [results] The dashboard refresh cadence is set by the interval component: "interval=10*1000,  # Update every 10 seconds"

---
name: per-sample-kreport2-report-generation
description: Use when running real-time or batch Oxford Nanopore metagenomic analysis
  with Nanometa Live and you need per-barcode Kraken2 classification reports in results/kraken2/
  that the GUI and load_kraken_data consume.
when_to_use_negative:
- Input reads have already been classified and only .kreport2 files need re-parsing
  — use the GUI's load_kraken_data path instead of re-running classification
- The goal is strain-level validation of a specific detection rather than whole-sample
  taxonomic profiling — use BLAST/minimap2 validation with Nextflow -resume
- Running on short-read (Illumina) data outside the Oxford Nanopore nanopore_output_directory
  workflow this pipeline is built for
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kraken2
  role: performs the k-mer-based taxonomic classification that produces the .kreport2
    reports
- name: nanometanf
  role: Nextflow backend that orchestrates per-sample Kraken2 classification with
    watchPath monitoring and max_classification_forks concurrency
  repo: https://github.com/FOI-Bioinformatics/nanometanf
- name: Nanometa Live
  role: Dash front end that loads and visualises the cumulative per-sample Kraken2
    reports in real time
  repo: https://github.com/FOI-Bioinformatics/nanometa_live
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae108
    title: 'Nanometa Live: a user-friendly application for real-time metagenomic data
      analysis and pathogen identification'
  source_content_hash: sha256:ffe4eaf38e4048eaa264f93a8d11978b64894398af6fed042cb4be30594c745c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FOI-Bioinformatics/nanometanf
  tools:
  - Kraken2
  - Nanometa Live
  - Nextflow
  - nanometanf
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# per-sample-kreport2-report-generation

## Summary

Generates per-sample Kraken2 taxonomic classification reports (*.kreport2.txt) from real-time Oxford Nanopore FASTQ batches within the Nanometa Live / nanometanf pipeline, so the Dash GUI can display species-level read counts during sequencing.

## When to use

Use when running real-time or batch Oxford Nanopore metagenomic analysis with Nanometa Live and you need per-barcode Kraken2 classification reports in results/kraken2/ that the GUI and load_kraken_data consume.

## When NOT to use

- Input reads have already been classified and only .kreport2 files need re-parsing — use the GUI's load_kraken_data path instead of re-running classification
- The goal is strain-level validation of a specific detection rather than whole-sample taxonomic profiling — use BLAST/minimap2 validation with Nextflow -resume
- Running on short-read (Illumina) data outside the Oxford Nanopore nanopore_output_directory workflow this pipeline is built for

## Inputs

- Basecalled FASTQ files from an Oxford Nanopore MinION or Flongle run (nanopore_output_directory, flat or per-barcode layout)
- Kraken2 database directory with hash.k2d, opts.k2d, taxo.k2d (e.g. MiniKraken2 8 GB)
- YAML configuration with kraken_memory_mapping and kraken2_memory_gb parameters

## Outputs

- Per-sample Kraken2 reports results/kraken2/*.kreport2.txt with columns [%, cumul_reads, reads, rank, taxid, name]
- Species-level read counts aggregated per sample for GUI display and watchlist screening

## How to apply

Load the YAML configuration, set kraken_memory_mapping as a boolean (true) and size kraken2_memory_gb as on-disk database size + 4 GB (e.g. 12 for the 8 GB MiniKraken2 database, 30 for GTDB Bac120, 74 for Kraken2 PlusPF). Point the pipeline at the basecalled FASTQ input directory (nanopore_output_directory) and the Kraken2 database containing hash.k2d, opts.k2d, and taxo.k2d. Each real-time FASTQ batch is first filtered with FastP, then classified with Kraken2 via the nanometanf Nextflow backend under the conda profile; memory-mapped forks share the OS page cache with concurrency capped by max_classification_forks (default 8, or --kraken2_enable_incremental true for the streaming v1.5+ architecture on runs with more than ten barcodes). Per-sample classification reports are written to results/kraken2/ as *.kreport2.txt files with columns [%, cumul_reads, reads, rank, taxid, name], which are combined and consumed downstream by the GUI.

## Related tools

- **Kraken2** (performs the k-mer-based taxonomic classification that produces the .kreport2 reports)
- **nanometanf** (Nextflow backend that orchestrates per-sample Kraken2 classification with watchPath monitoring and max_classification_forks concurrency) — https://github.com/FOI-Bioinformatics/nanometanf
- **Nanometa Live** (Dash front end that loads and visualises the cumulative per-sample Kraken2 reports in real time) — https://github.com/FOI-Bioinformatics/nanometa_live

## Examples

```
nextflow run foi-bioinformatics/nanometanf --realtime_mode --kraken2_enable_incremental true --max_classification_forks 8 --nanopore_output_dir /path/to/fastq --kraken2_db /path/to/db --outdir results -profile conda
```

## Evaluation signals

- load_kraken_data parses each report into exactly the expected columns [%, cumul_reads, reads, rank, taxid, name]
- A *.kreport2.txt file exists in results/kraken2/ for every barcode present in the run
- Unclassified rate computed as reads with rank 'U' versus rank 'R' plus unclassified falls within plausible ranges for the sample type
- Real-time latency and per-batch update remain bounded (v2.0 target 22 s real-time latency benchmark)
- Cumulative reports grow monotonically during the sequencing run, matching new FASTQ batches observed by watchPath

## Limitations

- kraken2_memory_gb must be sized as on-disk database size + 4 GB; undersizing causes failures with larger databases such as Kraken2 PlusPFP (~80 GB, needs 84 GB)
- --max_concurrent_batches is currently advisory only, not an enforced per-barcode throttle (audit note P2.9)
- No tool versions specified for Kraken2 in the migration guide; behaviour depends on the installed dependency versions

## Evidence

- [methods] In the described workflow, each real-time FASTQ batch first undergoes initial filtering with FastP and is then classified with Kraken2, with per-sample Kraken2 reports combined to produce the taxonomic data: [evidence span withheld — anchor status: not_found]
- [readme] Real-time monitoring of cumulative Kraken2 reports during a sequencing run.: [evidence span withheld — anchor status: external_doc]
- [readme] Streaming Kraken2 architecture (v1.5+) with per-sample parallelism, append-only batch storage, and incremental taxid counting: [evidence span withheld — anchor status: external_doc]
- [results] unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum(): "unclassified = combined[combined['rank'] == 'U']['reads_clade'].sum()"
- [methods] Real-time FASTQ monitoring | yes | yes | Now uses native `watchPath`: "Real-time FASTQ monitoring | yes | yes | Now uses native `watchPath`"

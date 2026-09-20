---
name: nanopore-basecalling-with-dorado
description: Use when the starting point is a folder of ONT POD5 files (direct RNA,
  cDNA, or genomic DNA) that must be basecalled, optionally with modified bases, into
  a single unmapped BAM before alignment or modkit analysis.
when_to_use_negative:
- Input is already a merged unmapped BAM or mapped BAM — use the standalone kallisto,
  remap, or annotateRNA entry points instead of re-running basecalling
- Input is an existing CDNA FASTQ — use the fastq CDNA workflow, which creates an
  unmapped BAM without re-basecalling POD5
- Reads are not Oxford Nanopore POD5 data (e.g. Illumina FASTQ or PacBio data) — Dorado
  is specific to ONT reads
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Dorado
  role: Basecalling engine run on each POD5 file in parallel, with modified-base model
    support; models are downloaded automatically by Dogme
  repo: https://github.com/nanoporetech/dorado
- name: Dogme
  role: Nextflow pipeline orchestrating model download, parallel Dorado basecalling,
    and BAM merging
  repo: https://github.com/mortazavilab/dogme
- name: Nextflow
  role: Workflow engine in which the basecalling stage and its parallel POD5 processes
    are implemented
- name: samtools
  role: Tested companion (v1.15.1) for BAM handling/merging steps
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:666450e334d9e98e0fb50ac4b150de226d360411a6f1878523796607b74a21f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - Dogme
  - Dorado
  - Nextflow
  - samtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nanopore-basecalling-with-dorado

## Summary

Automates the basecalling layer of Oxford Nanopore reprocessing with Dogme/Nextflow: Dorado models are downloaded, each POD5 input is basecalled into a per-file BAM, and the per-POD5 BAMs are merged into a single unmapped BAM for downstream mapping and modification calling.

## When to use

Use when the starting point is a folder of ONT POD5 files (direct RNA, cDNA, or genomic DNA) that must be basecalled, optionally with modified bases, into a single unmapped BAM before alignment or modkit analysis.

## When NOT to use

- Input is already a merged unmapped BAM or mapped BAM — use the standalone kallisto, remap, or annotateRNA entry points instead of re-running basecalling
- Input is an existing CDNA FASTQ — use the fastq CDNA workflow, which creates an unmapped BAM without re-basecalling POD5
- Reads are not Oxford Nanopore POD5 data (e.g. Illumina FASTQ or PacBio data) — Dorado is specific to ONT reads

## Inputs

- Folder of raw Oxford Nanopore POD5 files
- Read type parameter: RNA (direct RNA), CDNA, or DNA
- Optional barcode kit name (kitName) for demultiplexing
- Dogme Nextflow config specifying top-level pod5 directory, output directory, and work directory

## Outputs

- One BAM per input POD5 file (unmapped, modified-base tags MM/ML present when modified basecalling is enabled)
- Single merged unmapped BAM (merged.bam) stored in the bams folder
- Optionally demultiplexed per-barcode BAMs named ${sample}.${barcode}

## How to apply

Run the Dogme Nextflow basecalling stage for the supplied read type — direct RNA ('RNA'), cDNA ('CDNA'), or genomic DNA ('DNA') — which first downloads the latest Dorado basecalling models (skipping the download if the model directory already exists). Dorado then basecalls the input folder of POD5 files in parallel, producing one BAM per POD5 input; Dorado supports modified basecalling, duplex basecalling, and POD5 input for highest performance. The per-POD5 BAMs are merged into a single unmapped BAM (merged.bam) in the bams folder, ready for minimap2 mapping and modkit modification extraction. Optionally, set kitName to have Dorado classify and demultiplex barcoded reads, emitting `${sample}.${barcode}` outputs that continue through barcode-aware downstream steps. Dogme 1.4.0 was tested with Dorado 1.2 alongside samtools 1.15.1, minimap2 2.28, and modkit 0.5.

## Related tools

- **Dorado** (Basecalling engine run on each POD5 file in parallel, with modified-base model support; models are downloaded automatically by Dogme) — https://github.com/nanoporetech/dorado
- **Dogme** (Nextflow pipeline orchestrating model download, parallel Dorado basecalling, and BAM merging) — https://github.com/mortazavilab/dogme
- **Nextflow** (Workflow engine in which the basecalling stage and its parallel POD5 processes are implemented)
- **samtools** (Tested companion (v1.15.1) for BAM handling/merging steps)

## Examples

```
nextflow run mortazavilab/dogme --entry basecall --readType RNA --pod5Dir /path/to/pod5 -profile dogme
```

## Evaluation signals

- merged.bam exists in the bams folder after the stage completes
- merged.bam is a valid BAM file (e.g. readable with samtools quickcheck/flagstat)
- merged.bam contains basecalled reads corresponding to all input POD5 files (read counts reconcile with the per-POD5 BAMs)
- Per-POD5 BAMs each contain MM/ML modified-base tags when a modified basecalling model was used
- Dogme's generate_report.py emits qc_summary.csv and inventory_report.tsv with per-BAM statistics that can be inspected for completeness

## Limitations

- Dorado model download is only run if the model directory does not already exist, so stale model directories will not be refreshed automatically
- No specific Dorado model versions or checksums are recorded in the article context; 'latest dorado models' is resolved at run time
- Unclassified reads in demultiplexed runs are published without downstream analysis
- Dorado basecalling requires compatible hardware (Apple silicon or Nvidia GPUs); Dogme was tested on SLURM clusters and Macs

## Evidence

- [intro] Given a folder of pod5 files, the pipeline downloads dorado models, runs dorado, merges bams, and maps them with minimap2: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [intro] Dogme supports 3 read types: RNA, CDNA, DNA: "Dogme supports 3 different read types: direct RNA ("RNA"), cDNA ("CDNA"), and genomic DNA ("DNA")."
- [results] Dogme basecalls individual POD5 files with Dorado in parallel, producing one BAM per POD5: [evidence span withheld — anchor status: not_found]
- [readme] dorado model download skipped if model directory exists: [evidence span withheld — anchor status: external_doc]
- [readme] Optional Dorado demultiplexing with kitName: [evidence span withheld — anchor status: external_doc]

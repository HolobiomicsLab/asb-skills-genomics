---
name: parallel-basecalling-of-pod5-batches
description: Use when the input is a folder of raw Oxford Nanopore POD5 files (direct
  RNA, cDNA, or genomic DNA) and the goal is GPU-parallel Dorado basecalling with
  modified-base tags, consolidated into a single unmapped BAM for downstream modkit/minimap2
  steps.
when_to_use_negative:
- Input is already basecalled (FASTQ or unmapped BAM) — use the Dogme 'remap' or 'kallisto'
  entry points instead of re-running basecalling
- Input is barcoded reads needing classification/demultiplexing as the primary goal
  — use Dogme's optional Dorado demultiplexing (kitName) workflow rather than plain
  parallel basecalling
- The data are not Oxford Nanopore POD5 files (e.g., Illumina FASTQ or PacBio data)
  — Dorado basecalling is specific to ONT reads
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: High-performance basecaller run in parallel on individual POD5 files to produce
    per-POD5 BAMs with modified-base tags; also downloads/loads the basecalling models
  repo: https://github.com/nanoporetech/dorado
- name: Dogme
  role: Nextflow pipeline orchestrating model download, parallel Dorado basecalling,
    and per-POD5 BAM merging with retry/error strategies
  repo: https://github.com/mortazavilab/dogme
- name: Nextflow
  role: Workflow engine that parallelizes the basecalling stage across POD5 files
    and manages restartability
- name: samtools
  role: Used at v1.15.1 in the tested Dogme stack for BAM handling during merging
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:b5b3e33071778e1bc7c42710cf9984f8ffc28934bea59e052e12f63aa99a74dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - Dogme
  - Dorado
  - Nextflow
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# parallel-basecalling-of-pod5-batches

## Summary

Automates the basecalling layer of ONT reprocessing in Dogme: download the latest Dorado models, basecall each POD5 file with Dorado in parallel to produce one BAM per POD5, then merge the per-POD5 BAMs into a single unmapped BAM for downstream mapping with minimap2.

## When to use

Use when the input is a folder of raw Oxford Nanopore POD5 files (direct RNA, cDNA, or genomic DNA) and the goal is GPU-parallel Dorado basecalling with modified-base tags, consolidated into a single unmapped BAM for downstream modkit/minimap2 steps.

## When NOT to use

- Input is already basecalled (FASTQ or unmapped BAM) — use the Dogme 'remap' or 'kallisto' entry points instead of re-running basecalling
- Input is barcoded reads needing classification/demultiplexing as the primary goal — use Dogme's optional Dorado demultiplexing (kitName) workflow rather than plain parallel basecalling
- The data are not Oxford Nanopore POD5 files (e.g., Illumina FASTQ or PacBio data) — Dorado basecalling is specific to ONT reads

## Inputs

- Folder of raw ONT POD5 files from MinION sequencing
- Dogme Nextflow config specifying read type ('RNA', 'CDNA', or 'DNA'), output directories, and work directory
- Dorado basecalling models (auto-downloaded by the pipeline if not already present)

## Outputs

- One basecalled BAM per input POD5 file, each carrying modified-base MM/ML tags
- Single merged unmapped BAM (merged.bam) stored in the bams folder, ready for minimap2 mapping

## How to apply

Run the Dogme Nextflow basecalling stage for the supplied read type — 'RNA' (direct RNA), 'CDNA', or 'DNA' — which triggers download of the latest Dorado basecalling models; per the README, the model download is skipped if the model directory already exists to avoid repeated downloads. Basecall the input POD5 folder with Dorado (v1.2 as tested in Dogme) in parallel, producing one BAM per POD5 input, leveraging Dorado's POD5 support, modified-basecalling, and automatic hardware detection across Nvidia GPUs or Apple silicon. Merge the per-POD5 BAMs into a single unmapped BAM (merged.bam) stored in the bams folder, using samtools (v1.15.1 as tested) as appropriate. The merged.bam is the handoff artifact: mapping with minimap2 and, for RNA and DNA, modkit modification extraction consume this file, so modified-base MM/ML tags must survive basecalling and merging. Rationale: parallelizing at POD5-file granularity exploits Dorado's multi-GPU linear scaling while keeping each process restartable under Nextflow retry/error strategies.

## Related tools

- **Dorado** (High-performance basecaller run in parallel on individual POD5 files to produce per-POD5 BAMs with modified-base tags; also downloads/loads the basecalling models) — https://github.com/nanoporetech/dorado
- **Dogme** (Nextflow pipeline orchestrating model download, parallel Dorado basecalling, and per-POD5 BAM merging with retry/error strategies) — https://github.com/mortazavilab/dogme
- **Nextflow** (Workflow engine that parallelizes the basecalling stage across POD5 files and manages restartability)
- **samtools** (Used at v1.15.1 in the tested Dogme stack for BAM handling during merging)

## Examples

```
nextflow run mortazavilab/dogme --readType 'RNA' --pod5Dir /path/to/pod5/ -profile standard
```

## Evaluation signals

- merged.bam exists in the bams folder and is a valid BAM file (readable by samtools)
- merged.bam contains basecalled reads corresponding to all input POD5 files (read counts reconcile with per-POD5 BAMs)
- MM/ML modified-base tags from Dorado are preserved through merging so downstream modkit extraction works
- One output BAM exists per input POD5 file before merging
- Dorado model download ran exactly once (skipped on subsequent runs when the model directory already exists)

## Limitations

- Dorado basecalling requires suitable hardware (Nvidia GPUs including multi-GPU, or Apple silicon M-series); CPU-only environments will be very slow
- Dorado model selection is automatic (latest models downloaded by Dogme), but no specific model versions or checksums are recorded, complicating exact reproducibility
- Parallelism is at POD5-file granularity, so a single very large POD5 file cannot be sharded across GPUs by this stage

## Evidence

- [intro] Given a folder of pod5 files, the pipeline downloads dorado models, runs dorado, merges bams, and maps them with minimap2: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [full_text] Dogme basecalls individual POD5 files with Dorado in parallel, producing one BAM per POD5: [evidence span withheld — anchor status: not_found]
- [readme] dorado model download is only run if the model directory does not already exist (avoids repeated downloads): [evidence span withheld — anchor status: external_doc]
- [readme] Dogme supports 3 different read types: direct RNA ("RNA"), cDNA ("CDNA"), and genomic DNA ("DNA"): "Dogme supports 3 different read types: direct RNA ("RNA"), cDNA ("CDNA"), and genomic DNA ("DNA")."
- [readme] Dorado runs on Nvidia GPUs including multi-GPU with linear scaling: [evidence span withheld — anchor status: external_doc]

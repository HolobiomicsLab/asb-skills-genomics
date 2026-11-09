---
name: bam-file-merging
description: Use when a basecalling stage has produced multiple per-input BAM files
  (one per POD5) that must be combined into one unmapped BAM before mapping or modification
  extraction.
when_to_use_negative:
- Input BAMs are already mapped/aligned — merging unmapped basecalled BAMs is specific
  to the pre-mapping stage; use remap-style entry points instead
- Only a single POD5/BAM exists, so no consolidation is needed
- The goal is to manipulate modified-base records in an aligned modBAM — use modkit
  rather than raw BAM merging
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: Produces the per-POD5 basecalled BAMs that are subsequently merged into a
    single unmapped BAM
  repo: https://github.com/nanoporetech/dorado
- name: Dogme
  role: Nextflow pipeline that orchestrates the merge-bams step and stores merged.bam
    in the bams folder before minimap2 mapping
  repo: https://github.com/mortazavilab/dogme
- name: Nextflow
  role: Workflow engine executing the basecalling and merge steps in parallel with
    retry/error strategies
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:04dd97f13630484fc5504c83589572bff504e3fe1a1c3ff3d3ecf9c79cffb2c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - Dogme
  - Dorado
  - Nextflow
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# bam-file-merging

## Summary

Consolidates the per-POD5 BAMs produced by parallel Dorado basecalling in the Dogme Nextflow pipeline into a single unmapped BAM (merged.bam) that serves as the input for downstream minimap2 mapping.

## When to use

Use when a basecalling stage has produced multiple per-input BAM files (one per POD5) that must be combined into one unmapped BAM before mapping or modification extraction.

## When NOT to use

- Input BAMs are already mapped/aligned — merging unmapped basecalled BAMs is specific to the pre-mapping stage; use remap-style entry points instead
- Only a single POD5/BAM exists, so no consolidation is needed
- The goal is to manipulate modified-base records in an aligned modBAM — use modkit rather than raw BAM merging

## Inputs

- One basecalled BAM per input POD5 file, produced by parallel Dorado basecalling
- Folder of raw ONT POD5 files (implicit reference set for completeness checking)

## Outputs

- A single unmapped BAM (merged.bam) stored in the bams folder, containing basecalled reads corresponding to all input POD5 files

## How to apply

After Dogme runs Dorado in parallel over the input folder of POD5 files, each POD5 yields its own basecalled BAM. Merge these per-POD5 BAMs into a single unmapped BAM (merged.bam) stored in the bams folder, using the pipeline's 'merge bams' step prior to mapping with minimap2. Because basecalling is parallelized across POD5 inputs, merging restores a single coherent read set representing all input files. Dogme's dorado model download is only run if the model directory does not already exist, and processes include retry/error strategies, so a re-run of the merge stage should tolerate long-running-task failures. The merged BAM is unmapped: modification extraction (modkit) and FASTQ extraction/kallisto all start from this consolidated file depending on read type (RNA, CDNA, or DNA).

## Related tools

- **Dorado** (Produces the per-POD5 basecalled BAMs that are subsequently merged into a single unmapped BAM) — https://github.com/nanoporetech/dorado
- **Dogme** (Nextflow pipeline that orchestrates the merge-bams step and stores merged.bam in the bams folder before minimap2 mapping) — https://github.com/mortazavilab/dogme
- **Nextflow** (Workflow engine executing the basecalling and merge steps in parallel with retry/error strategies)

## Evaluation signals

- merged.bam exists in the bams folder after the basecalling stage completes
- merged.bam is a valid BAM file (e.g., readable with samtools quickcheck/ view header parse)
- Read count in merged.bam corresponds to the sum of basecalled reads across all input POD5 files
- Every input POD5 file is represented — no per-POD5 BAM was dropped during the parallel merge
- merged.bam remains unmapped (no alignment records) since mapping with minimap2 is a separate downstream step

## Limitations

- Merging assumes the per-POD5 basecalling jobs all succeeded; partial failures require Dogme's retry/error strategies before the merged BAM is complete
- The merged BAM is unmapped — it cannot be used directly for modification calling with modkit or genome-browser inspection until minimap2 mapping is run
- Specific merging tool (e.g., samtools version 1.15.1 is listed as a Dogme prerequisite) and exact merge command are not recorded in the provided context

## Evidence

- [intro] Parallel basecalling produces per-POD5 BAMs merged into one unmapped BAM: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [abstract] Dogme automates basecalling and merging of ONT POD5 files: "Dogme automates the reprocessing of ONT POD5 files by integrating basecalling using Dorado"
- [full_text] Per-POD5 parallel basecalling followed by BAM merging (task_001 card): [evidence span withheld — anchor status: not_found]
- [readme] Dorado model download idempotency and process robustness in the pipeline: [evidence span withheld — anchor status: external_doc]

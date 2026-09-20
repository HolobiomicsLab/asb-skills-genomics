---
name: dorado-model-download-and-selection
description: Use when a Dogme run starts from a folder of raw POD5 files and the correct
  Dorado basecalling models are not yet present, or when reprocessing nanopore reads
  requires refreshed modification models.
when_to_use_negative:
- Input is already a basecalled unmapped BAM or FASTQ — skip basecalling/model selection
  and use the remap, kallisto, or reports entry points instead
- The Dorado model directory already exists for the run — Dogme deliberately skips
  the download step, so forcing a re-download would only add redundant I/O
tools:
- name: Dorado
  role: ONT basecalling engine whose models are downloaded/selected; run afterwards
    on individual POD5 files in parallel
  repo: https://github.com/nanoporetech/dorado
- name: Dogme
  role: Nextflow pipeline that orchestrates the model download, guards it against
    redundant re-download, and records versions
  repo: https://github.com/mortazavilab/dogme
- name: Nextflow
  role: Workflow engine implementing the basecall stage that triggers model download
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag066
    title: 'Dogme: a nextflow pipeline for reprocessing nanopore RNA and DNA modifications'
  source_content_hash: sha256:5a6d831da5762f5aa59a0c7582adebfcbf397994b3ecd11674e2f10c6cceb9b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - Dogme
  - Dorado
  - Nextflow
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dorado-model-download-and-selection

## Summary

Acquires and selects the latest Dorado basecalling models within the Dogme Nextflow pipeline so that ONT POD5 files can be basecalled for RNA, CDNA, or DNA read types with up-to-date modification-aware models.

## When to use

Use when a Dogme run starts from a folder of raw POD5 files and the correct Dorado basecalling models are not yet present, or when reprocessing nanopore reads requires refreshed modification models.

## When NOT to use

- Input is already a basecalled unmapped BAM or FASTQ — skip basecalling/model selection and use the remap, kallisto, or reports entry points instead
- The Dorado model directory already exists for the run — Dogme deliberately skips the download step, so forcing a re-download would only add redundant I/O

## Inputs

- Folder of ONT POD5 raw read files
- Dogme Nextflow config (readType: RNA, CDNA, or DNA; paths to top-level pod5 directory and outputs)
- Empty or existing Dorado model directory

## Outputs

- Downloaded Dorado basecalling model directory
- Recorded software/model versions in ${sample}.softwareVersion.txt (via scripts/software_versions.py)

## How to apply

As the first step of the Dogme basecalling stage, trigger download of the latest Dorado basecalling models for the supplied read type (RNA, CDNA, or DNA). Dogme only runs the model download when the model directory does not already exist, avoiding repeated downloads across restarts of the pipeline; if a model directory is already populated, reuse it rather than re-downloading. The selected models are then used by the subsequent parallel Dorado basecalling of individual POD5 files, and the versions used are recorded by scripts/software_versions.py into ${sample}.softwareVersion.txt. Run the Nextflow basecall entry point so download, basecalling, and BAM merging proceed as one validated stage.

## Related tools

- **Dorado** (ONT basecalling engine whose models are downloaded/selected; run afterwards on individual POD5 files in parallel) — https://github.com/nanoporetech/dorado
- **Dogme** (Nextflow pipeline that orchestrates the model download, guards it against redundant re-download, and records versions) — https://github.com/mortazavilab/dogme
- **Nextflow** (Workflow engine implementing the basecall stage that triggers model download)

## Examples

```
nextflow run mortazavilab/dogme -entry basecall --readType RNA
```

## Evaluation signals

- Model directory is populated (dogme skips the download only when the model directory already exists)
- scripts/software_versions.py reports the Dorado and model versions used in ${sample}.softwareVersion.txt
- Downstream Dorado basecalling produces one BAM per POD5 input, confirming the selected models ran successfully
- Merged BAM validates as a valid BAM file containing basecalled reads corresponding to all input POD5 files

## Limitations

- Dorado model selection is automatic — no specific model versions or checksums are recorded in the context, so exact model provenance must be recovered from software_versions.py output
- The download step is idempotent by design: it only runs when the model directory is absent, so a stale model directory will silently prevent model refresh
- Model availability depends on the installed Dorado version (Dogme tested with dorado 1.2)

## Evidence

- [intro] Model download precedes basecalling in the pipeline flow: "Given a folder of pod5 files, Dogme will download the latest dorado models, run dorado, merge bams, and map them with minimap2."
- [readme] Idempotent download guard in Dogme 1.4.0: [evidence span withheld — anchor status: external_doc]
- [intro] Read types determine which models are used: "Dogme supports 3 different read types: direct RNA ("RNA"), cDNA ("CDNA"), and genomic DNA ("DNA")."
- [readme] Version recording script exists for provenance: [evidence span withheld — anchor status: external_doc]

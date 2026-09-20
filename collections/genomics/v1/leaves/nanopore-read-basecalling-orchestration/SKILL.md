---
name: nanopore-read-basecalling-orchestration
description: Use when raw nanopore signal files (FAST5, .fast5.gz, POD5, .pod5.gz)
  or already-basecalled FASTQ (.fastq, .fq, .fastq.gz, .fq.gz) must enter VirDetector's
  preprocessing before taxonomic classification.
when_to_use_negative:
- Input is already basecalled FASTQ and basecalling is disabled — the pipeline will
  only preprocess and trim
- Input is FASTA (.fasta, .fa, etc.) — VirDetector accepts these only from the alignment
  step onwards, not for basecalling
- A non-Linux environment is used without the Docker installation route
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Dorado
  role: GPU basecaller selectable via params.basecaller in nextflow.config for converting
    FAST5/POD5 signal files to FASTQ
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: CPU basecaller selectable via params.basecaller in nextflow.config, an alternative
    to Dorado
- name: Nextflow
  role: Workflow engine executing `nextflow run main.nf` to orchestrate basecalling
    and per-sample filtering
- name: VirDetector
  role: Pipeline wrapping the preprocessing stage; installs Dorado/Guppy and defines
    the *input* directory sample convention
  repo: https://github.com/NLKaiser/VirDetector
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:7600722a2a2dbe51a744df8e1b522c3060bc70bb574e8f3aa40301c047d205d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/dorado
  tools:
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  - nanopore sequencer
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nanopore-read-basecalling-orchestration

## Summary

Orchestrate optional basecalling of Oxford Nanopore signal files (FAST5/POD5) into FASTQ within VirDetector's Nextflow preprocessing stage, choosing Dorado (GPU) or Guppy (CPU), followed by NanoFilt quality filtering and NanoComp read-statistics visualisation.

## When to use

Use when raw nanopore signal files (FAST5, .fast5.gz, POD5, .pod5.gz) or already-basecalled FASTQ (.fastq, .fq, .fastq.gz, .fq.gz) must enter VirDetector's preprocessing before taxonomic classification.

## When NOT to use

- Input is already basecalled FASTQ and basecalling is disabled — the pipeline will only preprocess and trim
- Input is FASTA (.fasta, .fa, etc.) — VirDetector accepts these only from the alignment step onwards, not for basecalling
- A non-Linux environment is used without the Docker installation route

## Inputs

- FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz), zipped or unzipped
- FAST5 files (.fast5, .fast5.gz)
- POD5 files (.pod5, .pod5.gz)
- nextflow.config with params.basecaller ('dorado' or 'guppy'), flowcell type, and kit specification

## Outputs

- Basecalled FASTQ files per sample (when signal-space input is used)
- Trimmed/filtered FASTQ file for every sample (NanoFilt)
- NanoComp read-statistics plots comparing unfiltered vs. filtered reads

## How to apply

Place input files into VirDetector's *input* directory, treating each top-level file as a separate sample and files inside subdirectories as one combined sample. In *nextflow.config*, set params.basecaller to 'dorado' or 'guppy' and specify the flowcell type and kit if basecalling is required; disable basecalling for pure FASTQ input. Run `nextflow run main.nf` so Nextflow orchestrates basecalling (Dorado on GPU or Guppy on CPU) followed by per-sample NanoFilt filtering and NanoComp quality plots comparing unfiltered vs. filtered reads. Basecalling can be skipped when testing all other steps of the pipeline.

## Related tools

- **Dorado** (GPU basecaller selectable via params.basecaller in nextflow.config for converting FAST5/POD5 signal files to FASTQ) — https://github.com/nanoporetech/dorado
- **Guppy** (CPU basecaller selectable via params.basecaller in nextflow.config, an alternative to Dorado)
- **Nextflow** (Workflow engine executing `nextflow run main.nf` to orchestrate basecalling and per-sample filtering)
- **VirDetector** (Pipeline wrapping the preprocessing stage; installs Dorado/Guppy and defines the *input* directory sample convention) — https://github.com/NLKaiser/VirDetector

## Examples

```
conda activate VirDetector; nextflow run main.nf
```

## Evaluation signals

- The results directory contains basecalled FASTQ files when signal-space input was provided
- A trimmed FASTQ file exists for every sample listed in the input directory (or metadata file, if specified)
- Read-statistics plot images comparing unfiltered vs. filtered reads are present in the results directory
- params.basecaller is set consistently: 'dorado' or 'guppy' with flowcell and kit specified, or basecalling disabled for FASTQ input
- Top-level files in *input* yielded separate samples while each subdirectory was combined into a single sample

## Limitations

- Testing all pipeline steps with `nextflow run main.nf` excludes basecalling by default
- Installing VirDetector with Conda includes both basecallers Guppy and Dorado and can take several hours depending on computing resources; users who do not wish to install them must remove the respective parameter
- If a metadata file is specified, only samples defined in that file are passed to subsequent processing steps

## Evidence

- [readme] Accepted input formats before basecalling: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] Basecaller selection parameter: "To select the basecaller to be used, set **params.basecaller** to either "dorado" or "guppy""
- [methods] Included basecallers: "The installation includes the basecallers Guppy and Dorado. If you do not wish to install these, please remove the respective parameter"
- [methods] Pipeline execution command: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:
``nnextflow run main.nf
"
- [abstract] Preprocessing precedes classification: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically"

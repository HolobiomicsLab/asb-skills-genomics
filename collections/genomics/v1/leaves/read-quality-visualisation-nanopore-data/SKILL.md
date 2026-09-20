---
name: read-quality-visualisation-nanopore-data
description: Use when preprocessing raw nanopore output (FASTQ .fastq/.fq/.fastq.gz/.fq.gz,
  FAST5, or POD5) that requires per-sample read filtering plus comparative quality
  visualisations before taxonomic classification or genome reconstruction.
when_to_use_negative:
- Input consists only of FASTA files (.fasta, .fas, .fsa, .fna, .fa), which VirDetector
  uses from the alignment step onwards and do not enter the basecalling/quality-filtering
  stage
- Samples have already been basecalled and quality-filtered and only downstream steps
  (classification, consensus reconstruction, phylogeny) are needed
- All pipeline steps are disabled — samples will only be preprocessed and trimmed,
  so no downstream quality comparison is produced
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: VirDetector
  role: Pipeline whose preprocessing stage runs NanoFilt filtering and NanoComp quality
    plotting
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine orchestrating basecalling and per-sample read filtering (nextflow
    run main.nf)
- name: Dorado
  role: GPU basecaller applied before filtering when input is FAST5/POD5
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: CPU basecaller alternative applied before filtering when input requires basecalling
- name: nanopore sequencer
  role: Source instrument producing the raw nanopore reads entering the preprocessing
    stage
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:89e955cb1677b3174e5a84a47df153bc7403889be7358b1131a6ad95fdcc7684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  - nanopore sequencer
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# read-quality-visualisation-nanopore-data

## Summary

Filter Oxford Nanopore reads with NanoFilt after optional basecalling and use NanoComp to generate quality plots comparing unfiltered and filtered FASTQ reads, as done in VirDetector's preprocessing stage (section 2.1).

## When to use

Use when preprocessing raw nanopore output (FASTQ .fastq/.fq/.fastq.gz/.fq.gz, FAST5, or POD5) that requires per-sample read filtering plus comparative quality visualisations before taxonomic classification or genome reconstruction.

## When NOT to use

- Input consists only of FASTA files (.fasta, .fas, .fsa, .fna, .fa), which VirDetector uses from the alignment step onwards and do not enter the basecalling/quality-filtering stage
- Samples have already been basecalled and quality-filtered and only downstream steps (classification, consensus reconstruction, phylogeny) are needed
- All pipeline steps are disabled — samples will only be preprocessed and trimmed, so no downstream quality comparison is produced

## Inputs

- FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz), zipped or unzipped
- FAST5 files (.fast5, .fast5.gz) requiring basecalling
- POD5 files (.pod5, .pod5.gz) requiring basecalling

## Outputs

- Basecalled FASTQ files (when basecalling is enabled)
- A trimmed/filtered FASTQ file for every sample (NanoFilt output)
- NanoComp read-statistics quality plot images comparing unfiltered vs. filtered reads

## How to apply

Place each input file as a separate sample in the pipeline's *input* directory (files in a subdirectory are combined into one sample). If the data requires basecalling, set params.basecaller in nextflow.config to 'dorado' (GPU) or 'guppy' (CPU), specifying flowcell type and kit; for FASTQ input, disable basecalling. Run `nextflow run main.nf` so Nextflow orchestrates optional basecalling followed by per-sample NanoFilt read filtering. NanoComp is then used to generate quality plots comparing the unfiltered and filtered reads, which are collected from the results directory alongside the trimmed/filtered FASTQ per sample.

## Related tools

- **VirDetector** (Pipeline whose preprocessing stage runs NanoFilt filtering and NanoComp quality plotting) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine orchestrating basecalling and per-sample read filtering (nextflow run main.nf))
- **Dorado** (GPU basecaller applied before filtering when input is FAST5/POD5) — https://github.com/nanoporetech/dorado
- **Guppy** (CPU basecaller alternative applied before filtering when input requires basecalling)
- **nanopore sequencer** (Source instrument producing the raw nanopore reads entering the preprocessing stage)

## Examples

```
conda activate VirDetector && nextflow run main.nf
```

## Evaluation signals

- The results directory contains basecalled FASTQ files (when basecalling was enabled)
- The results directory contains a trimmed FASTQ file for every sample
- The results directory contains read-statistics plot images comparing unfiltered vs. filtered reads (NanoComp output)
- Files placed in the *input* directory were processed as separate samples; files in a subdirectory were combined into one sample
- The VirDetector Conda environment was active (`conda activate VirDetector`) and nextflow.config's params.basecaller matches whether input required basecalling

## Limitations

- Basecalling must be explicitly enabled/disabled and parameterised (flowcell type and kit) in nextflow.config; wrong settings cause basecalling failures
- If a metadata file is specified, only samples defined in it are passed on to subsequent processing steps
- The installation including Guppy and Dorado can take several hours depending on available computing resources

## Evidence

- [other] After optional basecalling (Dorado on GPU or CPU-based Guppy), each sample is filtered with NanoFilt, and NanoComp is used to generate quality plots comparing the unfiltered and filtered reads.: [evidence span withheld — anchor status: not_found]
- [readme] VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input.: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] To select the basecaller to be used, set params.basecaller to either "dorado" or "guppy": "Example: To select the basecaller to be used, set **params.basecaller** to either "dorado" or "guppy""
- [readme] If you disable all steps, your samples will only be preprocessed and trimmed.: "Select the individual steps you want to execute. If you disable all steps, your samples will only be preprocessed and trimmed."

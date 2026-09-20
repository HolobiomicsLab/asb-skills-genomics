---
name: metagenomic-taxonomic-classification
description: Use when raw nanopore read data (FASTQ, FAST5, or POD5, optionally gzipped)
  from a suspected viral sample must be assigned a taxonomic identity as part of a
  virus surveillance workflow.
when_to_use_negative:
- The goal is de novo taxonomic profiling of complex metagenomes beyond virus detection
  — VirDetector is a targeted virus-surveillance pipeline, not a general-purpose classifier.
- Samples are already assembled consensus genomes in FASTA form and only phylogenetic
  placement is needed — these enter the pipeline from the alignment step onwards,
  not classification.
- The sequencing platform is not Oxford Nanopore — the pipeline is designed for nanopore
  sequencing data.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: VirDetector
  role: Pipeline that performs the taxonomic classification of each nanopore sample
    after preprocessing, invoked with a single Nextflow command
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine that executes the classification and downstream steps via
    'nextflow run main.nf'
  repo: https://github.com/NLKaiser/VirDetector
- name: Docker
  role: Alternative installation route for running VirDetector (and its classification
    step) on non-Linux operating systems
  repo: https://docs.docker.com/engine/install/
- name: Dorado
  role: Optional basecaller installed with the pipeline, used only if basecalling
    of raw signal files is enabled before classification
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: Alternative optional basecaller installed with the pipeline, selected via
    params.basecaller when basecalling precedes classification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:9a748ea6abddb06bffbb51a692f698def1de4a65a1d9c7079fe91ad0a7510474
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Conda
  - Docker
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# metagenomic-taxonomic-classification

## Summary

Taxonomic classification of nanopore sequencing samples as implemented in the VirDetector pipeline, which classifies each sample taxonomically after preprocessing and uses the detected virus identity to drive downstream consensus genome reconstruction and phylogenetic analyses.

## When to use

Use when raw nanopore read data (FASTQ, FAST5, or POD5, optionally gzipped) from a suspected viral sample must be assigned a taxonomic identity as part of a virus surveillance workflow.

## When NOT to use

- The goal is de novo taxonomic profiling of complex metagenomes beyond virus detection — VirDetector is a targeted virus-surveillance pipeline, not a general-purpose classifier.
- Samples are already assembled consensus genomes in FASTA form and only phylogenetic placement is needed — these enter the pipeline from the alignment step onwards, not classification.
- The sequencing platform is not Oxford Nanopore — the pipeline is designed for nanopore sequencing data.

## Inputs

- FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz) placed in the input directory
- FAST5 files (.fast5, .fast5.gz)
- POD5 files (.pod5, .pod5.gz)
- FASTA files (.fasta, .fas, .fsa, .fna, .fa) — used from the alignment step onwards
- Optional metadata .txt file in resources/ (restricts processing to listed samples and adds lineage/country/date)

## Outputs

- Taxonomic classification report for each sample
- Trimmed FASTQ per sample in structured results directories
- Viral consensus genome FASTA files
- Top-level results.txt summarising visual outputs and key results

## How to apply

Install VirDetector via Conda with 'bash install/install.sh --guppy --dorado' (or Docker via 'bash install/docker_install.sh') and activate the environment with 'conda activate VirDetector'. Place input files in the 'input' directory — files there are treated as separate samples, while files in subdirectories are combined into one sample — and run 'nextflow run main.nf' to execute all steps except basecalling. After optional basecalling and read preprocessing/trimming, the pipeline classifies each sample taxonomically before reconstructing viral consensus genomes. Classification results are delivered in the structured results directories as a taxonomic classification report alongside trimmed FASTQ files, mapping files, and consensus genomes; the top-level results.txt summarises the most important visual results.

## Related tools

- **VirDetector** (Pipeline that performs the taxonomic classification of each nanopore sample after preprocessing, invoked with a single Nextflow command) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine that executes the classification and downstream steps via 'nextflow run main.nf') — https://github.com/NLKaiser/VirDetector
- **Docker** (Alternative installation route for running VirDetector (and its classification step) on non-Linux operating systems) — https://docs.docker.com/engine/install/
- **Dorado** (Optional basecaller installed with the pipeline, used only if basecalling of raw signal files is enabled before classification) — https://github.com/nanoporetech/dorado
- **Guppy** (Alternative optional basecaller installed with the pipeline, selected via params.basecaller when basecalling precedes classification)

## Examples

```
conda activate VirDetector && nextflow run main.nf
```

## Evaluation signals

- The taxonomic classification report identifies the expected virus (e.g. West Nile virus in the shipped test dataset) for the input sample.
- Downstream consensus genome FASTA files for the classified virus are generated in the assembly output directory, confirming classification fed the assembly stage.
- Trimmed FASTQ files per sample appear in the structured results directories, showing preprocessing fed into classification.
- A top-level results.txt exists and summarises the most important results of the run.

## Limitations

- Basecalling is excluded from the default test execution ('nextflow run main.nf' tests all steps except basecalling) and must be explicitly enabled by setting params.basecaller to 'dorado' or 'guppy' with flowcell and kit specifications.
- If a metadata file is specified in nextflow.config, only the samples defined in that file are passed on to subsequent processing steps — unlisted samples are silently dropped.
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems must use the Docker installation, and the Conda installation can take several hours.

## Evidence

- [abstract] After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"
- [readme] VirDetector accepts zipped and unzipped files in FASTQ format as input, placed in the input directory: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] Metadata file restricts which samples proceed to subsequent processing: "If you specify a metadata file, only those samples are passed on to subsequent processing steps that are defined in this file!"
- [methods] Test command executes all steps except basecalling: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:
``nnextflow run main.nf
"
- [readme] The repository contains a West Nile virus test input in FASTQ format with a reference sequence: "The repository contains a test input file of the West Nile virus in FASTQ format, as well as a reference sequence."

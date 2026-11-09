---
name: nanopore-read-preprocessing-and-trimming
description: Use when raw nanopore sequencing files (FASTQ, .fastq.gz, FAST5, or POD5)
  must be basecalled, preprocessed, and primer-trimmed before viral genome reconstruction
  or taxonomic classification in VirDetector.
when_to_use_negative:
- Input is already assembled consensus genomes or clean FASTA sequences — these enter
  the pipeline at the alignment step, not preprocessing
- You only need to test pipeline execution without basecalling on raw signal data
  — run 'nextflow run main.nf' which skips basecalling
- Data is Illumina or other non-nanopore short-read sequencing — VirDetector is designed
  for nanopore sequencing
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: VirDetector
  role: Pipeline that orchestrates preprocessing and trimming of nanopore reads in
    a single Nextflow workflow
  repo: https://github.com/NLKaiser/VirDetector.git
- name: Guppy
  role: Optional basecaller installed with the pipeline and selected via params.basecaller
    for raw POD5/FAST5 input
- name: Dorado
  role: Alternative optional basecaller installed with the pipeline and selected via
    params.basecaller
  repo: https://github.com/nanoporetech/dorado
- name: Nextflow
  role: Workflow engine that executes the preprocessing and trimming steps via 'nextflow
    run main.nf'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:087cb75b7de410ddfe4128fe6449725cb09bc63d6cb4e04ba14a65390259d822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector.git
  tools:
  - Conda
  - Docker
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# nanopore-read-preprocessing-and-trimming

## Summary

Preprocess and trim Oxford Nanopore reads (FASTQ/FAST5/POD5) as the first stage of the VirDetector virus-surveillance pipeline, producing per-sample trimmed FASTQ files ready for reference-based or hybrid assembly and taxonomic classification.

## When to use

Use when raw nanopore sequencing files (FASTQ, .fastq.gz, FAST5, or POD5) must be basecalled, preprocessed, and primer-trimmed before viral genome reconstruction or taxonomic classification in VirDetector.

## When NOT to use

- Input is already assembled consensus genomes or clean FASTA sequences — these enter the pipeline at the alignment step, not preprocessing
- You only need to test pipeline execution without basecalling on raw signal data — run 'nextflow run main.nf' which skips basecalling
- Data is Illumina or other non-nanopore short-read sequencing — VirDetector is designed for nanopore sequencing

## Inputs

- FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz), zipped or unzipped
- FAST5 files (.fast5, .fast5.gz)
- POD5 files (.pod5, .pod5.gz)
- FASTA files (.fasta, .fas, .fsa, .fna, .fa) — used from the alignment step onwards
- Forward and reverse primer sequences in FASTA format (for primer trimming)
- Optional metadata .txt file (sample renaming/filtering)

## Outputs

- Trimmed FASTQ file per sample
- Basecalled reads (when params.basecaller is set to 'dorado' or 'guppy')
- Preprocessed samples passed to taxonomic classification and genome reconstruction

## How to apply

Place input files in the 'input' directory of the cloned VirDetector repository; files in subdirectories are automatically combined into one sample. Install the pipeline via 'bash install/install.sh --guppy --dorado' (Conda) or 'bash install/docker_install.sh' (Docker), and activate the environment with 'conda activate VirDetector'. Select a basecaller (params.basecaller = 'dorado' or 'guppy') only if raw signal data (POD5/FAST5) requires basecalling; the pipeline's test run with 'nextflow run main.nf' executes all steps except basecalling. For primer trimming, set params.trim_primers = true and supply forward and/or reverse primer sequences in separate FASTA files via params.forward_primers and params.reverse_primers. Optionally restrict processing to specific samples by providing a metadata file via params.metadata — only samples defined in that file are passed to subsequent steps. The output is trimmed FASTQ per sample, which feeds the taxonomic classification and consensus-genome reconstruction stages.

## Related tools

- **VirDetector** (Pipeline that orchestrates preprocessing and trimming of nanopore reads in a single Nextflow workflow) — https://github.com/NLKaiser/VirDetector.git
- **Guppy** (Optional basecaller installed with the pipeline and selected via params.basecaller for raw POD5/FAST5 input)
- **Dorado** (Alternative optional basecaller installed with the pipeline and selected via params.basecaller) — https://github.com/nanoporetech/dorado
- **Nextflow** (Workflow engine that executes the preprocessing and trimming steps via 'nextflow run main.nf')

## Examples

```
bash install/install.sh --guppy --dorado && conda activate VirDetector && nextflow run main.nf
```

## Evaluation signals

- A trimmed FASTQ file is produced for each sample placed in the input directory (or combined per subdirectory sample)
- The Conda environment 'VirDetector' activates without errors after 'bash install/install.sh --guppy --dorado'
- 'nextflow run main.nf' completes and results.txt is generated at the top level summarising main results
- When primer trimming is enabled, primer sequences are absent from the trimmed reads
- When a metadata file is set via params.metadata, only samples defined in that file appear in downstream outputs

## Limitations

- Installation including the Guppy and Dorado basecallers can take several hours depending on available computing resources; parameters for unused basecallers must be removed explicitly
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation
- The Conda environment must be activated ('conda activate VirDetector') before running the pipeline
- The repository's test run exercises all steps except basecalling

## Evidence

- [abstract] After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"
- [methods] Primers can be trimmed by specifying forward and reverse primer sequences in separate files in FASTA format.: "Primers can be trimmed by specifying forward and reverse primer sequences in separate files in FASTA format."
- [readme] VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input.: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] If the metadata file path is specified in the configuration file, only those samples are passed on to subsequent processing steps that are defined in this file!: "If the metadata file path is specified in the configuration file, only those samples are passed on to subsequent processing steps that are defined in this file!"

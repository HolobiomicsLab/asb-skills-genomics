---
name: nextflow-pipeline-configuration
description: 'Use when running VirDetector on nanopore data and you must select which
  steps execute or supply required settings in nextflow.config: params.ref for reference-based
  assembly, params.canu_genome_size (e.g. "11k") for hybrid assembly, params.'
when_to_use_negative:
- Input consists of already-assembled FASTA sequences only with no read preprocessing
  needed — while FASTA input is accepted from the alignment step onwards, read-oriented
  parameters (basecaller, trimming) do not apply.
- You need to change pipeline logic itself rather than its behaviour — VirDetector
  explicitly states nextflow.config is the only file you have to edit, so code changes
  fall outside this skill's scope.
- You are on a non-Linux operating system without the Docker installation — VirDetector
  was developed and tested under Ubuntu 22.04 LTS and expects Linux or Docker.
edam_topics:
- http://edamontology.org/topic_0769
tools:
- name: VirDetector
  role: The Nextflow-orchestrated pipeline whose behaviour is customised via nextflow.config
    parameters
  repo: https://github.com/NLKaiser/VirDetector.git
- name: Nextflow
  role: Workflow engine executing main.nf; configuration is passed through params.*
    entries in nextflow.config, and `nextflow run main.nf --help` inspects key settings
- name: Conda
  role: Installs VirDetector and all required programs/databases; the VirDetector
    environment must be activated before running the configured pipeline
- name: Docker
  role: Alternative installation route (bash install/docker_install.sh) with its own
    config-free test invocation `bash VirDetector_docker.sh`
- name: Dorado
  role: GPU basecaller selectable via params.basecaller = "dorado"; installation can
    be omitted by removing the parameter from the install command
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: CPU basecaller selectable via params.basecaller = "guppy", with flowcell type
    and kit specified
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:64ac5550b6bbe8f64dc1ca4da62e5c48261c685c82131657df4bc0956b7b7bc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector.git
  tools:
  - Canu
  - Conda
  - Docker
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# nextflow-pipeline-configuration

## Summary

Configure VirDetector's behaviour entirely through its nextflow.config file — enabling/disabling pipeline steps (basecalling, reference-based vs. hybrid assembly, phylogeny, structural variant detection) and setting required parameters such as params.ref, params.canu_genome_size, and params.basecaller — before launching the pipeline with `nextflow run main.nf`.

## When to use

Use when running VirDetector on nanopore data and you must select which steps execute or supply required settings in nextflow.config: params.ref for reference-based assembly, params.canu_genome_size (e.g. "11k") for hybrid assembly, params.basecaller (dorado/guppy), primer trimming, alignment tool, or outgroup selection.

## When NOT to use

- Input consists of already-assembled FASTA sequences only with no read preprocessing needed — while FASTA input is accepted from the alignment step onwards, read-oriented parameters (basecaller, trimming) do not apply.
- You need to change pipeline logic itself rather than its behaviour — VirDetector explicitly states nextflow.config is the only file you have to edit, so code changes fall outside this skill's scope.
- You are on a non-Linux operating system without the Docker installation — VirDetector was developed and tested under Ubuntu 22.04 LTS and expects Linux or Docker.

## Inputs

- nextflow.config file (the sole configuration file to edit)
- Input reads in the *input* directory: FASTQ (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 (.fast5, .fast5.gz), or POD5 (.pod5, .pod5.gz); FASTA (.fasta, .fas, .fsa, .fna, .fa) accepted from the alignment step onwards
- Reference sequence FASTA in the resources folder (for params.ref)
- Forward/reverse primer FASTA files (for params.trim_primers)
- Metadata .txt file in resources (for params.metadata; only samples defined in it are processed)
- VirDetector reference database database/sequences.fasta (accession used for outgroup selection)

## Outputs

- A configured pipeline execution producing structured results directories: basecalled/trimmed FASTQ per sample, mapping files, consensus genome FASTAs, tree files and visualisations, taxonomic classification report, and a top-level results.txt
- Nextflow report files (optional, toggleable in config)

## How to apply

Treat nextflow.config as the only file to edit: enable or disable modules with flags like params.assembly_hybrid = "true"/"false", and note that disabling all steps leaves only preprocessing and trimming. For reference-based assembly, place a reference FASTA in the resources folder and set params.ref = "${PWD}/resources/{your_file_name}.fasta"; for hybrid assembly, set params.canu_genome_size to the expected genome size (e.g. "11k" for the ~11,000-base West Nile virus genome). If basecalling is needed, set params.basecaller to "dorado" or "guppy" and specify flowcell type and kits for demultiplexing. Optional customisations include primer trimming via separate forward/reverse primer FASTA files (params.trim_primers = true, params.forward_primers, params.reverse_primers), choosing MAFFT (faster) or MUSCLE (more precise, more resources) via params.alignment_tool, renaming/filtering samples through params.metadata, and passing tool-specific arguments such as params.iqtree_args or params.mrbayes_args (e.g. outgroup NC_009942.1 from database/sequences.fasta for West Nile virus). Activate the environment with `conda activate VirDetector` and inspect current settings with `nextflow run main.nf --help` before executing `nextflow run main.nf`.

## Related tools

- **VirDetector** (The Nextflow-orchestrated pipeline whose behaviour is customised via nextflow.config parameters) — https://github.com/NLKaiser/VirDetector.git
- **Nextflow** (Workflow engine executing main.nf; configuration is passed through params.* entries in nextflow.config, and `nextflow run main.nf --help` inspects key settings)
- **Conda** (Installs VirDetector and all required programs/databases; the VirDetector environment must be activated before running the configured pipeline)
- **Docker** (Alternative installation route (bash install/docker_install.sh) with its own config-free test invocation `bash VirDetector_docker.sh`)
- **Dorado** (GPU basecaller selectable via params.basecaller = "dorado"; installation can be omitted by removing the parameter from the install command) — https://github.com/nanoporetech/dorado
- **Guppy** (CPU basecaller selectable via params.basecaller = "guppy", with flowcell type and kit specified)

## Examples

```
conda activate VirDetector; nextflow run main.nf --help; nextflow run main.nf
```

## Evaluation signals

- `nextflow run main.nf --help` reflects the intended parameter settings (step flags, basecaller, ref, genome size).
- The path given in params.ref resolves to an existing FASTA file in the resources folder for reference-based assembly.
- params.canu_genome_size matches the expected genome size (e.g. "11k" for the ~11000-base West Nile virus genome) when hybrid assembly is enabled.
- After execution, a top-level results.txt exists and summarises visual outputs and key results.
- For the shipped West Nile virus test input, consensus genome FASTAs contain a single record of length close to ~11000 bases.

## Limitations

- Specifying a metadata file is a hard filter: only samples defined in the metadata file are passed on to subsequent processing steps.
- Installing VirDetector with Conda (including Guppy and Dorado) can take several hours depending on available computing resources; basecalling itself is excluded from the default pipeline test run.
- Alignment tool choice is a speed/precision trade-off: MAFFT is generally faster while MUSCLE is more precise but requires more computing resources.
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems must use the Docker installation.

## Evidence

- [readme] VirDetector can be customised via the config file *nextflow.config*. This is the only file you have to edit to change the behaviour of the pipeline!: "VirDetector can be customised via the config file *nextflow.config*. This is the only file you have to edit to change the behaviour of the pipeline!"
- [readme] Example: If you want to disable the hybrid assembly, set **params.assembly_hybrid = "false"**: "Example: If you want to disable the hybrid assembly, set **params.assembly_hybrid = "false"**"
- [readme] Place your reference sequence file in fasta format in the folder *resources*, then change the file name here: **params.ref = "${PWD}/resources/{your_file_name}.fasta"**: "Place your reference sequence file in fasta format in the folder *resources*, then change the file name here: **params.ref = "${PWD}/resources/{your_file_name}.fasta"**"
- [readme] Example: You can specify a genome size of 11.000 bases like this: **params.canu_genome_size = "11k"**: "Example: You can specify a genome size of 11.000 bases like this: **params.canu_genome_size = "11k"**"
- [methods] Primers can be trimmed by specifying forward and reverse primer sequences in separate files in FASTA format.: "Primers can be trimmed by specifying forward and reverse primer sequences in separate files in FASTA format."
- [readme] If you specify a metadata file, only those samples are passed on to subsequent processing steps that are defined in this file!: "If you specify a metadata file, only those samples are passed on to subsequent processing steps that are defined in this file!"
- [readme] While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources.: "While MAFFT is generally faster, MUSCLE is more precise but requires more computing resources."

---
name: flowcell-kit-basecalling-configuration
description: Use when input samples are raw nanopore signal files (FAST5 or POD5)
  that require basecalling before preprocessing, or when barcoded runs need flowcell/kit
  specification and demultiplexing decisions.
when_to_use_negative:
- Input is already-basecalled FASTQ — basecalling is unnecessary and the pipeline
  can be run with basecalling disabled
- Input is FASTA assemblies (.fasta/.fas/.fsa/.fna/.fa), which VirDetector consumes
  only from the alignment step onwards, bypassing basecalling and preprocessing
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: VirDetector
  role: Pipeline orchestrating basecalling as the first optional preprocessing stage
    before NanoFilt/NanoComp filtering and quality plotting
  repo: https://github.com/NLKaiser/VirDetector
- name: Dorado
  role: GPU-based high-performance basecaller selected via params.basecaller = 'dorado',
    with automatic hardware detection
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: CPU-based basecaller alternative selected via params.basecaller = 'guppy'
- name: Nextflow
  role: Workflow engine that executes main.nf, orchestrating basecalling followed
    by per-sample read filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:6ade11eed3921fb80fb197662ee6aa35aec38958ab3151de692bd64c262c3247
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Dorado
  - Guppy
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# flowcell-kit-basecalling-configuration

## Summary

Configure nanopore signal-to-sequence basecalling in VirDetector by selecting Dorado (GPU) or Guppy (CPU) via params.basecaller in nextflow.config, specifying the flowcell type and sequencing kit for raw FAST5/POD5 input, with optional demultiplexing of barcoded samples.

## When to use

Use when input samples are raw nanopore signal files (FAST5 or POD5) that require basecalling before preprocessing, or when barcoded runs need flowcell/kit specification and demultiplexing decisions.

## When NOT to use

- Input is already-basecalled FASTQ — basecalling is unnecessary and the pipeline can be run with basecalling disabled
- Input is FASTA assemblies (.fasta/.fas/.fsa/.fna/.fa), which VirDetector consumes only from the alignment step onwards, bypassing basecalling and preprocessing

## Inputs

- FAST5 signal files (.fast5, .fast5.gz)
- POD5 signal files (.pod5, .pod5.gz)
- already-basecalled FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz) — basecalling disabled
- nextflow.config parameters: params.basecaller, flowcell type, kit specification, demultiplexing flag

## Outputs

- basecalled FASTQ files in the results directory
- demultiplexed per-barcode FASTQ files (if demultiplexing enabled)
- downstream per-sample trimmed/filtered FASTQ via NanoFilt and NanoComp quality plots

## How to apply

Place raw input files (FASTQ .fastq/.fq/.fastq.gz/.fq.gz, FAST5 .fast5/.fast5.gz, or POD5 .pod5/.pod5.gz) in the VirDetector input directory, where top-level files are treated as separate samples and files in subdirectories are combined into one sample. In nextflow.config set params.basecaller to either 'dorado' or 'guppy'; Dorado runs on GPU with sensible defaults and automatic hardware detection, while Guppy is the CPU-based option. When basecalling, also specify the flowcell type and the kits used, and decide whether barcoded samples should be demultiplexed. Skip basecalling entirely for already-basecalled FASTQ input — the pipeline test runs all steps except basecalling for this reason. Rationale: basecalling is the first optional stage before per-sample NanoFilt filtering and NanoComp quality plotting in the preprocessing stage (section 2.1).

## Related tools

- **VirDetector** (Pipeline orchestrating basecalling as the first optional preprocessing stage before NanoFilt/NanoComp filtering and quality plotting) — https://github.com/NLKaiser/VirDetector
- **Dorado** (GPU-based high-performance basecaller selected via params.basecaller = 'dorado', with automatic hardware detection) — https://github.com/nanoporetech/dorado
- **Guppy** (CPU-based basecaller alternative selected via params.basecaller = 'guppy')
- **Nextflow** (Workflow engine that executes main.nf, orchestrating basecalling followed by per-sample read filtering)

## Examples

```
conda activate VirDetector; nextflow run main.nf
```

## Evaluation signals

- Results directory contains basecalled FASTQ files as listed in the pipeline's Outputs section
- params.basecaller is set to exactly 'dorado' or 'guppy' in nextflow.config before running nextflow run main.nf
- Flowcell type and kit strings are specified when basecalling raw signal input, matching the sequencing run
- For FASTQ input, basecalling is disabled and the pipeline still completes preprocessing (trimming) of every sample
- Downstream NanoComp plots comparing unfiltered vs. filtered reads are generated for each sample

## Limitations

- The installation includes Guppy and Dorado; if you do not wish to install these, you must remove the respective parameter from install/install.sh
- Installation can take several hours depending on available computing resources
- VirDetector was developed and tested under Ubuntu 22.04 LTS; non-Linux systems must use the Docker installation
- The pipeline test run executes all steps except basecalling, so basecalling behaviour is not exercised by the default test

## Evidence

- [readme] Config: accepts FASTQ, FAST5, POD5: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] Config: basecaller selection: "To select the basecaller to be used, set **params.basecaller** to either "dorado" or "guppy""
- [readme] Config: flowcell/kit requirement: "If you want to basecall files, you must select the basecaller to be used for this. You must also select whether the barcoded samples should be demultiplexed. For this, you must specify the flowcell"
- [methods] Methods: installation includes basecallers: "The installation includes the basecallers Guppy and Dorado. If you do not wish to install these, please remove the respective parameter"
- [methods] Methods: test run excludes basecalling: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:"
- [readme] README: Dorado hardware detection: [evidence span withheld — anchor status: external_doc]

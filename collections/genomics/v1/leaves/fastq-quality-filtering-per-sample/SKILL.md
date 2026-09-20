---
name: fastq-quality-filtering-per-sample
description: Use when raw Oxford Nanopore reads (FASTQ, or FAST5/POD5 requiring basecalling)
  must be quality-filtered per sample before taxonomic classification or genome reconstruction,
  and per-sample read-quality statistics are needed.
when_to_use_negative:
- Input is already quality-filtered FASTQ and only downstream classification/assembly
  steps are needed — disable upstream preprocessing or use the filtered reads directly.
- Input consists solely of FASTA reference/assembled sequences — FASTA inputs enter
  the pipeline from the alignment step onwards, so FASTQ quality filtering does not
  apply.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: VirDetector
  role: Pipeline that orchestrates per-sample preprocessing and quality filtering
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine executing basecalling and per-sample read filtering
- name: Dorado
  role: GPU basecaller producing FASTQ from FAST5/POD5 before filtering
  repo: https://github.com/nanoporetech/dorado
- name: Guppy
  role: CPU basecaller alternative producing FASTQ before filtering
- name: NanoFilt
  role: Filters each sample's reads on quality
- name: NanoComp
  role: Generates quality plots comparing unfiltered and filtered reads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:f8f655d6d362730e3b2ea9f4219deaf08fb9beb38e5706869e754cc6cdddc41e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Dorado
  - Guppy
  - NanoComp
  - NanoFilt
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# fastq-quality-filtering-per-sample

## Summary

Per-sample quality filtering of nanopore FASTQ reads within VirDetector's preprocessing stage: after optional basecalling (Dorado or Guppy), each sample is filtered with NanoFilt and quality plots comparing unfiltered vs. filtered reads are generated with NanoComp.

## When to use

Use when raw Oxford Nanopore reads (FASTQ, or FAST5/POD5 requiring basecalling) must be quality-filtered per sample before taxonomic classification or genome reconstruction, and per-sample read-quality statistics are needed.

## When NOT to use

- Input is already quality-filtered FASTQ and only downstream classification/assembly steps are needed — disable upstream preprocessing or use the filtered reads directly.
- Input consists solely of FASTA reference/assembled sequences — FASTA inputs enter the pipeline from the alignment step onwards, so FASTQ quality filtering does not apply.

## Inputs

- FASTQ files (.fastq, .fq, .fastq.gz, .fq.gz), zipped or unzipped
- FAST5 files (.fast5, .fast5.gz) requiring basecalling
- POD5 files (.pod5, .pod5.gz) requiring basecalling

## Outputs

- Basecalled FASTQ files (when basecalling is enabled)
- Trimmed/filtered FASTQ file per sample
- NanoComp read-statistics plots comparing unfiltered vs. filtered reads

## How to apply

Place each input file (FASTQ .fastq/.fq/.fastq.gz/.fq.gz, FAST5, or POD5) as a separate sample in the VirDetector *input* directory; files inside a subdirectory are combined into one sample. Activate the VirDetector Conda environment, and in *nextflow.config* set params.basecaller to 'dorado' or 'guppy' (with flowcell type and kit) only if the input needs basecalling — skip basecalling for FASTQ input. Run `nextflow run main.nf` so Nextflow orchestrates basecalling (Dorado on GPU, Guppy on CPU) followed by per-sample read filtering with NanoFilt. NanoComp then generates quality plots comparing the unfiltered and filtered reads. Validate that the results directory contains the basecalled FASTQ files, a trimmed/filtered FASTQ for every sample, and the read-statistics plot images listed in the pipeline's Outputs section.

## Related tools

- **VirDetector** (Pipeline that orchestrates per-sample preprocessing and quality filtering) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine executing basecalling and per-sample read filtering)
- **Dorado** (GPU basecaller producing FASTQ from FAST5/POD5 before filtering) — https://github.com/nanoporetech/dorado
- **Guppy** (CPU basecaller alternative producing FASTQ before filtering)
- **NanoFilt** (Filters each sample's reads on quality)
- **NanoComp** (Generates quality plots comparing unfiltered and filtered reads)

## Examples

```
conda activate VirDetector && nextflow run main.nf
```

## Evaluation signals

- Results directory contains basecalled FASTQ files when basecalling was enabled
- A trimmed/filtered FASTQ file exists for every sample placed in the input directory
- Read-statistics plot images comparing unfiltered vs. filtered reads are present, matching the pipeline's Outputs section
- Subdirectory inputs are combined into a single sample while top-level files remain separate samples
- Filtered read counts/quality metrics in NanoComp plots show reads were removed or improved relative to unfiltered reads

## Limitations

- Basecalling (Dorado/Guppy) requires specifying flowcell type and kit in nextflow.config; misconfiguration affects the FASTQ fed to filtering.
- Enabling a metadata file causes only samples defined in that file to pass to subsequent processing steps.
- Conda installation including Guppy and Dorado can take several hours depending on computing resources.

## Evidence

- [methods] Preprocessing filtering with NanoFilt and NanoComp plots: [evidence span withheld — anchor status: not_found]
- [readme] Accepted input formats: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] Sample grouping rule: "Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample."
- [readme] Basecaller selection in config: "To select the basecaller to be used, set **params.basecaller** to either "dorado" or "guppy""
- [abstract] Single-command execution: "The pipeline automatically installs all required programs and databases and allows all its steps to be executed with a single console command."

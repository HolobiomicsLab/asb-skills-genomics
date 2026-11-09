---
name: nextflow-pipeline-execution
description: Use when ONT long-read fastq.gz samples must be run through the full
  ConsensuSV-ONT alignment-to-consensus workflow (minimap2/pbmm2 alignment, six SV
  callers, Truvari merge/collapse) inside the provided Docker/Nextflow runtime.
when_to_use_negative:
- Input reads are not ONT long-read fastq.gz files (e.g. Illumina or PacBio data)
  — the pipeline is built for Oxford Nanopore sequencing technology
- The sample list CSV contains headers or multiple columns — the README states only
  one column with fastq.gz paths is accepted, without a header row
- You already have per-caller VCFs or BAMs and only need consensus merging (Truvari
  merge/collapse) or CNN filtering rather than a full pipeline run
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Nextflow
  role: Workflow engine that parallelizes the alignment and six-caller SV calling
    steps across samples
- name: Docker
  role: Runtime environment image (antonipietryga/consensussv-ont-pipeline) that wraps
    the Nextflow pipeline so it runs reproducibly
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: ConsensuSV-ONT
  role: The consensus SV-calling pipeline being executed, from alignment file generation
    through variant calling to final filtering and scoring
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:7ce00e6c788f51f783dac8e478873798542c73182bab365540d0075f29112160
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  tools:
  - ConsensuSV-ONT
  - CuteSV
  - Docker
  - Dysgu
  - Nanovar
  - Nextflow
  - PBSV
  - SVIM
  - Sniffles
  - minimap2
  - pbmm2
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9523809523809523
n_claims: 21
---

# nextflow-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Executes the ConsensuSV-ONT meta-caller as a Nextflow pipeline inside the antonipietryga/consensussv-ont-pipeline Docker image, running six SV callers (CuteSV, Sniffles, Dysgu, SVIM, PBSV, Nanovar) in parallel on ONT fastq.gz samples and producing per-sample alignment BAMs plus per-caller and consensus VCFs.

## When to use

Use when ONT long-read fastq.gz samples must be run through the full ConsensuSV-ONT alignment-to-consensus workflow (minimap2/pbmm2 alignment, six SV callers, Truvari merge/collapse) inside the provided Docker/Nextflow runtime.

## When NOT to use

- Input reads are not ONT long-read fastq.gz files (e.g. Illumina or PacBio data) — the pipeline is built for Oxford Nanopore sequencing technology
- The sample list CSV contains headers or multiple columns — the README states only one column with fastq.gz paths is accepted, without a header row
- You already have per-caller VCFs or BAMs and only need consensus merging (Truvari merge/collapse) or CNN filtering rather than a full pipeline run

## Inputs

- fastq.gz ONT long-read read files (e.g. HG00733, HG00514, NA19240 from HGSVC)
- headerless one-column CSV listing paths to the fastq.gz sample files
- reference genome FASTA (hg38) passed via --ref
- per-task thread (--threads) and memory (--mem) limits

## Outputs

- alignments/ folder with per-sample minimap2 and pbmm2 BAM files
- vcfs/<sample>/ folders containing one VCF per caller per sample (18 VCFs for 3 samples x 6 callers) plus the ConsensuSV-ONT consensus VCF

## How to apply

Prepare a headerless CSV (one column, one path to a fastq.gz file per row) listing the ONT samples, then enter the container with 'docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline'. Launch 'nextflow pipeline.nf --input files.csv --ref <hg38 fasta> --outdir <dir>', setting --threads and --mem to the available per-task thread and memory limits. The pipeline aligns each sample's reads with minimap2 for the CuteSV, Sniffles, Dysgu, SVIM and Nanovar runs and with pbmm2 for the PBSV run, then executes the six callers in parallel and applies the paper's per-caller pre-filtering before consensus. Rationale for the runtime design is reproducibility and parallel processing: the docker image wraps a nextflow pipeline so non-computer-scientists can run the full calling stack uniformly. Outputs are written under the --outdir working directory: an 'alignments' folder of per-sample BAMs and a 'vcfs' folder with per-sample subfolders holding each caller's VCF plus the ConsensuSV-ONT result.

## Related tools

- **Nextflow** (Workflow engine that parallelizes the alignment and six-caller SV calling steps across samples)
- **Docker** (Runtime environment image (antonipietryga/consensussv-ont-pipeline) that wraps the Nextflow pipeline so it runs reproducibly) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **ConsensuSV-ONT** (The consensus SV-calling pipeline being executed, from alignment file generation through variant calling to final filtering and scoring) — github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv --ref hg38.fa --outdir out --threads 8 --mem 16
```

## Evaluation signals

- Both 'alignments' and 'vcfs' folders exist under the --outdir working directory after the run
- Every caller/sample combination produced a VCF: 6 callers x 3 samples = 18 per-caller VCFs in the per-sample vcfs subfolders, plus the consensus output
- Downstream Truvari collapse output is a valid VCF with fewer records than the union of the six input per-caller VCFs
- Pipeline completed within the declared --threads and --mem per-task limits inside the docker container
- Each per-caller VCF corresponds to the correct aligner provenance (minimap2 BAMs for CuteSV/Sniffles/Dysgu/SVIM/Nanovar, pbmm2 BAM for PBSV)

## Limitations

- Output location depends on the working directory passed to the pipeline, so results are only where --outdir points
- The article gives no version numbers or implementation parameters for the pipeline in its discussion section
- No changelog is available in the repository artifact
- The CSV sample manifest must be headerless — headers are shown in the README example only for illustration

## Evidence

- [intro] Scope of the pipeline as alignment-to-consensus platform: [evidence span withheld — anchor status: not_found]

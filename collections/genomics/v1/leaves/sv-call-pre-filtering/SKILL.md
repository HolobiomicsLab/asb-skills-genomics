---
name: sv-call-pre-filtering
description: Use when building per-caller SV candidate VCFs from ONT fastq.gz reads
  (e.g. HG00733, HG00514, NA19240) and preparing them for downstream consensus in
  ConsensuSV-ONT.
when_to_use_negative:
- Input is already a set of filtered or consensus SV VCFs rather than raw per-caller
  candidate calls from the six supported callers
- Reads are short-read (Illumina) or PacBio HiFi rather than ONT fastq.gz long reads,
  since the pipeline and its caller set are tuned for ONT data
edam_operation: http://edamontology.org/operation_3227
tools:
- name: ConsensuSV-ONT
  role: Wrapping pipeline that orchestrates alignment, per-caller SV calling, pre-filtering
    and consensus
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Parallelizes the alignment and per-caller SV calling tasks via pipeline.nf
- name: Docker
  role: Runtime environment image antonipietryga/consensussv-ont-pipeline that packages
    the pipeline
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: CuteSV
  role: One of six SV callers run on minimap2 BAMs to produce candidate VCFs
- name: Sniffles
  role: One of six SV callers run on minimap2 BAMs to produce candidate VCFs
- name: Dysgu
  role: One of six SV callers run on minimap2 BAMs to produce candidate VCFs
- name: SVIM
  role: One of six SV callers run on minimap2 BAMs to produce candidate VCFs
- name: PBSV
  role: SV caller run on pbmm2 BAMs to produce candidate VCFs
- name: Nanovar
  role: One of six SV callers run on minimap2 BAMs to produce candidate VCFs
- name: minimap2
  role: Aligns ONT reads to hg38 for the CuteSV, Sniffles, Dysgu, SVIM and Nanovar
    runs
- name: pbmm2
  role: Aligns ONT reads to hg38 for the PBSV run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:6ceea8f872d0aaf086d2ce76d9044e35afa255a25631bb7ddf7cb09583219598
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
  repo_url: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# sv-call-pre-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates and pre-filters per-caller structural variant candidate VCFs from ONT long-read alignments in the ConsensuSV-ONT pipeline, so that CuteSV, Sniffles, Dysgu, SVIM, PBSV and Nanovar raw calls are cleaned per caller before CNN-based consensus filtering.

## When to use

Use when building per-caller SV candidate VCFs from ONT fastq.gz reads (e.g. HG00733, HG00514, NA19240) and preparing them for downstream consensus in ConsensuSV-ONT.

## When NOT to use

- Input is already a set of filtered or consensus SV VCFs rather than raw per-caller candidate calls from the six supported callers
- Reads are short-read (Illumina) or PacBio HiFi rather than ONT fastq.gz long reads, since the pipeline and its caller set are tuned for ONT data

## Inputs

- fastq.gz ONT long-read read files (one or more per sample)
- headerless CSV file with one column containing paths to fastq.gz sample files
- reference genome FASTA (hg38)
- minimap2-generated per-sample BAM alignments (CuteSV, Sniffles, Dysgu, SVIM, Nanovar)
- pbmm2-generated per-sample BAM alignments (PBSV)

## Outputs

- one raw candidate VCF per caller per sample (e.g. 18 VCFs for 3 samples x 6 callers)
- per-sample BAM alignment files stored in the 'alignments' folder
- per-sample folders under the 'vcfs' output directory containing all individual caller VCFs and the ConsensuSV-ONT consensus VCF

## How to apply

Prepare a headerless CSV listing each sample's fastq.gz path, enter the antonipietryga/consensussv-ont-pipeline Docker container, and launch 'nextflow pipeline.nf --input files.csv --ref <hg38 fasta> --outdir <dir>', tuning --threads and --mem to per-task limits. Align each sample with minimap2 (for CuteSV, Sniffles, Dysgu, SVIM, Nanovar) and with pbmm2 (for PBSV), storing per-sample BAMs in the alignments folder. Run the six callers on their designated alignments in parallel via Nextflow, then apply the paper's per-caller pre-filtering to each caller's raw calls before the CNN-based consensus stage. Output one VCF per caller per sample (18 VCFs for 3 samples) into per-sample folders under the vcfs directory.

## Related tools

- **ConsensuSV-ONT** (Wrapping pipeline that orchestrates alignment, per-caller SV calling, pre-filtering and consensus) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Parallelizes the alignment and per-caller SV calling tasks via pipeline.nf)
- **Docker** (Runtime environment image antonipietryga/consensussv-ont-pipeline that packages the pipeline) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **CuteSV** (One of six SV callers run on minimap2 BAMs to produce candidate VCFs)
- **Sniffles** (One of six SV callers run on minimap2 BAMs to produce candidate VCFs)
- **Dysgu** (One of six SV callers run on minimap2 BAMs to produce candidate VCFs)
- **SVIM** (One of six SV callers run on minimap2 BAMs to produce candidate VCFs)
- **PBSV** (SV caller run on pbmm2 BAMs to produce candidate VCFs)
- **Nanovar** (One of six SV callers run on minimap2 BAMs to produce candidate VCFs)
- **minimap2** (Aligns ONT reads to hg38 for the CuteSV, Sniffles, Dysgu, SVIM and Nanovar runs)
- **pbmm2** (Aligns ONT reads to hg38 for the PBSV run)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv --ref hg38.fa --outdir results
```

## Evaluation signals

- Every caller/sample combination produced a VCF (3 samples x 6 callers = 18 VCFs) under the vcfs output directory
- Per-sample BAM files exist in the alignments folder, generated by minimap2 for five callers and pbmm2 for PBSV
- Each output VCF contains only variants that passed that caller's pre-filtering, applied before consensus generation
- The vcfs directory contains one folder per sample holding all individual SV caller VCFs plus the ConsensuSV-ONT consensus VCF

## Limitations

- The README states the csv must be headerless: 'Bear in mind that the column headers are provided only for the ease of the example, and should not be present in the csv file.'
- Resource-limited runs require explicit --threads and --mem settings; per-caller pre-filter thresholds are applied as described in the paper and are not exposed as standalone CLI parameters in the README
- Runtime environment assumes local directories are mounted into the Docker container (e.g. -v /mnt:/mnt -p 8082:8082)

## Evidence

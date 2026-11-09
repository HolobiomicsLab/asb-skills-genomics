---
name: multi-caller-vcf-collection
description: Use when starting a consensus SV-calling workflow from ONT fastq.gz samples
  and you need per-caller candidate VCFs before pre-filtering and CNN-based consensus;
  triggered by headerless sample CSVs of fastq.gz paths for HGSVC-style samples.
when_to_use_negative:
- Your input is already a set of caller VCFs — this skill only covers alignment and
  raw candidate generation, not consensus or CNN filtering
- Your reads are Illumina or PacBio short-read data; the pipeline targets fastq.gz
  Oxford Nanopore long-read samples
- You need per-sample variants for a single caller only — the pipeline runs all six
  callers as a set
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: ConsensuSV-ONT
  role: consensus pipeline that orchestrates the six callers and per-sample VCF collection
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: parallel execution engine running alignment and SV calling tasks
- name: Docker
  role: runtime environment image antonipietryga/consensussv-ont-pipeline
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: CuteSV
  role: SV caller run on minimap2 BAMs
- name: Sniffles
  role: SV caller run on minimap2 BAMs
- name: Dysgu
  role: SV caller run on minimap2 BAMs
- name: SVIM
  role: SV caller run on minimap2 BAMs
- name: Nanovar
  role: SV caller run on minimap2 BAMs
- name: PBSV
  role: SV caller run on pbmm2 BAMs
- name: minimap2
  role: aligner producing BAMs for CuteSV, Sniffles, Dysgu, SVIM and Nanovar
- name: pbmm2
  role: aligner producing BAMs for PBSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:4a27a3f704bceb69f0d061c1db4cee98c3d847f50f24f524098b6be746c4ff4e
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# multi-caller-vcf-collection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates one structural-variant candidate VCF per SV caller per ONT sample by running CuteSV, Sniffles, Dysgu, SVIM, PBSV and Nanovar on per-sample long-read alignments (minimap2 for the first five, pbmm2 for PBSV) inside the ConsensuSV-ONT Nextflow pipeline, producing 18 VCFs for three HGSVC samples (HG00733, HG00514, NA19240) that feed downstream consensus.

## When to use

Use when starting a consensus SV-calling workflow from ONT fastq.gz samples and you need per-caller candidate VCFs before pre-filtering and CNN-based consensus; triggered by headerless sample CSVs of fastq.gz paths for HGSVC-style samples.

## When NOT to use

- Your input is already a set of caller VCFs — this skill only covers alignment and raw candidate generation, not consensus or CNN filtering
- Your reads are Illumina or PacBio short-read data; the pipeline targets fastq.gz Oxford Nanopore long-read samples
- You need per-sample variants for a single caller only — the pipeline runs all six callers as a set

## Inputs

- fastq.gz ONT read files for each sample (e.g. HG00733, HG00514, NA19240)
- headerless one-column CSV listing sample fastq.gz paths
- reference genome FASTA (hg38)

## Outputs

- per-sample BAM alignment files in the 'alignments' output folder (minimap2 and pbmm2 generated)
- one VCF per caller per sample (18 VCFs total for three samples), organized in per-sample folders under the 'vcfs' output directory

## How to apply

Prepare a headerless CSV listing one fastq.gz path per sample row, enter the consensussv-ont-pipeline Docker container, and launch `nextflow pipeline.nf` with `--input`, `--ref` (hg38 fasta), `--outdir`, `--threads` and `--mem`. Align each sample's reads with minimap2 for the CuteSV, Sniffles, Dysgu, SVIM and Nanovar runs and with pbmm2 for the PBSV run, storing per-sample BAMs in the 'alignments' output folder. Run the six callers on their designated BAMs via the parallelized Nextflow pipeline, then apply per-caller pre-filtering to the raw calls. The rationale is that caller-specific aligners and per-caller outputs preserve each tool's raw candidate set for later image encoding and CNN filtering.

## Related tools

- **ConsensuSV-ONT** (consensus pipeline that orchestrates the six callers and per-sample VCF collection) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (parallel execution engine running alignment and SV calling tasks)
- **Docker** (runtime environment image antonipietryga/consensussv-ont-pipeline) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **CuteSV** (SV caller run on minimap2 BAMs)
- **Sniffles** (SV caller run on minimap2 BAMs)
- **Dysgu** (SV caller run on minimap2 BAMs)
- **SVIM** (SV caller run on minimap2 BAMs)
- **Nanovar** (SV caller run on minimap2 BAMs)
- **PBSV** (SV caller run on pbmm2 BAMs)
- **minimap2** (aligner producing BAMs for CuteSV, Sniffles, Dysgu, SVIM and Nanovar)
- **pbmm2** (aligner producing BAMs for PBSV)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline && nextflow pipeline.nf --input files.csv --ref hg38.fa --outdir out --threads 8 --mem 16G
```

## Evaluation signals

- For N samples in the CSV, exactly 6 caller/sample VCF combinations exist per sample (18 VCFs total for HG00733, HG00514 and NA19240)
- Each sample has both a minimap2-generated BAM (for CuteSV/Sniffles/Dysgu/SVIM/Nanovar) and a pbmm2-generated BAM (for PBSV) in the alignments folder
- VCFs are written to per-sample folders under the vcfs output directory as the pipeline specifies
- The input CSV is headerless with exactly one path per row, matching the README sample-preparation format
- Raw caller outputs have undergone per-caller pre-filtering before any downstream consensus step

## Limitations

- The public article's methods section does not name the six individual callers — caller identities (CuteSV, Sniffles, Dysgu, SVIM, PBSV, Nanovar) come from the task's source card, and no version numbers or per-caller parameters are given
- No changelog is available in the repository, complicating reproducibility checks
- The pipeline requires a Docker environment and per-task --threads/--mem limits, so it cannot be run tool-by-tool without the containerized Nextflow setup

## Evidence

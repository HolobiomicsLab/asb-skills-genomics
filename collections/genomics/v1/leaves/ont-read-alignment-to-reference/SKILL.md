---
name: ont-read-alignment-to-reference
description: Use when raw ONT fastq.gz reads (e.g. HGSVC samples HG00733, HG00514,
  NA19240) must be aligned to a reference genome so that downstream long-read structural
  variant callers can be run.
when_to_use_negative:
- Input data is short-read Illumina or PacBio rather than Oxford Nanopore long-read
  fastq.gz — the pipeline is built for ONT reads
- You need only final consensus SV calls and already possess per-caller BAMs or VCFs
  — alignment would be redundant
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns ONT reads to hg38 producing BAMs used as input by CuteSV, Sniffles,
    Dysgu, SVIM and Nanovar
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: pbmm2
  role: Aligns ONT reads to hg38 producing the BAM input required by the PBSV caller
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: ConsensuSV-ONT
  role: Docker-wrapped Nextflow pipeline that orchestrates the alignment and SV calling
    steps
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Executes the pipeline with parallel computing across samples and callers
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment image antonipietryga/consensussv-ont-pipeline wrapping
    the Nextflow pipeline
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:82034f8df234580fcd1c25ef2aa5946027c841745da4e9b0465406fc1c9532af
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
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 20
---

# ONT read alignment to reference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns Oxford Nanopore long-read FASTQ files to the hg38 reference genome with minimap2 (for CuteSV, Sniffles, Dysgu, SVIM, Nanovar) and pbmm2 (for PBSV) as the alignment stage of the ConsensuSV-ONT Nextflow pipeline, producing per-sample BAMs that feed six SV callers.

## When to use

Use when raw ONT fastq.gz reads (e.g. HGSVC samples HG00733, HG00514, NA19240) must be aligned to a reference genome so that downstream long-read structural variant callers can be run.

## When NOT to use

- Input data is short-read Illumina or PacBio rather than Oxford Nanopore long-read fastq.gz — the pipeline is built for ONT reads
- You need only final consensus SV calls and already possess per-caller BAMs or VCFs — alignment would be redundant

## Inputs

- Headerless CSV file with one column: path to each sample's fastq.gz ONT read file
- Reference genome FASTA (hg38)
- Parameters: --threads (max threads per task), --mem (max memory per thread), --outdir, --ref

## Outputs

- Per-sample minimap2 BAM alignments (for CuteSV, Sniffles, Dysgu, SVIM, Nanovar)
- Per-sample pbmm2 BAM alignments (for PBSV)
- alignments/ folder containing alignment files for each sample

## How to apply

Prepare a headerless single-column CSV listing the fastq.gz read paths per sample and enter the antonipietryga/consensussv-ont-pipeline Docker container. Launch the Nextflow pipeline with nextflow pipeline.nf --input files.csv --ref <hg38 fasta> --outdir <dir>, setting --threads and --mem to available per-task limits. The pipeline aligns each sample's reads to hg38 with minimap2 for the CuteSV, Sniffles, Dysgu, SVIM and Nanovar runs, and with pbmm2 for the PBSV run, storing the per-sample BAMs in the 'alignments' output folder. The rationale for dual aligners is caller-specific compatibility: PBSV expects pbmm2 alignments while the other five callers consume minimap2 BAMs. Correctness is judged by verifying that every caller/sample alignment was produced and that each of the six callers (18 VCFs total across three samples) yielded a VCF per sample in the vcfs folder.

## Related tools

- **minimap2** (Aligns ONT reads to hg38 producing BAMs used as input by CuteSV, Sniffles, Dysgu, SVIM and Nanovar) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **pbmm2** (Aligns ONT reads to hg38 producing the BAM input required by the PBSV caller) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **ConsensuSV-ONT** (Docker-wrapped Nextflow pipeline that orchestrates the alignment and SV calling steps) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Executes the pipeline with parallel computing across samples and callers) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment image antonipietryga/consensussv-ont-pipeline wrapping the Nextflow pipeline) — github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv --ref hg38.fa --outdir results --threads 8 --mem 8
```

## Evaluation signals

- Every sample listed in the headerless CSV has a BAM in the alignments/ output folder
- Both minimap2-derived BAMs (five callers) and the pbmm2-derived BAM (PBSV) exist per sample
- Downstream, all six callers produced a VCF for each sample — 18 VCFs in total for three samples — confirming alignments were valid
- One folder per sample appears under the vcfs output directory containing VCFs from each individual calling tool and ConsensuSV-ONT

## Limitations

- Memory and thread limits per task must be set explicitly via --threads and --mem; defaults may not match available resources
- The input CSV must be headerless — column headers are shown in the README example only for ease of reading and must not be present
- PBSV will fail or produce no calls if fed minimap2 BAMs, since it is run on the pbmm2-generated BAM file instead

## Evidence

- [methods] dual aligner workflow: [evidence span withheld — anchor status: not_found]

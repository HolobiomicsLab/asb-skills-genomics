---
name: long-read-structural-variant-calling
description: Use when calling deletions and insertions from ONT long-read FASTQ samples
  against a reference genome (e.g. hg38), when a consensus meta-caller should outperform
  single SV callers, or when benchmarking SV callsets with Truvari against ground
  truth.
when_to_use_negative:
- Input reads are Illumina or PacBio short/long reads rather than ONT data — the pipeline
  and its six callers are tuned for Oxford Nanopore long-read sequencing
- Input is already a collapsed or consensus VCF; this skill runs the full path from
  raw fastq.gz to consensus calls
- Only insertions or only callers outside the six supported ones (CuteSV, Sniffles,
  Dysgu, SVIM, PBSV, Nanovar) are required and cannot be substituted
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: ConsensuSV-ONT
  role: Consensus meta-caller orchestrating six SV callers and CNN-based variant filtering
    via a Nextflow pipeline
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Workflow engine providing parallelized execution of alignment, calling, merging,
    and filtering tasks
- name: Docker
  role: Runtime environment image wrapping the Nextflow pipeline for reproducible
    execution
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: minimap2
  role: Aligns ONT reads to hg38 to produce BAM inputs for CuteSV, Sniffles, Dysgu,
    SVIM and Nanovar
- name: pbmm2
  role: Aligns reads to produce the BAM input required by PBSV
- name: CuteSV
  role: One of six SV callers run on minimap2 BAMs to produce candidate deletion/insertion
    calls
- name: Sniffles
  role: One of six SV callers run on minimap2 BAMs to produce candidate calls
- name: Dysgu
  role: One of six SV callers run on minimap2 BAMs to produce candidate calls
- name: SVIM
  role: One of six SV callers run on minimap2 BAMs to produce candidate calls
- name: PBSV
  role: One of six SV callers run on the pbmm2-generated BAM to produce candidate
    calls
- name: Nanovar
  role: One of six SV callers run on minimap2 BAMs to produce candidate calls
- name: Truvari
  role: Merge module combines per-caller VCFs into one candidate VCF; collapse module
    removes overlapping redundant records; bench module computes precision/recall/F1
    against truth sets
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:0cadb014c9af6853cf20d37e9da15ac80b276bdd0834a1a11bae9a39cc68eae1
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
  - Truvari
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
yes_rate: 0.9565217391304348
n_claims: 23
---

# long-read-structural-variant-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consensus-based structural variant (SV) calling on Oxford Nanopore long-read data via ConsensuSV-ONT, which runs six independent SV callers on per-sample alignments, merges and collapses their calls with Truvari, and filters high-quality variants with a 3D convolutional neural network.

## When to use

Use when calling deletions and insertions from ONT long-read FASTQ samples against a reference genome (e.g. hg38), when a consensus meta-caller should outperform single SV callers, or when benchmarking SV callsets with Truvari against ground truth.

## When NOT to use

- Input reads are Illumina or PacBio short/long reads rather than ONT data — the pipeline and its six callers are tuned for Oxford Nanopore long-read sequencing
- Input is already a collapsed or consensus VCF; this skill runs the full path from raw fastq.gz to consensus calls
- Only insertions or only callers outside the six supported ones (CuteSV, Sniffles, Dysgu, SVIM, PBSV, Nanovar) are required and cannot be substituted

## Inputs

- Headerless CSV file listing one public ONT fastq.gz read file path per row (e.g. HG00733, HG00514, NA19240)
- Reference genome FASTA (hg38)
- Docker image antonipietryga/consensussv-ont-pipeline

## Outputs

- Per-sample BAM alignment files in the 'alignments' output folder
- One VCF per caller per sample (18 VCFs for 3 samples) in per-sample folders under 'vcfs'
- Per-sample collapsed candidate VCF from Truvari merge + collapse
- Per-sample ConsensuSV-ONT consensus VCF of high-quality (CNN-filtered) SV calls
- Truvari bench precision/recall/F1 metrics per method, sample, and SV type (DEL/INS)

## How to apply

Prepare a headerless CSV listing each sample's fastq.gz path, enter the antonipietryga/consensussv-ont-pipeline Docker container, and launch 'nextflow pipeline.nf --input files.csv --ref <hg38 fasta> --outdir <dir>', tuning --threads and --mem to available resources. The pipeline aligns each sample with minimap2 (used by CuteSV, Sniffles, Dysgu, SVIM, Nanovar) and pbmm2 (used by PBSV), stores BAMs in the 'alignments' folder, then runs the six SV callers on their designated BAMs. Per-caller raw calls are pre-filtered per the paper's criteria, combined with Truvari merge and made non-redundant with Truvari collapse, yielding a collapsed candidate VCF per sample. Candidates are encoded as 50x50x3 images, scored by a 3D CNN (Conv3D/BatchNorm/MaxPool stack, ~59,929 params), and variants with sigmoid output above 0.5 are kept as the final ConsensuSV-ONT VCF. Correct application yields 18 per-caller VCFs for three samples (3 samples x 6 callers) plus one consensus VCF per sample under the 'vcfs' output directory.

## Related tools

- **ConsensuSV-ONT** (Consensus meta-caller orchestrating six SV callers and CNN-based variant filtering via a Nextflow pipeline) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Workflow engine providing parallelized execution of alignment, calling, merging, and filtering tasks)
- **Docker** (Runtime environment image wrapping the Nextflow pipeline for reproducible execution) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **minimap2** (Aligns ONT reads to hg38 to produce BAM inputs for CuteSV, Sniffles, Dysgu, SVIM and Nanovar)
- **pbmm2** (Aligns reads to produce the BAM input required by PBSV)
- **CuteSV** (One of six SV callers run on minimap2 BAMs to produce candidate deletion/insertion calls)
- **Sniffles** (One of six SV callers run on minimap2 BAMs to produce candidate calls)
- **Dysgu** (One of six SV callers run on minimap2 BAMs to produce candidate calls)
- **SVIM** (One of six SV callers run on minimap2 BAMs to produce candidate calls)
- **PBSV** (One of six SV callers run on the pbmm2-generated BAM to produce candidate calls)
- **Nanovar** (One of six SV callers run on minimap2 BAMs to produce candidate calls)
- **Truvari** (Merge module combines per-caller VCFs into one candidate VCF; collapse module removes overlapping redundant records; bench module computes precision/recall/F1 against truth sets)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv --ref hg38.fa --outdir results
```

## Evaluation signals

- Every caller/sample combination produced a VCF: 18 per-caller VCFs for three samples plus one ConsensuSV-ONT VCF per sample under the 'vcfs' folder
- Collapsed candidate VCF contains fewer variant records than the union of the input per-caller VCFs and parses as a valid VCF
- The 'alignments' output folder contains per-sample BAMs, with pbmm2-derived BAMs only for the PBSV run
- CNN-filtered consensus VCF retains only variants whose sigmoid output exceeds 0.5
- Assembled precision/recall/F1 table reproduces reported values, e.g. F1 ConsensuSV-ONT DEL NA19240 = 0.6733 and INS NA19240 = 0.6487

## Limitations

- The collapsed candidate set gives the highest recall but low precision and therefore low F1; the CNN filtering step is needed for balanced performance
- The article leaves gaps: no version numbers or detailed parameters are given in some sections, and no changelog is present in the repository
- Benchmarking found a need for more detailed comparative analysis of mutation detection across tools and sequencing methods

## Evidence

- [results] Benchmark outcome: [evidence span withheld — anchor status: not_found]

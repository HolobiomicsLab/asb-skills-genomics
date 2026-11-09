---
name: consensus-sv-vcf-generation
description: Use when you have ONT fastq.gz long-read samples and need high-quality
  SV calls (DEL/INS) as VCFs from a consensus of six callers plus CNN filtering, rather
  than a single caller.
when_to_use_negative:
- Input data is short-read (Illumina/PacBio) rather than Oxford Nanopore long-read
  sequencing, which the six callers and pipeline are designed for
- You only need calls from a single SV caller and do not want CNN-based filtering
  overhead
- You lack the computational environment to run docker/nextflow and cannot use the
  provided runtime image
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: ConsensuSV-ONT
  role: Consensus meta-caller that collapses six independent SV callers' results and
    filters candidates with a 3D CNN to produce the final VCF
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the nextflow pipeline is executed
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline engine (pipeline.nf) orchestrating alignment, the six SV callers
    and CNN filtering with parallel computing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:8e576cbe00315755028247c5d05e6cf04efc05a462a1a1d361f477d24e702d87
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
  - Docker
  - Nextflow
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# consensus-sv-vcf-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate consensus structural variant VCFs by running six independent long-read SV callers inside the ConsensuSV-ONT docker/nextflow pipeline and filtering the collapsed candidate set with a 3D CNN (sigmoid output > 0.5), producing per-sample VCFs alongside the individual callers' VCFs.

## When to use

Use when you have ONT fastq.gz long-read samples and need high-quality SV calls (DEL/INS) as VCFs from a consensus of six callers plus CNN filtering, rather than a single caller.

## When NOT to use

- Input data is short-read (Illumina/PacBio) rather than Oxford Nanopore long-read sequencing, which the six callers and pipeline are designed for
- You only need calls from a single SV caller and do not want CNN-based filtering overhead
- You lack the computational environment to run docker/nextflow and cannot use the provided runtime image

## Inputs

- fastq.gz long-read sequencing files per sample
- CSV sample list with a single path column (no header)
- Reference genome (via --ref)
- Trained 3D convolutional neural network model for variant filtering

## Outputs

- Per-sample alignment files in the alignments output folder
- Per-sample VCF files from each of the six individual SV callers in the vcfs folder
- Filtered ConsensuSV-ONT consensus VCF per sample (high-quality SV calls, sigmoid > 0.5)

## How to apply

Prepare a CSV sample list whose single column holds paths to fastq.gz files (no header), then run the ConsensuSV-ONT nextflow pipeline inside the pulled antonipietryga/consensussv-ont-pipeline docker image with `nextflow pipeline.nf --input files.csv`, optionally tuning --threads, --mem, --outdir and --ref. The pipeline aligns each sample and runs six independent state-of-the-art SV callers, collapsing their results into per-sample candidate VCFs in the vcfs output folder. Each collapsed variant candidate is encoded for images and scored by the trained 3D CNN (conv3d stack with batch normalization, max pooling, flatten and dense layers totalling 59,929 parameters); variants whose sigmoid output exceeds 0.5 are kept as high-quality consensus calls. Write the filtered ConsensuSV-ONT VCF into the per-sample vcfs folder next to the six individual callers' VCFs, then benchmark against truth sets (e.g. Truvari bench for DEL and INS) to confirm quality.

## Related tools

- **ConsensuSV-ONT** (Consensus meta-caller that collapses six independent SV callers' results and filters candidates with a 3D CNN to produce the final VCF) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the nextflow pipeline is executed) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline engine (pipeline.nf) orchestrating alignment, the six SV callers and CNN filtering with parallel computing)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv
```

## Evaluation signals

- The vcfs output folder contains per-sample subfolders with VCFs from all six individual callers plus the ConsensuSV-ONT consensus VCF
- Kept candidates all have CNN sigmoid outputs exceeding 0.5
- Truvari bench of the consensus VCF against a benchmark truth set yields precision/recall/F1 matching reported values, e.g. F1 ConsensuSV-ONT DEL NA19240 = 0.6733 and INS NA19240 = 0.6487
- Consensus F1 outperforms each single caller and cnnLSV, while the unfiltered collapsed set shows highest recall but low precision
- Both alignments and vcfs folders are created in the working directory defined by --outdir

## Limitations

- The names of the six individual SV callers are not detailed in the summarized sections, limiting caller-level provenance
- No version numbers, parameters, or implementation details for ConsensuSV-ONT are given in the discussion section
- There is a need for a more detailed comparative analysis of mutation detection using different tools and between different sequencing methods

## Evidence

- [abstract] The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants: "The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants"
- [methods] The tool designed for getting consensus out of multiple SV callers' results.: "The tool designed for getting consensus out of multiple SV callers' results."

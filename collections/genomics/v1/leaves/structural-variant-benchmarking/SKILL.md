---
name: structural-variant-benchmarking
description: Use when a per-sample collapsed consensus VCF from the ConsensuSV-ONT
  pipeline (e.g. HG00733, HG00514) needs TP/FP labels against an HGSVC ground truth
  to train or evaluate a CNN SV quality filter.
when_to_use_negative:
- Input VCFs come from the individual SV callers rather than the collapsed consensus
  set — this skill labels only the consensus candidates
- No HGSVC benchmark VCF exists for the sample's genome, so no ground-truth counterparts
  can be defined
- Goal is calling SVs de novo from FASTQ rather than labelling existing consensus
  calls against a benchmark
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Runs bench comparison (default bp and sizefold similarity thresholds) between
    the collapsed consensus VCF and the HGSVC benchmark VCF to assign TP/FP labels
- name: ConsensuSV-ONT
  role: Produces the collapsed consensus VCF candidates in the vcfs output folder
    that are benchmarked
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Pipeline framework whose vcfs output folder supplies the per-sample consensus
    VCFs being labelled
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the pipeline and its outputs are generated
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:c53e125fc2c268da35370c8637ad8425566831bc003474c5f35be98fddbb6527
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
  - Docker
  - Nextflow
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# structural-variant-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Labels a collapsed structural variant (SV) candidate set as True Positives or False Positives by benchmarking each sample's ConsensuSV-ONT consensus VCF against the HGSVC benchmark SV VCF (Chaisson et al. 2019) using Truvari bench, producing labelled data for training the CNN classification model.

## When to use

Use when a per-sample collapsed consensus VCF from the ConsensuSV-ONT pipeline (e.g. HG00733, HG00514) needs TP/FP labels against an HGSVC ground truth to train or evaluate a CNN SV quality filter.

## When NOT to use

- Input VCFs come from the individual SV callers rather than the collapsed consensus set — this skill labels only the consensus candidates
- No HGSVC benchmark VCF exists for the sample's genome, so no ground-truth counterparts can be defined
- Goal is calling SVs de novo from FASTQ rather than labelling existing consensus calls against a benchmark

## Inputs

- Per-sample collapsed consensus VCF from the ConsensuSV-ONT pipeline vcfs output folder
- Matching HGSVC benchmark SV VCF (Chaisson et al. 2019) for the sample's genome (e.g. HG00733, HG00514)

## Outputs

- Per-sample labelled table tp_fp_labels.tsv with columns for sample ID, variant coordinates, SV type, and TP/FP label
- Truvari bench comparison output classifying each candidate as TP or FP

## How to apply

Load each sample's collapsed consensus VCF from the pipeline's vcfs output folder, which contains all VCFs from the individual SV calling tools and ConsensuSV-ONT. Download the matching HGSVC benchmark SV VCF (Chaisson et al. 2019) for that sample's genome and run Truvari bench with default matching parameters (default bp and sizefold similarity thresholds) to compare candidates against the benchmark. Parse the Truvari bench output to classify each candidate as TP (counterpart present in the ground truth) or FP (no counterpart). Write a per-sample labelled table tp_fp_labels.tsv with sample ID, variant coordinates, SV type, and the TP/FP label; the rationale is that these labels are the supervision signal for the ConsensuSV-ONT convolutional neural network that filters high-quality variants.

## Related tools

- **Truvari** (Runs bench comparison (default bp and sizefold similarity thresholds) between the collapsed consensus VCF and the HGSVC benchmark VCF to assign TP/FP labels)
- **ConsensuSV-ONT** (Produces the collapsed consensus VCF candidates in the vcfs output folder that are benchmarked) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Pipeline framework whose vcfs output folder supplies the per-sample consensus VCFs being labelled)
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the pipeline and its outputs are generated) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
nextflow pipeline.nf --input files.csv --outdir results  # then: truvari bench -b hgsvc_<sample>.vcf.gz -c results/vcfs/<sample>/consensus.vcf
```

## Evaluation signals

- tp_fp_labels.tsv exists for each sample and every row carries exactly one of the labels TP or FP
- No candidate rows are dropped: row count in tp_fp_labels.tsv matches the variant count in the input consensus VCF
- TP/FP assignments reproduce the Truvari bench classification (TP = variant with a counterpart in the HGSVC benchmark, FP = absent)
- Labelled rows include sample ID, variant coordinates, and SV type columns populated for every candidate
- Per-sample benchmark uses the HGSVC VCF matching that sample's genome (e.g. HG00733 or HG00514)

## Limitations

- Default Truvari bench matching parameters (bp and sizefold similarity thresholds) are used, so labels depend on those default match tolerances
- Ground truth is limited to the HGSVC benchmark set (Chaisson et al. 2019) for the specific genomes benchmarked, so calls outside benchmarked regions/types cannot be reliably labelled
- The article does not provide version numbers or implementation details for ConsensuSV-ONT in all sections, and the repository has no changelog, complicating reproducibility

## Evidence

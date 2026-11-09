---
name: vcf-parsing-and-annotation
description: Use when a collapsed SV candidate VCF from ONT long-read consensus calling
  must be converted into a labelled dataset (e.g. to train or evaluate a CNN variant
  filter) using a matched benchmark truth set such as HGSVC.
when_to_use_negative:
- The input is short-read (Illumina/PacBio) SV calls — ConsensuSV-ONT and this labelling
  workflow are designed for Oxford Nanopore long-read data
- No matched ground-truth/benchmark VCF exists for the sample's genome, so no TP/FP
  counterpart comparison is possible
- The VCF already carries validated labels and the goal is model training/evaluation
  rather than label generation
edam_operation: http://edamontology.org/operation_3227
tools:
- name: Truvari
  role: Benchmarks the collapsed candidate SV set against the HGSVC ground-truth VCF
    (default bp and sizefold similarity thresholds) to assign TP/FP labels
- name: ConsensuSV-ONT
  role: Produces the collapsed consensus VCF candidate set (in the vcfs output folder)
    that is parsed and annotated
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Runs the pipeline that generates the alignment files and per-sample VCF folders
    consumed by this skill
- name: Docker
  role: Provides the runtime environment (antonipietryga/consensussv-ont-pipeline
    image) in which the pipeline and its outputs are produced
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:2364d1b4b3a25c0e2efa8cb0bb3872da3d04f0ef738d865753fac1f950fccc2b
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
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# VCF parsing and annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse the per-sample collapsed consensus SV VCFs emitted by the ConsensuSV-ONT pipeline (in the 'vcfs' output folder) and annotate each candidate structural variant with a TP/FP label by benchmarking against HGSVC ground-truth VCFs using Truvari bench with default matching parameters.

## When to use

Use when a collapsed SV candidate VCF from ONT long-read consensus calling must be converted into a labelled dataset (e.g. to train or evaluate a CNN variant filter) using a matched benchmark truth set such as HGSVC.

## When NOT to use

- The input is short-read (Illumina/PacBio) SV calls — ConsensuSV-ONT and this labelling workflow are designed for Oxford Nanopore long-read data
- No matched ground-truth/benchmark VCF exists for the sample's genome, so no TP/FP counterpart comparison is possible
- The VCF already carries validated labels and the goal is model training/evaluation rather than label generation

## Inputs

- Per-sample collapsed consensus SV VCF from the ConsensuSV-ONT pipeline's vcfs output folder
- HGSVC benchmark SV VCF (Chaisson et al. 2019) matched to the sample's genome (e.g. HG00733, HG00514)

## Outputs

- Per-sample labelled table tp_fp_labels.tsv with columns for sample ID, variant coordinates, SV type, and TP/FP label

## How to apply

Load each sample's collapsed consensus VCF from the pipeline's 'vcfs' output folder, where folders per sample hold VCFs from the individual SV callers and ConsensuSV-ONT. Download the matching HGSVC benchmark SV VCF (Chaisson et al. 2019) for the sample's genome (e.g. HG00733, HG00514). Run Truvari bench with default matching parameters — its default bp and sizefold similarity thresholds — to compare candidate variants against the benchmark set, then parse the bench output to classify each candidate as TP (counterpart present in the ground truth) or FP (no counterpart). The rationale is that the CNN filtering stage of ConsensuSV-ONT requires high-quality labels, and a benchmark comparison with default Truvari matching provides an automated, reproducible division of the candidate set into true and false positives.

## Related tools

- **Truvari** (Benchmarks the collapsed candidate SV set against the HGSVC ground-truth VCF (default bp and sizefold similarity thresholds) to assign TP/FP labels)
- **ConsensuSV-ONT** (Produces the collapsed consensus VCF candidate set (in the vcfs output folder) that is parsed and annotated) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Runs the pipeline that generates the alignment files and per-sample VCF folders consumed by this skill)
- **Docker** (Provides the runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the pipeline and its outputs are produced) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
truvari bench -b HG00733_hgsvc_truth.vcf.gz -c HG00733_consensus.vcf -o bench_out/ --passonly
```

## Evaluation signals

- tp_fp_labels.tsv exists for each processed sample
- Every candidate variant in the labelled table carries a TP or FP label (no null labels)
- No candidate rows are dropped: the labelled table's row count matches the number of variants in the input VCF
- Each labelled table contains sample ID, variant coordinates, SV type, and label columns
- Truvari bench was run with its default matching parameters (default bp and sizefold similarity thresholds), not custom thresholds

## Limitations

- Default Truvari matching parameters may not suit all SV size/type regimes; any threshold change departs from the published workflow
- Labelling quality depends on the completeness of the HGSVC benchmark for the sample's genome — variants absent from the truth set are labelled FP, not necessarily wrong
- The README provides no version numbers or changelog for the pipeline, so reproducibility of exact VCF outputs may vary

## Evidence

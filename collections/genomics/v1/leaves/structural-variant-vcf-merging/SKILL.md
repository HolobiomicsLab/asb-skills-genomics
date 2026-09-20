---
name: structural-variant-vcf-merging
description: Use when multiple per-caller structural variant VCFs exist for the same
  ONT sample and a single non-overlapping consensus SV candidate set is needed before
  consensus filtering.
when_to_use_negative:
- When only a single caller's VCF is available — there is nothing to merge and collapse
  across callers.
- When the goal is final high-quality variant selection — that is handled by the downstream
  convolutional neural network filtering step, not by Truvari merge/collapse.
- For short-read (Illumina) variant consensus; ConsensuSV-ONT targets Oxford Nanopore
  long-read sequencing data.
tools:
- name: Truvari
  role: Its merge module combines the per-caller VCFs into one consolidated file;
    its collapse module removes overlapping variants to produce the non-redundant
    candidate set.
- name: ConsensuSV-ONT
  role: The consensus SV pipeline in which the merging/collapse step is embedded,
    taking consensus out of multiple SV callers' results.
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Pipeline framework orchestrating the merge and collapse steps for each sample
    with parallel computing.
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline) wrapping the
    Nextflow pipeline so the merge/collapse step runs reproducibly.
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:bc9a7fac2a28b6bca60efff54296e795d63901d130946c6112744a1106bb10fa
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structural-variant-vcf-merging

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Consolidates the VCF callsets produced by the six independent long-read SV callers into one non-redundant candidate set, using the Truvari merge module to combine records and the Truvari collapse module to remove overlapping variants inside the ConsensuSV-ONT Nextflow/Docker pipeline.

## When to use

Use when multiple per-caller structural variant VCFs exist for the same ONT sample and a single non-overlapping consensus SV candidate set is needed before consensus filtering.

## When NOT to use

- When only a single caller's VCF is available — there is nothing to merge and collapse across callers.
- When the goal is final high-quality variant selection — that is handled by the downstream convolutional neural network filtering step, not by Truvari merge/collapse.
- For short-read (Illumina) variant consensus; ConsensuSV-ONT targets Oxford Nanopore long-read sequencing data.

## Inputs

- Per-caller structural variant VCF files from the six independent long-read SV callers, stored per sample in the vcfs output folders
- Per-sample alignment files (from fastq.gz ONT reads aligned to the reference genome)

## Outputs

- A single merged candidate VCF per sample combining records from all six callers
- A collapsed, non-redundant VCF per sample with overlapping variants removed, written into the sample's folder under the vcfs output directory

## How to apply

Collect the VCF files emitted by the six independent SV calling tools from the per-sample vcfs folders of the ConsensuSV-ONT output directory. First run the Truvari merge module, which combines the per-caller VCF records into one consolidated candidate VCF per sample. Then run the Truvari collapse module on that merged VCF to remove overlapping, redundant variant records, yielding the non-redundant candidate set. Write the resulting collapsed VCF back into the sample's folder under the vcfs output directory; this step is orchestrated by the Nextflow pipeline inside the Docker image (antonipietryga/consensussv-ont-pipeline). The collapsed candidates then feed the downstream CNN-based filtering of high-quality variants.

## Related tools

- **Truvari** (Its merge module combines the per-caller VCFs into one consolidated file; its collapse module removes overlapping variants to produce the non-redundant candidate set.)
- **ConsensuSV-ONT** (The consensus SV pipeline in which the merging/collapse step is embedded, taking consensus out of multiple SV callers' results.) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Pipeline framework orchestrating the merge and collapse steps for each sample with parallel computing.)
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline) wrapping the Nextflow pipeline so the merge/collapse step runs reproducibly.) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
nextflow pipeline.nf --input files.csv --outdir out --ref ref.fa --threads 8 --mem 16
```

## Evaluation signals

- The collapsed VCF contains fewer variant records than the union of the input per-caller VCFs, confirming redundancy removal.
- The collapsed output remains a syntactically valid VCF file.
- A merged (pre-collapse) VCF exists per sample that combines records from all six caller VCFs.
- Each sample's folder under the vcfs output directory contains both the individual-caller VCFs and the collapsed consensus VCF.
- The pipeline step executed inside the Docker/Nextflow environment without missing per-caller inputs.

## Limitations

- The article section text does not name the six individual SV callers, nor provide Truvari version numbers, parameters, or overlap/collapse thresholds used.
- The consolidation is candidate generation only — quality filtering is performed separately by the CNN (3D Conv/MaxPooling/Dense architecture, 59,929 total parameters), not by Truvari.
- No changelog is available in the repository, limiting traceability of pipeline changes.

## Evidence

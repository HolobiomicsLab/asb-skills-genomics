---
name: sv-variant-overlap-collapse
description: Use when a sample has multiple SV caller VCFs (e.g. the six callers of
  ConsensuSV-ONT) that must be merged and deduplicated into one overlapping-variant-free
  candidate VCF before consensus scoring or CNN filtering.
when_to_use_negative:
- Input is already a single consensus VCF that has been through collapse — re-running
  would not reduce overlap and could alter records
- You only have one caller's output and no multi-caller union to consolidate; Truvari
  merge/collapse addresses redundancy across callers, not within-callset calling errors
- The task is quality filtering of consensus variants — that is the separate CNN-based
  filtering step of ConsensuSV-ONT
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3175
tools:
- name: Truvari
  role: Provides the merge module (combines per-caller VCFs into one file) and the
    collapse module (removes overlapping variants to yield the non-redundant candidate
    set)
- name: ConsensuSV-ONT
  role: Pipeline in which this merge-and-collapse consensus step is embedded, following
    six independent SV callers and preceding CNN-based variant filtering
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Orchestrates the merge/collapse steps per sample with parallel computing inside
    the pipeline
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the Nextflow pipeline runs the merge/collapse step
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:a1a300443275310c7809d88cfcb177296d6e023c998f3c8db8aa9da2290fa74d
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# sv-variant-overlap-collapse

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines the per-caller structural variant VCFs produced by six independent ONT SV callers into a single non-redundant candidate set, using the Truvari merge module followed by the Truvari collapse module, as orchestrated inside the ConsensuSV-ONT Nextflow pipeline.

## When to use

Use when a sample has multiple SV caller VCFs (e.g. the six callers of ConsensuSV-ONT) that must be merged and deduplicated into one overlapping-variant-free candidate VCF before consensus scoring or CNN filtering.

## When NOT to use

- Input is already a single consensus VCF that has been through collapse — re-running would not reduce overlap and could alter records
- You only have one caller's output and no multi-caller union to consolidate; Truvari merge/collapse addresses redundancy across callers, not within-callset calling errors
- The task is quality filtering of consensus variants — that is the separate CNN-based filtering step of ConsensuSV-ONT

## Inputs

- Per-sample VCF files from six independent structural variant callers (long-read/ONT callsets)
- Sample fastq.gz-derived alignments processed by the ConsensuSV-ONT Nextflow pipeline

## Outputs

- A merged consolidated candidate VCF per sample (Truvari merge output)
- A non-redundant collapsed candidate VCF per sample, written into the sample's folder under the vcfs output directory

## How to apply

Collect all VCF files produced by the six individual SV calling tools for each sample from the per-sample vcfs folders. Run the Truvari merge module to combine the per-caller VCF records into one consolidated candidate VCF per sample. Then run the Truvari collapse module on that merged VCF to remove overlapping, redundant variant records, yielding the non-redundant candidate set. Write the resulting collapsed VCF into the sample's folder under the vcfs output directory, within the Nextflow pipeline executed inside the Docker image. This step is the consensus-generation stage that precedes CNN-based filtering of high-quality variants.

## Related tools

- **Truvari** (Provides the merge module (combines per-caller VCFs into one file) and the collapse module (removes overlapping variants to yield the non-redundant candidate set))
- **ConsensuSV-ONT** (Pipeline in which this merge-and-collapse consensus step is embedded, following six independent SV callers and preceding CNN-based variant filtering) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Orchestrates the merge/collapse steps per sample with parallel computing inside the pipeline) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the Nextflow pipeline runs the merge/collapse step) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
nextflow pipeline.nf --input files.csv --outdir results --threads 8 --ref ref.fa
```

## Evaluation signals

- The collapsed VCF contains fewer variant records than the union of the input per-caller VCFs (overlap records removed).
- The collapsed output remains a valid, parseable VCF file.
- The output is located in the sample's folder under the vcfs output directory alongside the individual SV calling tools' VCFs.
- Every input per-caller VCF contributed records to the merged intermediate before collapse (merge step retained the union).
- Downstream ConsensuSV-ONT steps (CNN filtering of high-quality variants) receive the collapsed, non-overlapping candidate set as input.

## Limitations

- The article and README do not specify Truvari version numbers, parameter values, or overlap/reciprocal-match thresholds used for merge and collapse.
- No changelog was found for the pipeline repository artifact, complicating version tracking of this step.
- The names of the six individual SV callers are not given in the methods section, so caller-specific compatibility must be verified independently.

## Evidence

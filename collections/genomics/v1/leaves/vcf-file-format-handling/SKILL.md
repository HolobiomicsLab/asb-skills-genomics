---
name: vcf-file-format-handling
description: Use when multiple SV callers each emit their own VCF for the same long-read
  (ONT) sample and a single, non-overlapping candidate VCF is needed as input to consensus
  filtering or scoring.
when_to_use_negative:
- Input is a single VCF from one caller with no overlapping callsets to reconcile
  — Truvari merge/collapse would add no value
- The task is short-variant (SNV/indel) calling from Illumina data rather than structural
  variant consensus from ONT long reads, which is the scope of ConsensuSV-ONT
- VCFs are final, already-collapsed consensus outputs that only need downstream CNN
  filtering of high-quality variants
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Truvari
  role: Provides the merge module (combine per-caller VCFs into one file) and the
    collapse module (remove overlapping variants to yield the non-redundant candidate
    set)
- name: ConsensuSV-ONT
  role: Consensus pipeline whose per-sample vcfs folders hold the caller VCFs that
    Truvari merges and collapses
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Nextflow
  role: Orchestrates the merge/collapse steps inside pipeline.nf with parallel computing
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline) in which the
    Nextflow pipeline and Truvari steps execute
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:0891b2daf32a6441abfb709e9d21e7397e7d9627871f6cc093f3363f48e92676
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
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# vcf-file-format-handling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines and de-replicates structural variant calls delivered in VCF format: per-caller VCFs from the six independent ONT SV callers in ConsensuSV-ONT are merged with Truvari and collapsed into a single non-redundant candidate set per sample.

## When to use

Use when multiple SV callers each emit their own VCF for the same long-read (ONT) sample and a single, non-overlapping candidate VCF is needed as input to consensus filtering or scoring.

## When NOT to use

- Input is a single VCF from one caller with no overlapping callsets to reconcile — Truvari merge/collapse would add no value
- The task is short-variant (SNV/indel) calling from Illumina data rather than structural variant consensus from ONT long reads, which is the scope of ConsensuSV-ONT
- VCFs are final, already-collapsed consensus outputs that only need downstream CNN filtering of high-quality variants

## Inputs

- Six per-caller VCF files (one from each independent SV caller) for each ONT sample
- fastq.gz sample files and a single-column csv sample list (paths to fastq.gz) that drive the upstream pipeline

## Outputs

- One merged candidate VCF per sample (union of all caller records)
- One collapsed, non-redundant candidate VCF per sample, written into the sample's folder under the vcfs output directory

## How to apply

Collect the per-caller VCF files produced by the six independent SV calling tools from each sample's folder under the pipeline's vcfs output directory. Run the Truvari merge module to combine all per-caller VCF records into one consolidated candidate VCF per sample. Then run the Truvari collapse module on the merged VCF to remove overlapping, redundant variant records, yielding the non-redundant candidate set. Write the collapsed VCF back into the sample's folder under the vcfs output directory, orchestrated within the Nextflow pipeline (pipeline.nf, launched with --input files.csv) inside the Docker image. The rationale is that Truvari's merge/collapse produces the unique candidate set that the ConsensuSV-ONT consensus step and CNN-based filtering then operate on.

## Related tools

- **Truvari** (Provides the merge module (combine per-caller VCFs into one file) and the collapse module (remove overlapping variants to yield the non-redundant candidate set))
- **ConsensuSV-ONT** (Consensus pipeline whose per-sample vcfs folders hold the caller VCFs that Truvari merges and collapses) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Nextflow** (Orchestrates the merge/collapse steps inside pipeline.nf with parallel computing) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline) in which the Nextflow pipeline and Truvari steps execute) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/

## Examples

```
nextflow pipeline.nf --input files.csv
```

## Evaluation signals

- The collapsed VCF contains fewer variant records than the union of the input per-caller VCFs (redundant overlapping calls were removed).
- The collapsed output remains a valid VCF file that downstream ConsensuSV-ONT steps can parse.
- The per-sample vcfs output directory contains, for every sample, the individual caller VCFs plus the consensus (collapsed) VCF.
- The merged intermediate VCF contains approximately the union of records from all six caller VCFs before collapse.
- Running `nextflow pipeline.nf --input files.csv` completes inside the Docker container and creates both alignments and vcfs folders in the working/output directory.

## Limitations

- The article does not name the six individual SV callers or give Truvari parameter/version details in the extracted sections, so exact merge/collapse thresholds must be taken from the pipeline source.
- Collapse quality depends on caller VCF correctness upstream; the README describes output layout but no changelog or detailed validation statistics were found in the repository documentation.
- The skill is scoped to ONT long-read structural variant callsets within the ConsensuSV-ONT pipeline environment; no short-read or hybrid workflow is described.

## Evidence

- [other] Truvari merge and collapse of SV caller VCFs: [evidence span withheld — anchor status: not_found]

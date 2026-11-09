---
name: cnn-inference-on-variant-encodings
description: Use when you have per-sample collapsed candidate VCFs from the ConsensuSV-ONT
  nextflow pipeline (vcfs output folder) and need to filter them down to high-quality
  SV calls via CNN scoring before benchmarking with Truvari.
when_to_use_negative:
- Input is short-read (Illumina) SV data — the pipeline and its callers target Oxford
  Nanopore long-read sequencing
- You already have the final filtered ConsensuSV-ONT VCF and only need benchmarking
  (go straight to Truvari bench)
- No trained CNN model is available or the candidates are raw single-caller VCFs that
  have not been collapsed
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: ConsensuSV-ONT
  role: Provides the collapsed candidate VCFs and the trained 3D CNN used to score
    and filter high-quality variants
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the variant encoding and CNN inference run
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Wraps the pipeline steps (calling, collapsing, CNN filtering) with parallel
    computing; launched with --input files.csv
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:d91fb278877b4c19be2eb8659c9182c542666ebd5714d0be26040a42c6703373
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# cnn-inference-on-variant-encodings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores collapsed structural variant candidates from ConsensuSV-ONT by encoding each variant as an image-like tensor and running it through the trained 3D convolutional neural network, keeping variants whose sigmoid output exceeds 0.5 as high-quality consensus calls.

## When to use

Use when you have per-sample collapsed candidate VCFs from the ConsensuSV-ONT nextflow pipeline (vcfs output folder) and need to filter them down to high-quality SV calls via CNN scoring before benchmarking with Truvari.

## When NOT to use

- Input is short-read (Illumina) SV data — the pipeline and its callers target Oxford Nanopore long-read sequencing
- You already have the final filtered ConsensuSV-ONT VCF and only need benchmarking (go straight to Truvari bench)
- No trained CNN model is available or the candidates are raw single-caller VCFs that have not been collapsed

## Inputs

- Per-sample collapsed candidate VCF files from the ConsensuSV-ONT nextflow pipeline vcfs output folder
- Trained 3D convolutional neural network model (Conv3D architecture, 59,929 parameters) shipped with the pipeline
- fastq.gz sample files and a one-column CSV sample list passed via --input

## Outputs

- Filtered consensus VCF (the ConsensuSV-ONT VCF) written into the per-sample vcfs folder
- Per-variant sigmoid scores used for the 0.5 threshold decision

## How to apply

Load the per-sample collapsed candidate VCF files from the vcfs output folder produced inside the antonipietryga/consensussv-ont-pipeline docker image after running `nextflow pipeline.nf --input files.csv`. Encode each collapsed variant candidate into a 50x50x3 image-like representation matching the network's expected input shape, then score it with the trained 3D CNN (Conv3D layers of 4, 8, and 16 filters with batch normalization and max pooling, flatten, dense-32, and a final 1-unit sigmoid output; 59,929 total parameters). Apply the decision threshold: keep variants whose sigmoid output exceeds 0.5 as high-quality calls. Write the filtered consensus calls as the ConsensuSV-ONT VCF into the per-sample vcfs folder alongside the six individual callers' VCF files, so they can be compared against the collapsed set and cnnLSV in Truvari bench runs.

## Related tools

- **ConsensuSV-ONT** (Provides the collapsed candidate VCFs and the trained 3D CNN used to score and filter high-quality variants) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the variant encoding and CNN inference run) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Wraps the pipeline steps (calling, collapsing, CNN filtering) with parallel computing; launched with --input files.csv) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv
```

## Evaluation signals

- Every retained call has a sigmoid score strictly above the 0.5 threshold; discarded candidates score at or below 0.5
- The filtered consensus VCF appears in the per-sample vcfs folder next to the six individual callers' VCF files
- Truvari bench on the output reproduces Table 1 values, e.g. F1 ConsensuSV-ONT DEL NA19240 = 0.6733 and INS NA19240 = 0.6487
- Precision and F1 of the consensus calls exceed each of the six individual callers and cnnLSV, while the unfiltered collapsed set shows high recall but low F1
- Loaded model summary matches the documented architecture: 50x50x3 input, Conv3D 4→8→16 filters, dense-32, dense-1, 59,929 total parameters

## Limitations

- The README documents the CNN architecture but not the exact procedure for encoding variants into the 50x50x3 image representation
- The sigmoid threshold of 0.5 is fixed by the task recipe; no parameter or tuning guidance is given in the article or README
- The method is specific to ONT long-read data and the six bundled SV callers; performance on other platforms or caller sets is not established

## Evidence

- [methods] CNN scores collapsed candidates with 0.5 threshold: "The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants."
- [readme] Consensus tool purpose: "The tool designed for getting consensus out of multiple SV callers' results."
- [readme] CNN input shape and first layer confirm 50x50x3 encoding: "conv3d (Conv3D)             (None, 50, 50, 3, 4)      112"

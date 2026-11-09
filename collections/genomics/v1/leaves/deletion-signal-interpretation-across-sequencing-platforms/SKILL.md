---
name: deletion-signal-interpretation-across-sequencing-platforms
description: Use when comparing how the same deletion locus (e.g. chr12:121,792,102-121,792,155
  or chr5:114,989,663-114,999,300) appears in alignment-derived image encodings across
  PacBio, Illumina, and ONT sequencing technologies.
when_to_use_negative:
- Input is raw fastq.gz without alignments — the encoding step requires pre-built
  BAM alignments (the pipeline generates these internally from fastq.gz plus a csv
  sample list)
- The goal is calling novel structural variants in a new sample rather than interpreting/validating
  how known deletions are encoded across platforms
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Provides the 50x50x3 variant-encoding algorithm and the convolutional neural
    network used to score and inspect deletion image encodings
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment image (antonipietryga/consensussv-ont-pipeline) hosting
    the encoding pipeline
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline runner executing the encoding and calling workflow inside the Docker
    container
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:c99a7c81b45c2796ac27c2867cc22c6e30b3dd643a09c7ec06f7c10badfb813d
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
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# deletion-signal-interpretation-across-sequencing-platforms

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret how a structural-variant deletion manifests differently in image encodings derived from short-read (Illumina) versus long-read (PacBio, ONT) alignments, using the ConsensuSV-ONT 50x50x3 encoding and CNN as the reference framework.

## When to use

Use when comparing how the same deletion locus (e.g. chr12:121,792,102-121,792,155 or chr5:114,989,663-114,999,300) appears in alignment-derived image encodings across PacBio, Illumina, and ONT sequencing technologies.

## When NOT to use

- Input is raw fastq.gz without alignments — the encoding step requires pre-built BAM alignments (the pipeline generates these internally from fastq.gz plus a csv sample list)
- The goal is calling novel structural variants in a new sample rather than interpreting/validating how known deletions are encoded across platforms

## Inputs

- PacBio pbmm2 BAM alignment files over the deletion loci
- Illumina and ONT minimap2 BAM alignment files over the deletion loci
- Deletion coordinates: chr12:121,792,102-121,792,155 (53 bp) and chr5:114,989,663-114,999,300 (~10,000 bp)

## Outputs

- Six 50x50x3 image encodings (one per sequencing-technology/deletion combination)
- Predicted variant-quality probabilities from the ConsensuSV-ONT CNN for PacBio and ONT encodings
- Qualitative comparison noting the Illumina absence-of-mapping signal for the large chr5 deletion

## How to apply

Retrieve public alignments (PacBio pbmm2 BAM; Illumina and ONT minimap2 BAM) covering the deletion loci, then apply the ConsensuSV-ONT variant-encoding algorithm around chr12:121,792,102-121,792,155 and chr5:114,989,663-114,999,300 to generate 50x50x3 image encodings per technology/deletion combination (six images total). Feed each encoding into the ConsensuSV-ONT sequential CNN (59,929 total parameters, Conv3D/BatchNormalization/MaxPooling3D blocks, single-unit Dense output) and record the predicted variant-quality probability. Inspect the Illumina encoding of the large chr5 deletion specifically: it should exhibit absence-of-mapping (coverage loss over the ~10,000 bp deletion) rather than the split-read-style deletion signal visible in the long-read encodings.

## Related tools

- **ConsensuSV-ONT** (Provides the 50x50x3 variant-encoding algorithm and the convolutional neural network used to score and inspect deletion image encodings) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment image (antonipietryga/consensussv-ont-pipeline) hosting the encoding pipeline) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline runner executing the encoding and calling workflow inside the Docker container) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline
```

## Evaluation signals

- Exactly six 50x50x3 images are produced (three technologies x two deletions)
- The Illumina encoding of chr5:114,989,663-114,999,300 shows absence-of-mapping (coverage loss) in the central region, not a split-read-style deletion signal
- CNN architecture check: sequential model with 59,929 total parameters and a single-unit Dense output layer
- PacBio and ONT encodings for both deletions display a central deletion-style signal consistent with long-read split-read mapping

## Limitations

- The paper's discussion notes there is a need for a more detailed comparative analysis of mutation detection between different sequencing methods — cross-platform signal interpretation here is illustrative, not exhaustive
- The algorithm works with fastq.gz inputs and a csv sample list (single path column, no header); deviating formats break the pipeline
- Names of the six individual SV callers are not given in the article's methods section

## Evidence

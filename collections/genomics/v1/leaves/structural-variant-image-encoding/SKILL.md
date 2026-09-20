---
name: structural-variant-image-encoding
description: Use when you have ONT/PacBio/Illumina read alignments (BAM) around candidate
  SV loci and need to convert them into fixed-size 50x50x3 image encodings for CNN-based
  variant quality filtering or cross-technology encoding comparison.
when_to_use_negative:
- Input is already a VCF of final consensus calls and no variant-quality re-filtering
  via the CNN is needed
- Data are short-read-only and the goal is small indel detection — the article scopes
  the method to Oxford Nanopore long-read SV detection
- You need the raw per-caller variant lists only, without CNN-based quality encoding/filtering
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Provides the variant-encoding algorithm that converts read-mapping data around
    SV loci into 50x50x3 images for the CNN
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment image (antonipietryga/consensussv-ont-pipeline) within
    which the encoding and CNN run
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline wrapper that executes encoding and variant calling with parallel
    computing
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: TensorFlow/Keras
  role: Runs the sequential Conv3D CNN that consumes the image encodings and outputs
    variant-quality probability
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:772a730d41404228ce65e2a0813ee8411b63d7eef58b879607593357dfab1eb9
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
  - TensorFlow/Keras
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# structural-variant-image-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes structural variant loci from read alignments into 50x50x3 image tensors so a convolutional neural network (ConsensuSV-ONT's 59,929-parameter sequential model) can filter high-quality SV calls as a computer-vision problem.

## When to use

Use when you have ONT/PacBio/Illumina read alignments (BAM) around candidate SV loci and need to convert them into fixed-size 50x50x3 image encodings for CNN-based variant quality filtering or cross-technology encoding comparison.

## When NOT to use

- Input is already a VCF of final consensus calls and no variant-quality re-filtering via the CNN is needed
- Data are short-read-only and the goal is small indel detection — the article scopes the method to Oxford Nanopore long-read SV detection
- You need the raw per-caller variant lists only, without CNN-based quality encoding/filtering

## Inputs

- BAM alignment files per sample (ONT/PacBio long reads; minimap2 or pbmm2 alignments)
- fastq.gz sample files plus a csv sample list (one column of paths, no header) as pipeline input
- Reference genome (--ref parameter)

## Outputs

- 50x50x3 image encodings of each structural variant locus per sequencing technology
- Encoded SV images with high-quality/low-quality labels for CNN training/filtering
- VCF files per sample in the vcfs/ output folder (consensus and per-caller)

## How to apply

Apply the ConsensuSV-ONT variant-encoding algorithm to read-mapping data around each candidate SV locus (e.g. chr12:121,792,102-121,792,155 and chr5:114,989,663-114,999,300) to produce one 50x50x3 image per technology/deletion combination. The three channels capture read-mapping features around the variant, with long reads (PacBio, ONT) showing split-read-style deletion signals while Illumina encodes absence of mapping where reads fail to span the deletion. Feed each encoding directly into the sequential Keras CNN (three Conv3D+BatchNorm+MaxPooling3D blocks with 4/8/16 filters, Flatten to 1728, Dense(32), Dense(1) sigmoid output) which outputs a variant-quality probability. The pipeline is delivered as a docker image wrapping a nextflow pipeline, so encoding runs automatically as part of 'nextflow pipeline.nf --input files.csv'. Correct application is confirmed by checking that exactly the expected number of 50x50x3 images are produced and that technology-specific signatures (coverage loss vs split-read signal) are visible in the encodings.

## Related tools

- **ConsensuSV-ONT** (Provides the variant-encoding algorithm that converts read-mapping data around SV loci into 50x50x3 images for the CNN) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment image (antonipietryga/consensussv-ont-pipeline) within which the encoding and CNN run) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline wrapper that executes encoding and variant calling with parallel computing) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **TensorFlow/Keras** (Runs the sequential Conv3D CNN that consumes the image encodings and outputs variant-quality probability)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv --outdir consensuSV-ONT --ref ref.fa
```

## Evaluation signals

- Exactly the expected number of 50x50x3 image encodings is produced (e.g. six images for two deletions x three technologies)
- The CNN model summary reports 59,929 total parameters (59,873 trainable, 56 non-trainable) with Flatten output of 1728 units
- The Illumina encoding of a large (~10,000 bp) deletion such as chr5:114,989,663-114,999,300 shows lack of data / coverage loss rather than a split-read-style deletion signal
- Long-read (PacBio, ONT) encodings of the same locus show the split-read-style deletion signal in the central part of the image
- alignments/ and vcfs/ folders are created in the working directory after running the nextflow pipeline

## Limitations

- Encoding behaviour is technology-dependent: for large deletions (~10,000 bp) the Illumina encoding indicates absence of data due to reads not mapping across the deletion, so cross-technology encodings are not interchangeable
- The article's discussion notes no version numbers, parameters, or implementation details are given for ConsensuSV-ONT in that section, and no changelog is available in the repository
- Images are only interpretable by the paired CNN architecture; the encoding shape (50x50x3) must match the model input

## Evidence

- [results] For the large deletion of approximately 10,000 bp on chr5, the Illumina encoding differs from the others: it does not show the deletion in the central part of the image, instead indicating lack of data due to reads not mapping along the deletion length.: [evidence span withheld — anchor status: not_found]

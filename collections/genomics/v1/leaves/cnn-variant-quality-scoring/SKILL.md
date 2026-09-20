---
name: cnn-variant-quality-scoring
description: Use when Oxford Nanopore long-read SV candidate calls from the six ConsensuSV-ONT
  callers need a quality score, i.e. candidate variants must be filtered via image-encoded
  evidence into a CNN probability.
when_to_use_negative:
- The input data are short reads (e.g. Illumina-only alignments) — the paper positions
  this skill for ONT long-read SV detection; note the source card shows Illumina encodings
  of a large deletion lose coverage signal rather than split-read evidence.
- You only have raw fastq.gz and expect final variant calls without running the six
  SV callers first — the CNN scores candidate variants produced by the consensus stage,
  not raw reads.
- You need caller names, version numbers, or threshold parameters not documented here
  — the article notes such implementation details are missing.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: ConsensuSV-ONT
  role: Provides the six SV callers whose consensus candidates are encoded as images
    and scored by the 59,929-parameter Conv3D CNN for quality filtering
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment image (antonipietryga/consensussv-ont-pipeline) that packages
    the pipeline and CNN so scoring runs reproducibly
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline orchestrator launched inside the docker container to run calling,
    encoding, and CNN scoring with parallel computing
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:24076f098707d24f60b411bb57967d2953bc0e171792621336b6b1678c1d69fd
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cnn-variant-quality-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Conv3D-based CNN module of ConsensuSV-ONT that encodes candidate structural variants from Oxford Nanopore long-read alignments into 50x50x3 images and scores each with a variant-quality probability for filtering high-quality calls.

## When to use

Use when Oxford Nanopore long-read SV candidate calls from the six ConsensuSV-ONT callers need a quality score, i.e. candidate variants must be filtered via image-encoded evidence into a CNN probability.

## When NOT to use

- The input data are short reads (e.g. Illumina-only alignments) — the paper positions this skill for ONT long-read SV detection; note the source card shows Illumina encodings of a large deletion lose coverage signal rather than split-read evidence.
- You only have raw fastq.gz and expect final variant calls without running the six SV callers first — the CNN scores candidate variants produced by the consensus stage, not raw reads.
- You need caller names, version numbers, or threshold parameters not documented here — the article notes such implementation details are missing.

## Inputs

- fastq.gz read files for each sample plus a csv sample list (single column of fastq.gz paths, no header row)
- Reference genome (--ref)
- Consensus/candidate structural variant calls from the six SV callers run on ONT alignments
- 50x50x3 variant image encodings derived from read-mapping data around each candidate SV

## Outputs

- Variant-quality probability (single value from the dense_1 Dense(1) output layer) per candidate SV
- Filtered set of high-quality structural variants alongside the unfiltered VCFs in the vcfs output folder

## How to apply

After the six independent SV callers produce consensus candidate SVs, encode the read-mapping evidence around each variant into a 50x50x3 image (per source card: e.g. 53 bp deletion at chr12:121,792,102-121,792,155 and ~10 kb deletion at chr5:114,989,663-114,999,300 each yield one image per technology). Feed each encoding through the sequential CNN — Conv3D(4 filters) → BatchNormalization → MaxPooling3D → Conv3D(8) → BatchNormalization → MaxPooling3D → Conv3D(16) → BatchNormalization → MaxPooling3D → Flatten(1728) → Dense(32) → Dense(1), 59,929 total parameters (59,873 trainable). The single-unit dense_1 output returns the variant-quality probability used to retain high-quality SVs. Run the whole flow via the provided docker image wrapping the nextflow pipeline so consensus calling, encoding, and scoring execute together.

## Related tools

- **ConsensuSV-ONT** (Provides the six SV callers whose consensus candidates are encoded as images and scored by the 59,929-parameter Conv3D CNN for quality filtering) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment image (antonipietryga/consensussv-ont-pipeline) that packages the pipeline and CNN so scoring runs reproducibly) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline orchestrator launched inside the docker container to run calling, encoding, and CNN scoring with parallel computing) — github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline; nextflow pipeline.nf --input files.csv
```

## Evaluation signals

- Each input encoding must have shape 50x50x3 to match the Conv3D input (None, 50, 50, 3, 4).
- Loaded model must report Total params: 59,929 and Trainable params: 59,873, matching the README architecture summary.
- Flatten layer output must equal 1728 features (6*6*16*3) before the Dense(32)/Dense(1) head; the final dense_1 layer must have 33 parameters.
- The CNN output must be a single probability per candidate SV (Dense with None,1 output) suitable for threshold-based high-quality filtering.
- Per the source card, six 50x50x3 images (PacBio/Illumina/ONT x two deletions) should be produced for the reproducibility test, with the Illumina chr5 encoding showing coverage loss rather than the long-read deletion signal.

## Limitations

- Missing implementation details: no changelog, version numbers, or thresholds documented for the pipeline (repository artifact gap).
- The names and settings of the six individual SV callers are not given in the methods text, limiting reproducibility checks of the consensus feeding the CNN.
- The paper flags a need for more detailed comparative analysis across tools and sequencing methods, so cross-technology CNN behavior (e.g. Illumina vs ONT encodings) is not fully characterized.

## Evidence

- [abstract] The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants.: "The method uses six independent, state-of-the-art structural variant callers for long-read sequencing along with a convolutional neural network for filtering high-quality variants."
- [readme] Total params: 59,929 Trainable params: 59,873 Non-trainable params: 56: "Total params: 59,929
Trainable params: 59,873
Non-trainable params: 56"
- [discussion] we propose ConsensuSV-ONT tool for long-read SV detection that uses a consensus of independent tools: "we propose ConsensuSV-ONT tool for long-read SV detection that uses a consensus of independent tools"
- [readme] nextflow pipeline.nf --input files.csv: "nextflow pipeline.nf --input files.csv"

---
name: multi-platform-alignment-comparison
description: Use when comparing how the same structural variant locus is encoded visually
  across sequencing platforms, e.g. verifying that short-read (Illumina) encodings
  show coverage loss rather than the split-read-style deletion signal of long reads.
when_to_use_negative:
- Input data is short-read only and the goal is de novo SV calling rather than cross-platform
  encoding comparison (Illumina cannot produce the long-read split-read-style deletion
  encoding)
- Variant calls are needed for whole-genome samples rather than inspection of specific
  pre-identified deletion loci
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: ConsensuSV-ONT
  role: Provides the variant-to-50x50x3-image encoding algorithm and the CNN used
    to score the per-platform encodings
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the encoding pipeline is executed
- name: Nextflow
  role: Pipeline framework wrapping the encoding/CNN processing with parallel computing
- name: PacBio
  role: Long-read sequencing platform whose alignments supply one encoding of the
    two deletion loci
- name: Illumina
  role: Short-read sequencing platform whose alignments supply the contrasting encoding
    (coverage loss for the large deletion)
- name: Oxford Nanopore (ONT) sequencing technology
  role: Long-read sequencing platform whose alignments supply the third per-locus
    encoding
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:c5e0503e55ad683f13f41e340df346ad6a10c52e761867804edb5e046888ce64
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
  - Illumina
  - Nextflow
  - Oxford Nanopore (ONT) sequencing technology
  - PacBio
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multi-platform-alignment-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the cross-platform (PacBio, Illumina, ONT) image encoding comparison of two example deletions (Fig. 2 of ConsensuSV-ONT), showing how structural variant signal differs between short-read and long-read technologies in 50x50x3 CNN input images.

## When to use

Use when comparing how the same structural variant locus is encoded visually across sequencing platforms, e.g. verifying that short-read (Illumina) encodings show coverage loss rather than the split-read-style deletion signal of long reads.

## When NOT to use

- Input data is short-read only and the goal is de novo SV calling rather than cross-platform encoding comparison (Illumina cannot produce the long-read split-read-style deletion encoding)
- Variant calls are needed for whole-genome samples rather than inspection of specific pre-identified deletion loci

## Inputs

- PacBio pbmm2 alignment BAM files at chr12:121,792,102-121,792,155 and chr5:114,989,663-114,999,300
- Illumina and ONT minimap2 alignment BAM files at the same two loci
- ConsensuSV-ONT convolutional neural network (sequential, 59,929 total parameters, single-unit Dense output)

## Outputs

- Six 50x50x3 variant image encodings (one per technology/deletion combination)
- Predicted variant-quality probabilities from the CNN for the PacBio and ONT encodings
- Qualitative comparison of Illumina vs long-read encodings showing coverage-loss vs split-read-style deletion signal

## How to apply

Retrieve public alignments (Chaisson et al. 2019) covering the two deletion loci: PacBio (pbmm2 BAM) and Illumina plus ONT (minimap2). Apply the ConsensuSV-ONT variant-encoding algorithm to the read-mapping data around chr12:121,792,102-121,792,155 (53 bp deletion) and chr5:114,989,663-114,999,300 (~10,000 bp deletion) to produce a 50x50x3 image per technology/deletion combination (six images total). Feed each encoding into the ConsensuSV-ONT CNN (sequential architecture, 59,929 total parameters, single-unit Dense output) and record the predicted variant-quality probabilities for the PacBio and ONT encodings. Rationale: the comparison demonstrates that for the large chr5 deletion the Illumina encoding lacks the central deletion signal because reads do not map along the deletion length, unlike long-read encodings.

## Related tools

- **ConsensuSV-ONT** (Provides the variant-to-50x50x3-image encoding algorithm and the CNN used to score the per-platform encodings) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the encoding pipeline is executed)
- **Nextflow** (Pipeline framework wrapping the encoding/CNN processing with parallel computing)
- **PacBio** (Long-read sequencing platform whose alignments supply one encoding of the two deletion loci)
- **Illumina** (Short-read sequencing platform whose alignments supply the contrasting encoding (coverage loss for the large deletion))
- **Oxford Nanopore (ONT) sequencing technology** (Long-read sequencing platform whose alignments supply the third per-locus encoding)

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline && nextflow pipeline.nf --input files.csv
```

## Evaluation signals

- Exactly six 50x50x3 images are produced (three technologies x two deletions)
- The Illumina encoding of chr5:114,989,663-114,999,300 shows absence of mapping (coverage loss along the deletion) rather than the split-read-style deletion signal seen in long reads
- PacBio and ONT encodings of the ~10 kb chr5 deletion show the deletion signal in the central part of the image
- The CNN scores each encoding via a sequential model with 59,929 total parameters and a single-unit Dense output, yielding a variant-quality probability

## Limitations

- The article notes there is a need for a more detailed comparative analysis of mutation detection using different tools and between different sequencing methods
- The comparison is restricted to two example deletion loci and does not constitute a systematic cross-platform benchmark
- The short-read encoding cannot represent deletions spanning read length because Illumina reads do not map along the deletion length

## Evidence

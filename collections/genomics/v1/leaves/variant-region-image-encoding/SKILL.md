---
name: variant-region-image-encoding
description: Use when you have a consensus VCF of candidate SVs plus a minimap2 BAM
  of quality-passing ONT reads and need image tensors (50x50x3) as CNN input for high-quality
  variant filtering.
when_to_use_negative:
- Input is short-read (Illumina/PacBio) data rather than Oxford Nanopore long-read
  sequencing, which is the scope of ConsensuSV-ONT
- You already have feature tables or per-variant quality scores and do not need image-encoded
  CNN filtering
- No alignment BAM exists yet — run the alignment-generation step of the ConsensuSV-ONT
  pipeline first
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Pipeline whose consensus VCF supplies the candidate SVs being encoded into
    images; encoding is an internal step of its CNN-based filtering module
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which
    the encoding and CNN filtering steps execute
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline engine that orchestrates variant calling through image encoding to
    CNN filtering with parallel computing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:749e1d82fdb43a7a7c272def022b8fba704e797ae889d2dfb6ecc7bfb4c7bd61
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# variant-region-image-encoding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes candidate structural variants from ConsensuSV-ONT into fixed 50x50x3 images built from per-read alignment features over the variant region (expanded by the variant size on both sides), so a CNN can filter high-quality SV calls.

## When to use

Use when you have a consensus VCF of candidate SVs plus a minimap2 BAM of quality-passing ONT reads and need image tensors (50x50x3) as CNN input for high-quality variant filtering.

## When NOT to use

- Input is short-read (Illumina/PacBio) data rather than Oxford Nanopore long-read sequencing, which is the scope of ConsensuSV-ONT
- You already have feature tables or per-variant quality scores and do not need image-encoded CNN filtering
- No alignment BAM exists yet — run the alignment-generation step of the ConsensuSV-ONT pipeline first

## Inputs

- Consensus candidate SV list as VCF (per-candidate coordinates and variant types) from ConsensuSV-ONT
- minimap2 alignment BAM (Chaisson et al.) of quality-passing long reads, including split-read records
- Sample FASTQ files (fastq.gz) plus csv sample list if generating alignments upstream

## Outputs

- One 50x50x3 image per candidate variant (PNG or NPZ), named by its variant identifier
- Image directory consumed by the CNN-based high-quality variant filter

## How to apply

Parse the candidate SV list (VCF from the consensus caller) to obtain per-candidate coordinates and variant types. For each candidate, use pysam to pull quality-passing reads (including split-read records) whose alignments overlap the variant region extended by flanking sequence equal to the size of the mutation on both sides, using the Chaisson et al. minimap2 BAM. Build a raw read-by-position matrix where each row is a read, each column a genomic position, and cell values encode per-position alignment features. Resize and normalize this matrix with OpenCV into a fixed 50x50x3 image matching the CNN input shape, then write one PNG or NPZ per variant, named by its variant identifier.

## Related tools

- **ConsensuSV-ONT** (Pipeline whose consensus VCF supplies the candidate SVs being encoded into images; encoding is an internal step of its CNN-based filtering module) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment (antonipietryga/consensussv-ont-pipeline image) in which the encoding and CNN filtering steps execute) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline engine that orchestrates variant calling through image encoding to CNN filtering with parallel computing)

## Evaluation signals

- The output directory contains exactly one 50x50x3 image per candidate SV — image count equals the length of the candidate VCF list
- Each saved NPZ array or PNG loads with the expected shape (50, 50, 3) and expected format
- Every image file name matches a variant identifier present in the candidate SV list (no missing or orphan images)
- The encoded images are directly accepted by the CNN (Conv3D layers expecting 50x50x3 inputs, as printed in the README model summary)

## Limitations

- No version numbers or detailed parameters for the encoding step are given in the paper's discussion of ConsensuSV-ONT
- Repository provides no changelog, so implementation changes to the encoding may be undocumented
- The paper's missing-signals list notes names of the six individual SV callers are not given, limiting upstream provenance of the candidates being encoded

## Evidence

- [methods] Each candidate variant is encoded by extracting quality-passing reads mapped to the mutation region and its surroundings, with the window expanded by the size of the mutation on both sides, forming the raw read-by-position matrix later resized to a 50x50x3 image.: [evidence span withheld — anchor status: not_found]

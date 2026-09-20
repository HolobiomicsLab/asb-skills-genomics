---
name: read-by-position-matrix-construction
description: 'Use when you need per-variant image inputs for CNN filtering of ONT
  structural variants: given a consensus VCF of candidate SVs and a Chaisson et al.
  minimap2 BAM, build one alignment image per candidate.'
when_to_use_negative:
- Input is already the consensus VCF-free output of individual SV callers without
  consensus coordinates — the matrix construction is anchored on per-candidate coordinates
  from the consensus list
- Sequencing data is short-read (Illumina) rather than Oxford Nanopore long-read data;
  the pipeline targets ONT long-read SV detection
- You need per-position variant statistics or genotype calls rather than image-encoded
  candidates for CNN filtering
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ConsensuSV-ONT
  role: Supplies the candidate SV list and wraps the image-encoding step that feeds
    its CNN filter
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: Docker
  role: Runtime environment containing the pipeline (pysam/OpenCV dependencies) for
    matrix construction
  repo: https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- name: Nextflow
  role: Pipeline engine running the encoding workflow with parallel computing across
    samples/variants
  repo: https://github.com/SFGLab/ConsensuSV-ONT-pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:ae7b1e1e8838904704f95a98f265236284259cfcdf5476e6511ef647fd3431cb
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# read-by-position-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Encodes each structural variant candidate from ConsensuSV-ONT as a raw read-by-position alignment matrix — one row per quality-passing read, one column per genomic position — which is resized with OpenCV into the fixed 50x50x3 image consumed by the CNN-based high-quality variant filter.

## When to use

Use when you need per-variant image inputs for CNN filtering of ONT structural variants: given a consensus VCF of candidate SVs and a Chaisson et al. minimap2 BAM, build one alignment image per candidate.

## When NOT to use

- Input is already the consensus VCF-free output of individual SV callers without consensus coordinates — the matrix construction is anchored on per-candidate coordinates from the consensus list
- Sequencing data is short-read (Illumina) rather than Oxford Nanopore long-read data; the pipeline targets ONT long-read SV detection
- You need per-position variant statistics or genotype calls rather than image-encoded candidates for CNN filtering

## Inputs

- VCF of consensus structural variant candidates (from the ConsensuSV-ONT caller consensus)
- Chaisson et al. minimap2 BAM alignment file for the sample (including split-read records)
- Per-candidate variant coordinates and types parsed from the VCF

## Outputs

- One 50x50x3 image per candidate variant, saved as PNG or NPZ, named by its variant identifier
- Raw read-by-position alignment matrices (intermediate), resized and normalized with OpenCV

## How to apply

Parse the candidate SV list (VCF from the consensus caller) to get per-candidate coordinates and variant types. For each candidate, use pysam to pull quality-passing reads — including split-read records — whose alignments overlap the mutation region expanded by the size of the mutation on both sides, from the minimap2 BAM. Build the raw read-by-position matrix where each row is a read, each column a genomic position, and cell values encode per-position alignment features. Resize and normalize the matrix with OpenCV into the fixed 50x50x3 shape expected by the ConsensuSV-ONT CNN (whose first Conv3D layer has output shape (None, 50, 50, 3, 4)), then write one PNG or NPZ per variant, named by its variant identifier. Rationale: a fixed-size image representation lets a single convolutional network score candidates produced by the six independent SV callers regardless of SV length.

## Related tools

- **ConsensuSV-ONT** (Supplies the candidate SV list and wraps the image-encoding step that feeds its CNN filter) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline
- **Docker** (Runtime environment containing the pipeline (pysam/OpenCV dependencies) for matrix construction) — https://hub.docker.com/repository/docker/antonipietryga/consensussv-ont-pipeline/
- **Nextflow** (Pipeline engine running the encoding workflow with parallel computing across samples/variants) — https://github.com/SFGLab/ConsensuSV-ONT-pipeline

## Examples

```
docker run -v /mnt:/mnt -p 8082:8082 -it antonipietryga/consensussv-ont-pipeline
```

## Evaluation signals

- The output directory contains exactly one 50x50x3 image per candidate — the image count equals the candidate list length
- Each saved array/PNG loads with the expected shape (50, 50, 3) and format (PNG or NPZ)
- Every image's window corresponds to the mutation region expanded by the size of the mutation on both sides
- Reads contributing rows are quality-passing and include split-read records from the minimap2 BAM
- Images match the input shape of the CNN model summary, whose first conv3d layer expects (None, 50, 50, 3)

## Limitations

- The article does not specify the per-position alignment feature values encoded in matrix cells, nor OpenCV interpolation/normalization parameters
- Encoding depends on the Chaisson et al. minimap2 BAM; other aligners' outputs are outside the described workflow
- No changelog was found in the repository, so implementation drift around the encoding cannot be tracked from repository artifacts

## Evidence

- [methods] Each candidate variant is encoded by extracting quality-passing reads mapped to the mutation region and its surroundings, with the window expanded by the size of the mutation on both sides: [evidence span withheld — anchor status: not_found]

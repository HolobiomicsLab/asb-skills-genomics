---
name: opencv-image-resizing
description: Use when structural variant candidate regions have been converted to
  raw read-by-position matrices (rows = reads from a BAM, columns = genomic positions)
  and a fixed-size CNN input image (50x50x3) is required.
when_to_use_negative:
- Input is already a fixed-shape image (e.g. previously saved 50x50x3 PNG/NPZ) — re-resizing
  would double-resample the alignment features.
- The reads have not yet been extracted into a raw read-by-position matrix; this skill
  resizes matrices, it does not pull alignments from the BAM.
- Filtering is being done by rule-based VCF criteria rather than the ConsensuSV-ONT
  CNN, which is the only consumer of the 50x50x3 encoding.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OpenCV
  role: Resizes and normalizes the raw read-by-position matrix into the fixed 50x50x3
    image matching the CNN input shape
- name: ConsensuSV-ONT
  role: Defines the encoding scheme and consumes the resized images in its Conv3D-based
    CNN for filtering high-quality variants
  repo: github.com/SFGLab/ConsensuSV-ONT-pipeline
- name: pysam
  role: Extracts quality-passing reads (including split-read records) overlapping
    the variant region to build the raw matrix prior to resizing
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-025-01486-1
    title: ConsensuSV-ONT – A modern method for accurate structural variant calling
  source_content_hash: sha256:246d9c6d72ccf940baa9d9bbe2da8439edade05dac147ff98e542af0946d6b3a
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
  - OpenCV
  - pysam
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# opencv-image-resizing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resizes raw per-variant read-by-position alignment matrices into fixed-shape 50x50x3 images so they can be consumed by the ConsensuSV-ONT convolutional neural network that filters high-quality structural variants from Oxford Nanopore long-read data.

## When to use

Use when structural variant candidate regions have been converted to raw read-by-position matrices (rows = reads from a BAM, columns = genomic positions) and a fixed-size CNN input image (50x50x3) is required.

## When NOT to use

- Input is already a fixed-shape image (e.g. previously saved 50x50x3 PNG/NPZ) — re-resizing would double-resample the alignment features.
- The reads have not yet been extracted into a raw read-by-position matrix; this skill resizes matrices, it does not pull alignments from the BAM.
- Filtering is being done by rule-based VCF criteria rather than the ConsensuSV-ONT CNN, which is the only consumer of the 50x50x3 encoding.

## Inputs

- Raw read-by-position alignment matrix per SV candidate (rows = quality-passing reads from Chaisson minimap2 BAM, columns = genomic positions in the variant region plus flanks expanded by the mutation size)
- Per-candidate coordinates and variant types from the consensus VCF

## Outputs

- One 50x50x3 normalized image per SV candidate, written as PNG or NPZ and named by its variant identifier

## How to apply

For each candidate SV, first build the raw matrix from quality-passing reads (including split-read records) pulled with pysam from the minimap2 BAM over the variant region expanded by the mutation size on both sides. Then apply OpenCV (cv2.resize) to resample this variable-width, variable-depth matrix to the fixed 50x50 shape expected by the CNN, normalizing cell values as alignment features across the 3 channels. Save the result per variant as a PNG or NPZ file named by its variant identifier. The rationale is that the downstream CNN (Conv3D layers with output shapes (None, 50, 50, 3, 4) in the README's model summary) requires a constant input tensor shape, so every candidate must be resampled to exactly 50x50x3 regardless of read coverage or variant size.

## Related tools

- **OpenCV** (Resizes and normalizes the raw read-by-position matrix into the fixed 50x50x3 image matching the CNN input shape)
- **ConsensuSV-ONT** (Defines the encoding scheme and consumes the resized images in its Conv3D-based CNN for filtering high-quality variants) — github.com/SFGLab/ConsensuSV-ONT-pipeline
- **pysam** (Extracts quality-passing reads (including split-read records) overlapping the variant region to build the raw matrix prior to resizing)

## Examples

```
import cv2; img = cv2.resize(read_by_position_matrix, (50, 50), interpolation=cv2.INTER_LINEAR)
```

## Evaluation signals

- The output directory contains exactly one image per candidate SV — the count equals the length of the candidate VCF list.
- Every saved PNG or NPZ loads with shape (50, 50, 3), matching the CNN's first-layer input shape shown in the README model summary.
- Each file is named by its variant identifier, allowing round-trip mapping back to the consensus VCF records.
- Resized images derive from reads that pass quality filtering, including split-read records, over windows expanded by the mutation size on both sides.
- Pixel values are normalized alignment features rather than raw byte intensities, so no cell should exceed the expected normalized feature range.

## Limitations

- Resampling to a fixed 50x50 shape discards resolution for large variants or high-coverage regions, compressing many reads/positions into few pixels.
- The article does not publish the exact interpolation or normalization parameters used by OpenCV, so exact reproduction of the images may require the repository code.
- No version numbers or implementation details for the encoding step are given in the article; the README documents the CNN but not the image-generation internals.

## Evidence

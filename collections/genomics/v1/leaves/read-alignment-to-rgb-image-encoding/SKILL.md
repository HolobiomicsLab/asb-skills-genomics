---
name: read-alignment-to-rgb-image-encoding
description: Use when long-read SAM/BAM alignments must be converted into fixed-size
  RGB images (100x100) encoding CIGAR deletion features, e.g., as image-modality input
  for a dual-attention structural-variant-calling model.
when_to_use_negative:
- The goal is insertion or base substitution variant calling — the paper states it
  'currently focuses exclusively on experiments related to deletion variants calling'.
- Input is raw FASTQ/basecalled reads without an existing alignment (SAM/BAM) and
  CIGAR fields.
- A schema-preserving numeric feature representation is required rather than a rasterized
  image modality (e.g., for non-vision models).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: DASV
  role: 'Provides the image-conversion pipeline: CIGAR-based deletion extraction,
    sliding-window tiling (window 100, stride 50), and 100x100 RGB rendering of alignments.'
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:6de4be5bee6e8bf0e95da3d9b1e170de0d33ecdad163707e1586443f427a90e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# read-alignment-to-rgb-image-encoding

## Summary

A skill for converting quality-controlled SAM/BAM read alignments (e.g., HG002 from GIAB AshkenazimTrio) into 100x100 RGB images whose pixels encode CIGAR operation states (D, M, S, I), so that deletion-variant features are spatially mapped onto image regions for local feature perception in DASV.

## When to use

Use when long-read SAM/BAM alignments must be converted into fixed-size RGB images (100x100) encoding CIGAR deletion features, e.g., as image-modality input for a dual-attention structural-variant-calling model.

## When NOT to use

- The goal is insertion or base substitution variant calling — the paper states it 'currently focuses exclusively on experiments related to deletion variants calling'.
- Input is raw FASTQ/basecalled reads without an existing alignment (SAM/BAM) and CIGAR fields.
- A schema-preserving numeric feature representation is required rather than a rasterized image modality (e.g., for non-vision models).

## Inputs

- Quality-controlled SAM/BAM alignment files (e.g., HG002 from GIAB AshkenazimTrio, filtered for MAPQ >= 20 and FLAG excluding 4 and >256)
- CIGAR strings from the filtered alignments
- Average sequencing depth of the alignment file (for high-frequency region selection)

## Outputs

- 100x100 RGB PNG images, one per genomic window, named per genomic region
- Per-image pixel colors restricted to white (255,255,255), red (255,0,0; mode D), green (0,255,0; mode M), blue (0,0,255; mode S), and black (0,0,0; mode I)

## How to apply

First quality-control the SAM/BAM by dropping reads with alignment quality <20 and FLAG identifiers of 4 and >256. Extract deletion features from the CIGAR field, count deletions per region, merge ranges within the same region, and select high-frequency regions according to the file's average sequencing depth. Tile each selected region with a sliding window of size 100 and stride length 50, then render each window onto a 100x100 pixel canvas where each pixel row represents one read (top-to-bottom, left-to-right) on a white (255,255,255) background with CIGAR modes D, M, S, I colored red (255,0,0), green (0,255,0), blue (0,0,255), and black (0,0,0). Pad images shorter than 100 pixels with whitespace and compress images longer than 100 pixels so every final image is exactly 100x100 pixels, saving the PNG set named per genomic region. This rasterization maps deletion-variant features of genes to corresponding image regions, giving the downstream model local perception of data features.

## Related tools

- **DASV** (Provides the image-conversion pipeline: CIGAR-based deletion extraction, sliding-window tiling (window 100, stride 50), and 100x100 RGB rendering of alignments.) — https://github.com/deconvolution-w/DASV

## Evaluation signals

- Every saved image exists and is a 100x100 RGB PNG.
- All pixel values are confined to the white background and the four CIGAR color values: (255,255,255), (255,0,0), (0,255,0), (0,0,255), (0,0,0).
- Windows tile selected high-frequency regions with window size 100 and stride 50, and images shorter than 100 pixels are whitespace-padded while longer ones are compressed to 100x100.
- Rendered windows are consistent with the image examples shown in the paper's Fig. 2.
- Inputs were pre-filtered: no retained read has alignment quality <20 or FLAG 4 or >256.

## Limitations

- Conversion into images may cause information loss, primarily due to the use of 100-sized sliding windows.
- Only deletion variant calling is covered; insertion and base substitution detection is future work.
- Requires already-aligned SAM/BAM data filtered to MAPQ >= 20 with FLAG 4 and >256 dropped; raw reads are not a valid input.

## Evidence

- [methods] Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [methods] Sequenced fragments with an alignment quality number <20 were removed: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] the sliding window was set to 100 and the stride length was set to 50: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [methods] Images that were <100 pixels were padded with whitespace: "Images that were <100 pixels were padded with whitespace and those that were >100 pixels were compressed so that the final image size was 100*100 pixels"
- [intro] This would map the deletion variants features of genes to the corresponding regions of the image: "Convert gene alignment information into images. This would map the deletion variants features of genes to the corresponding regions of the image"
- [readme] Our method achieved the best results on all datasets.: "Our method achieved the best results on all datasets."

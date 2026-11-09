---
name: sliding-window-high-frequency-region-tiling
description: Use when quality-controlled SAM/BAM alignments (reads with MAPQ <20,
  FLAG 4 or >256 already dropped) must be converted into fixed-size 100x100 images
  that map deletion-variant features onto image regions for downstream local feature
  perception.
when_to_use_negative:
- The analysis goal is insertion or base substitution calling — the article states
  it 'currently focuses exclusively on experiments related to deletion variants calling'
- Input reads have not passed quality control (MAPQ <20 or FLAG 4/>256 reads still
  present), since CIGAR-based deletion framing assumes pre-filtered SAM files
- A direct sequence-level variant callset is required rather than image representations
  for dual-attention model input
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3175
tools:
- name: DASV
  role: Implements the CIGAR-based deletion framing, high-frequency region selection,
    sliding-window tiling (window 100, stride 50), and 100x100 pixel image rendering
    of alignments
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:caaac058fedab890c7d944477c0216feffaab66a440b68b5ab8411d3c345efa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# sliding-window-high-frequency-region-tiling

## Summary

Tiling of high-frequency deletion regions extracted from CIGAR fields of quality-controlled SAM/BAM alignments using a sliding window of size 100 with stride 50, then rendering each window onto a 100x100 pixel canvas for DASV's image-based deletion variant feature perception.

## When to use

Use when quality-controlled SAM/BAM alignments (reads with MAPQ <20, FLAG 4 or >256 already dropped) must be converted into fixed-size 100x100 images that map deletion-variant features onto image regions for downstream local feature perception.

## When NOT to use

- The analysis goal is insertion or base substitution calling — the article states it 'currently focuses exclusively on experiments related to deletion variants calling'
- Input reads have not passed quality control (MAPQ <20 or FLAG 4/>256 reads still present), since CIGAR-based deletion framing assumes pre-filtered SAM files
- A direct sequence-level variant callset is required rather than image representations for dual-attention model input

## Inputs

- Quality-controlled SAM/BAM alignments (e.g., HG002 from GIAB AshkenazimTrio), filtered to drop reads with alignment quality <20 and FLAG identifiers of 4 and >256
- CIGAR fields of the filtered SAM file for deletion feature extraction

## Outputs

- 100x100 RGB PNG images, one per sliding window, named per genomic region, with pixels confined to white background and the four CIGAR colors (red/green/blue/black)

## How to apply

First extract deletion features from the CIGAR field of the filtered SAM file, count deletions per region, merge ranges within the same region, and select high-frequency regions according to the file's average sequencing depth. Then tile each selected region with a sliding window of size 100 and stride length 50. Render each window onto a 100x100 pixel canvas where each pixel row is one read arranged top-to-bottom, left-to-right: white background (255,255,255) with CIGAR modes D, M, S, I colored red (255,0,0), green (0,255,0), blue (0,0,255), and black (0,0,0). Pad windows shorter than 100 pixels with whitespace and compress windows longer than 100 pixels so every final image is exactly 100x100 pixels, then save the PNGs named per genomic region.

## Related tools

- **DASV** (Implements the CIGAR-based deletion framing, high-frequency region selection, sliding-window tiling (window 100, stride 50), and 100x100 pixel image rendering of alignments) — https://github.com/deconvolution-w/DASV

## Evaluation signals

- Each saved image exists and is exactly 100x100 pixels (padding for windows <100 px, compression for windows >100 px)
- Every image is an RGB PNG whose pixel values are confined to white (255,255,255) and the four CIGAR colors: red (255,0,0), green (0,255,0), blue (0,0,255), black (0,0,0)
- Selected regions are high-frequency deletion regions consistent with the file's average sequencing depth threshold
- Windows tile each selected region at stride 50 with size 100, so consecutive windows overlap by 50 pixels
- Rendered images visually match the image examples in the paper's Fig. 2

## Limitations

- The 100-sized sliding window can cause information loss during conversion into images; the article notes this loss is not quantified
- Only deletion variants (CIGAR D-mode regions) are targeted; insertion and base substitution detection is explicitly left as future work
- Windows longer than 100 pixels are compressed and shorter windows are padded, introducing distortion relative to the original alignment layout

## Evidence

- [methods] the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [methods] Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file: "Deletion features were obtained based on the CIGAR field of the quality-controlled SAM file"
- [methods] Images that were <100 pixels were padded with whitespace and those that were >100 pixels were compressed so that the final image size was 100*100 pixels: "Images that were <100 pixels were padded with whitespace and those that were >100 pixels were compressed so that the final image size was 100*100 pixels"
- [abstract] DASV converts gene alignment information into images: "DASV converts gene alignment information into images"

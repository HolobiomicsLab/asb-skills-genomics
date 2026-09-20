---
name: structural-variant-image-representation
description: Use when third-generation sequencing alignments (SAM/BAM, e.g. HG002
  from GIAB AshkenazimTrio) must be converted into fixed-size images so that CIGAR-encoded
  deletion features can be fed to a multi-modal (image+sequence) deep learning SV
  caller.
when_to_use_negative:
- The goal is insertion or base substitution variant calling — the paper's workflow
  covers exclusively deletion variants calling
- Input is raw FASTQ or unassembled reads without CIGAR-bearing alignments, since
  deletion features are extracted from the CIGAR field
- Short-read NGS data with the read-length and base-pair mismatch limitations the
  authors cite as motivation for using third-generation long reads
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3518
tools:
- name: DASV
  role: Pipeline whose image-generation stage performs the CIGAR-to-pixel rendering
    and whose dual-attention model consumes the resulting images together with sequence
    encodings
  repo: https://github.com/deconvolution-w/DASV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:6e361040d919b0ccbe847665728b453da4b3275be7a4f79e5279f41841eaa121
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# structural-variant-image-representation

## Summary

Renders quality-controlled SAM/BAM read alignments as 100x100 pixel tri-channel images where CIGAR operation modes (D, M, S, I) are encoded as distinct colors, so deletion-variant features are mapped onto spatially corresponding image regions for local feature perception by DASV's dual-attention model.

## When to use

Use when third-generation sequencing alignments (SAM/BAM, e.g. HG002 from GIAB AshkenazimTrio) must be converted into fixed-size images so that CIGAR-encoded deletion features can be fed to a multi-modal (image+sequence) deep learning SV caller.

## When NOT to use

- The goal is insertion or base substitution variant calling — the paper's workflow covers exclusively deletion variants calling
- Input is raw FASTQ or unassembled reads without CIGAR-bearing alignments, since deletion features are extracted from the CIGAR field
- Short-read NGS data with the read-length and base-pair mismatch limitations the authors cite as motivation for using third-generation long reads

## Inputs

- Quality-controlled SAM/BAM alignment file of third-generation sequencing reads (e.g. HG002 from GIAB AshkenazimTrio, reads with MAPQ <20 and FLAG 4 or >256 removed)
- CIGAR strings from the filtered SAM file
- Average sequencing depth of the alignment file (used to select high-frequency deletion regions)

## Outputs

- A set of 100x100 RGB PNG images, one per sliding-window genomic region, named per genomic region
- Images whose pixels are confined to the white background (255,255,255) and four CIGAR colors: red (255,0,0) for D, green (0,255,0) for M, blue (0,0,255) for S, black (0,0,0) for I

## How to apply

Start from SAM/BAM alignments quality-controlled by dropping reads with alignment quality <20 and FLAG identifiers of 4 and >256. Extract deletion features from the CIGAR field, count deletions per region, merge ranges within the same region, and select high-frequency regions according to the file's average sequencing depth. Tile each selected region with a sliding window of size 100 and stride length 50, then render each window onto a 100x100 pixel canvas where each pixel row is one read arranged top-to-bottom and left-to-right on a white background (RGB 255,255,255), coloring CIGAR modes D, M, S, I as red (255,0,0), green (0,255,0), blue (0,0,255), and black (0,0,0) respectively. Pad windows shorter than 100 pixels with whitespace and compress longer ones so every image is exactly 100x100 pixels; save the PNGs named per genomic region. This image representation gives the downstream dual-attention network local perception of deletion-variant features alongside the zero-padded (to length 100) sequence encoding.

## Related tools

- **DASV** (Pipeline whose image-generation stage performs the CIGAR-to-pixel rendering and whose dual-attention model consumes the resulting images together with sequence encodings) — https://github.com/deconvolution-w/DASV

## Examples

```
git clone https://github.com/deconvolution-w/DASV  # then run DASV's image-generation step on the filtered SAM to emit 100x100 PNGs per region
```

## Evaluation signals

- Each saved image exists and is exactly a 100x100 RGB PNG
- All pixel values are confined to the white background (255,255,255) and the four CIGAR colors: red, green, blue, black
- Window tiling used sliding window size 100 and stride length 50 over each selected high-frequency region
- Rendered images visually match the image examples shown in the paper's Fig. 2
- Upstream filtering removed reads with alignment quality <20 and FLAG identifiers of 4 and >256 before extraction

## Limitations

- The 100-sized sliding window can cause information loss during conversion into images, which the authors flag as unquantified
- Only deletion variant calling is covered; insertion and base substitution detection is left as future work
- Images longer than 100 pixels are compressed, potentially distorting per-read pixel resolution

## Evidence

- [other] DASV's conversion step maps the deletion-variant features of genes from the alignment information onto the corresponding regions of an image, which is beneficial for the local perception of data features.: [evidence span withheld — anchor status: not_found]
- [methods] the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [methods] Images that were <100 pixels were padded with whitespace and those that were >100 pixels were compressed so that the final image size was 100*100 pixels: "Images that were <100 pixels were padded with whitespace and those that were >100 pixels were compressed so that the final image size was 100*100 pixels"
- [abstract] The source code is available at https://github.com/deconvolution-w/DASV.: "The source code is available at https://github.com/deconvolution-w/DASV."

---
name: spatial-attention-map-normalization
description: Use when building a dual-attention image-sequence fusion model for deletion
  SV calling from third-generation sequencing alignments, specifically when generating
  the spatial attention map over CIGAR-D marked regions of 100x100 pixel deletion
  images.
when_to_use_negative:
- The task requires insertion or base substitution detection — the article states
  the study 'currently focuses exclusively on experiments related to deletion variants
  calling'
- Input is short-read NGS data without long-read alignment (BAM/SAM from third-generation
  sequencing) — DASV leverages long-read alignment features from data such as the
  GIAB Ashkenazim Trio
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DASV
  role: 'Implements the spatial attention method: softmax normalization of attention
    maps over CIGAR-D marked deletion-region images prior to sequence-image fusion'
  repo: https://github.com/deconvolution-w/DASV
- name: ResNet
  role: Multi-scale backbone with residual connections that learns features from the
    attention-normalized fused image-sequence representation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:7d79353a59db3ea49e51e282a0e428953a301b8539fe5f76ce3204df06d60405
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - ResNet
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spatial-attention-map-normalization

## Summary

A spatial-attention branch within DASV that constrains attention over 100x100-pixel deletion-region images: binary CIGAR-D attention images are convolved with the original images, multiplied, and the resulting attention map is normalized with softmax so values lie in 0-1 before feature fusion with sequence encodings.

## When to use

Use when building a dual-attention image-sequence fusion model for deletion SV calling from third-generation sequencing alignments, specifically when generating the spatial attention map over CIGAR-D marked regions of 100x100 pixel deletion images.

## When NOT to use

- The task requires insertion or base substitution detection — the article states the study 'currently focuses exclusively on experiments related to deletion variants calling'
- Input is short-read NGS data without long-read alignment (BAM/SAM from third-generation sequencing) — DASV leverages long-read alignment features from data such as the GIAB Ashkenazim Trio

## Inputs

- 100x100-pixel high-frequency deletion-region images (sliding window 100, stride 50; CIGAR D/M/S/I mapped to red/green/blue/black)
- Binary spatial attention images marking CIGAR D regions (RGB 255,255,255) versus background (0,0,0)
- Encoded base sequences (A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100)
- Quality-controlled SAM/BAM alignment files (MAPQ >= 20; FLAG 4 and >256 removed)

## Outputs

- Softmax-normalized spatial attention map with values constrained to 0-1
- Spatially attended image feature map for downstream channel-attention fusion and ResNet classification
- Per-region deletion-variant predictions from the trained multi-scale network

## How to apply

From the quality-controlled SAM/BAM, mark CIGAR D regions as white pixels (RGB 255,255,255) against black background (0,0,0) to build binary attention images, then convolve them with the original high-frequency deletion-region images and multiply element-wise. Normalize the resulting attention map with a softmax function so attention values are constrained to the 0-1 range, which lets the model weight deletion-relevant image regions for local perception. The spatial-attention feature map is then fed to the channel-attention fusion branch, where GAP and GMP downsampling derive global receptive-field weight vectors that are recalculated by convolutional layers, summed, sigmoid-normalized, and multiplied per-channel into the sequence features (bases A/C/T/G encoded as 2/3/4/5, deletions as 9, zero-padded to length 100). The fused features are learned by a multi-scale ResNet backbone with residual connections trained end-to-end on labeled deletion/non-deletion image-sequence pairs.

## Related tools

- **DASV** (Implements the spatial attention method: softmax normalization of attention maps over CIGAR-D marked deletion-region images prior to sequence-image fusion) — https://github.com/deconvolution-w/DASV
- **ResNet** (Multi-scale backbone with residual connections that learns features from the attention-normalized fused image-sequence representation)

## Examples

```
git clone https://github.com/deconvolution-w/DASV  # run spatial attention + fusion training on 100x100 CIGAR-D images with HG002 validation
```

## Evaluation signals

- Check the spatial attention map is softmax-normalized with all values in [0,1] as the article requires
- Verify attention images mark CIGAR D regions as RGB 255,255,255 against 0,0,0 background before convolution and multiplication with the original images
- Confirm the trained model reproduces the reported pattern that feature fusion improves F1 score over sequence-only features (Supplementary Table 3)
- Validate precision/recall/F1 on HG002 (NIST Reference Material 8392, 5464 deletion callings) are in [0,1] and internally consistent against the benchmark comparison (Supplementary Table 4)
- Confirm pipeline outputs for the channel-attention branch use GAP/GMP downsampling with sigmoid-normalized per-channel weights multiplied into encoded sequence features

## Limitations

- The 100-sized sliding window may cause information loss during conversion into images, and this loss is not quantified in the article
- Only deletion variant calling is covered; insertion and base substitution detection is left as future work
- Attention images shorter than 100 pixels are padded with whitespace and longer ones compressed to 100x100, which may distort deletion-region geometry

## Evidence

- [methods] Softmax-normalized spatial attention map constrained to 0-1: "the softmax function is used to normalize the attention map and limit the values of the attention map to 0-1"
- [methods] GAP/GMP channel attention fusion branch: "It is downsampled using Global Average Pooling(GAP) and Global Max Pooling(GMP) to obtain its global receptive field"
- [abstract] Dual attention integrates images with sequencing data: "integrates them with genomic sequencing data through a dual attention mechanism"
- [methods] Sliding window and canvas parameters for image generation: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [supplementary] Feature fusion improves F1 over sequence-only features: [evidence span withheld — anchor status: not_found]

---
name: multi-modal-feature-attention-weighting
description: Use when calling deletion structural variants from third-generation sequencing
  alignments and you have both generated deletion-region images and CIGAR-framed encoded
  base sequences that must be fused for classification.
when_to_use_negative:
- The task requires insertion or base substitution calling — the article states 'This
  study currently focuses exclusively on experiments related to deletion variants
  calling'.
- Only raw read alignments without the upstream pre-processing outputs (100x100 images
  and zero-padded 100-length encoded sequences) are available.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DASV
  role: Implements the dual attention fusion of image and sequence modalities for
    deletion structural variation calling
  repo: https://github.com/deconvolution-w/DASV
- name: ResNet
  role: Multi-scale backbone with residual connections that learns the fused dual-attention
    features for deletion region identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:9a61b25fd331e62a429bc705c8229139b9aab533ea1a9dea915c581dd5bb9e56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - ResNet
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# multi-modal-feature-attention-weighting

## Summary

Fuses 100x100-pixel deletion-region alignment images with encoded base sequences (A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to 100) via a dual attention mechanism — spatial attention over images and GAP/GMP channel attention over sequence features — feeding a ResNet backbone for deletion variant calling (DASV).

## When to use

Use when calling deletion structural variants from third-generation sequencing alignments and you have both generated deletion-region images and CIGAR-framed encoded base sequences that must be fused for classification.

## When NOT to use

- The task requires insertion or base substitution calling — the article states 'This study currently focuses exclusively on experiments related to deletion variants calling'.
- Only raw read alignments without the upstream pre-processing outputs (100x100 images and zero-padded 100-length encoded sequences) are available.

## Inputs

- 100x100-pixel high-frequency deletion-region images (sliding window 100, stride 50; CIGAR D/M/S/I mapped to red/green/blue/black)
- Encoded hard-coded base sequences (A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100) from the pre-processed BAM file
- Binary spatial attention images marking CIGAR D regions (RGB 255,255,255) versus background (0,0,0)
- Labeled deletion/non-deletion image-sequence training pairs

## Outputs

- Fused dual-attention feature maps learned by the ResNet backbone
- Per-region deletion-variant predictions for validation data (e.g. HG002, NIST Reference Material 8392 with 5464 deletion callings)

## How to apply

First build the spatial attention branch: create binary attention images marking CIGAR D regions (RGB 255,255,255) versus background (0,0,0), convolve with the original images, multiply, and normalize the attention map with softmax to constrain values to 0-1. Then build the channel-attention branch: downsample the spatial-attention feature map using Global Average Pooling (GAP) and Global Max Pooling (GMP) to obtain a global receptive field, recalculate weight vectors with convolutional layers, sum and normalize with sigmoid, and multiply the per-channel weights into the encoded sequence features. Assemble the fused features into a multi-scale ResNet backbone with residual connections and train end-to-end on labeled deletion/non-deletion image-sequence pairs. The rationale is that images capture local deletion-region perception while pooled global sequence features supply the receptive field the image branch lacks.

## Related tools

- **DASV** (Implements the dual attention fusion of image and sequence modalities for deletion structural variation calling) — https://github.com/deconvolution-w/DASV
- **ResNet** (Multi-scale backbone with residual connections that learns the fused dual-attention features for deletion region identification)

## Evaluation signals

- The spatial attention map values are constrained to 0-1 after softmax normalization.
- Fused dual-attention features improve F1 score over sequence-only features (as reported in Supplementary Table 3).
- On HG002 validation data (NIST Reference Material 8392, 5464 deletion callings), precision, recall and F1 in [0,1] are internally consistent when recomputed from the benchmark comparison (Supplementary Table 4).
- DASV outperforms cuteSV, SVIM, Sniffles and PBSV in F1 score on the evaluated datasets.
- Sequence tensors shorter than the length-100 threshold are zero-padded to exactly 100 before fusion.

## Limitations

- Only deletion variant calling is covered; insertion and base substitution detection remain future work.
- Information loss during conversion into images due to the 100-sized sliding window is not quantified.
- The method requires high coverage depth to show its notable advantages.

## Evidence

- [abstract] integrates them with genomic sequencing data through a dual attention mechanism: "integrates them with genomic sequencing data through a dual attention mechanism"
- [methods] softmax normalization to 0-1: "the softmax function is used to normalize the attention map and limit the values of the attention map to 0-1"
- [methods] GAP/GMP downsampling: "It is downsampled using Global Average Pooling(GAP) and Global Max Pooling(GMP) to obtain its global receptive field"
- [methods] sequence zero-padding to 100: "data whose length is shorter than the specified threshold (100) will be padded with zeros(0) up to 100"
- [readme] README scope claim: "Our method achieved the best results on all datasets. And we have submitted the manuscript to [briefings in bioinformatics](https://academic.oup.com/bib/)."

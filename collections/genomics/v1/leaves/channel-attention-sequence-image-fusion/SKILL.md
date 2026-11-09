---
name: channel-attention-sequence-image-fusion
description: Use when quality-controlled SAM/BAM alignments from nanopore-style reads
  have already been converted to 100x100 pixel deletion-region images and zero-padded
  encoded base sequences, and deletion/non-deletion classification per region is the
  goal.
when_to_use_negative:
- The task requires insertion or base substitution detection — the article states
  'This study currently focuses exclusively on experiments related to deletion variants
  calling'
- Input is raw, unaligned read data rather than quality-controlled SAM/BAM files (alignment
  quality <20 and FLAG 4/>256 fragments must already be removed)
- Only short-read Illumina data are available — the method is motivated by 'the long
  reads advantages of third-generation sequencing data'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3474
tools:
- name: DASV
  role: Implements the dual-attention (spatial softmax + GAP/GMP sigmoid channel attention)
    fusion of deletion-region images with encoded sequences for deletion calling
  repo: https://github.com/deconvolution-w/DASV
- name: ResNet
  role: Multi-scale backbone with residual connections that learns the fused features
    for deletion region identification
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:76787c6cd56da4242eab8794a2347848069822145d43ca5a7fd5cf3cff955710
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - ResNet
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# channel-attention-sequence-image-fusion

## Summary

A dual-attention feature-fusion step from DASV that combines spatially attended 100x100 deletion-region images with GAP/GMP-downsampled genomic sequence features, feeding a ResNet multi-scale backbone for deletion structural variant calling in third-generation sequencing data.

## When to use

Use when quality-controlled SAM/BAM alignments from nanopore-style reads have already been converted to 100x100 pixel deletion-region images and zero-padded encoded base sequences, and deletion/non-deletion classification per region is the goal.

## When NOT to use

- The task requires insertion or base substitution detection — the article states 'This study currently focuses exclusively on experiments related to deletion variants calling'
- Input is raw, unaligned read data rather than quality-controlled SAM/BAM files (alignment quality <20 and FLAG 4/>256 fragments must already be removed)
- Only short-read Illumina data are available — the method is motivated by 'the long reads advantages of third-generation sequencing data'

## Inputs

- 100x100-pixel high-frequency deletion-region images (sliding window 100, stride 50; CIGAR D/M/S/I mapped to red/green/blue/black)
- Encoded base sequences (A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100) from the pre-processed BAM/SAM files
- Labeled deletion/non-deletion image-sequence training pairs

## Outputs

- Fused image-sequence feature maps from the dual-attention module
- Per-region deletion-variant predictions (probabilities/labels)
- Precision, recall and F1 score when compared against a benchmark set

## How to apply

Build the spatial attention branch by generating binary attention images marking CIGAR D regions (RGB 255,255,255 vs 0,0,0 background), convolving with the original images, multiplying, and normalizing the attention map with softmax to constrain values to 0-1. Then apply channel attention: downsample the spatial-attention feature map via Global Average Pooling (GAP) and Global Max Pooling (GMP) to obtain a global receptive field, recalculate weight vectors with convolutional layers, sum and normalize with sigmoid, and multiply the per-channel weights into the encoded sequence features. Feed the fused representation into a multi-scale ResNet backbone with residual connections, trained end-to-end on labeled deletion/non-deletion image-sequence pairs. The rationale is that feature fusion improves F1 score over sequence-only features (Supplementary Table 3).

## Related tools

- **DASV** (Implements the dual-attention (spatial softmax + GAP/GMP sigmoid channel attention) fusion of deletion-region images with encoded sequences for deletion calling) — https://github.com/deconvolution-w/DASV
- **ResNet** (Multi-scale backbone with residual connections that learns the fused features for deletion region identification)

## Evaluation signals

- The spatial attention map after softmax normalization must contain only values in the 0-1 range
- Feature fusion should improve F1 score relative to sequence-only features, reproducing Supplementary Table 3
- Precision, recall and F1 on the HG002 benchmark (NIST Reference Material 8392, 5464 deletion callings) must each lie in [0,1] and be internally consistent when recomputed from the benchmark comparison (Supplementary Table 4)
- Encoded sequences shorter than the threshold of 100 must be zero-padded to exactly length 100 before fusion
- All images must be exactly 100x100 pixels after whitespace padding (<100 px) or compression (>100 px)

## Limitations

- The method only covers deletion variant calling; insertion and base substitution detection is explicitly future work
- Converting alignments into images can lose information due to the use of 100-sized sliding windows; this loss is not quantified
- Information loss in image conversion is noted as a concern ('mitigating potential information loss during conversion into images (primarily due to the use of 100-sized sliding windows)')

## Evidence

- [methods] spatial softmax normalization constrains attention: "the softmax function is used to normalize the attention map and limit the values of the attention map to 0-1"
- [methods] GAP/GMP channel downsampling: "It is downsampled using Global Average Pooling(GAP) and Global Max Pooling(GMP) to obtain its global receptive field"
- [abstract] dual attention integrates images with sequence data: "integrates them with genomic sequencing data through a dual attention mechanism"
- [intro] ResNet backbone rationale: [evidence span withheld — anchor status: not_found]
- [readme] README confirms DASV repository and best results claim: "Our method achieved the best results on all datasets. And we have submitted the manuscript to [briefings in bioinformatics]"

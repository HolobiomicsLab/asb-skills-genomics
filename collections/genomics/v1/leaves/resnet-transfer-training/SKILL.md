---
name: resnet-transfer-training
description: Use when quality-controlled SAM/BAM alignments have already been encoded
  into 100x100-pixel deletion-region images and zero-padded base sequences (length
  100), and a fused deep model must be trained to call deletion variants.
when_to_use_negative:
- The task is insertion or base substitution calling — the article states it 'currently
  focuses exclusively on experiments related to deletion variants calling'
- Input is raw sequencing reads without a quality-controlled alignment (SAM/BAM with
  alignment quality >= 20); the upstream QC and image-generation steps must run first
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: DASV
  role: Dual Attention Structural Variation method that converts gene alignment information
    into images, fuses them with genomic sequencing data via dual attention, and trains
    the multi-scale network for deletion calling
  repo: https://github.com/deconvolution-w/DASV
- name: ResNet
  role: Backbone network of the multi-scale architecture; residual connections enhance
    the model's expression ability for learning fused features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:9f85fa631c07a289b6829b3ce5586f2d06fb5d1d0a76f767fc036060df6bc571
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
n_claims: 10
---

# resnet-transfer-training

## Summary

Train the DASV multi-scale network, whose backbone is a ResNet with residual connections, end-to-end on fused image-sequence features of deletion regions to predict deletion structural variants from third-generation sequencing alignment data.

## When to use

Use when quality-controlled SAM/BAM alignments have already been encoded into 100x100-pixel deletion-region images and zero-padded base sequences (length 100), and a fused deep model must be trained to call deletion variants.

## When NOT to use

- The task is insertion or base substitution calling — the article states it 'currently focuses exclusively on experiments related to deletion variants calling'
- Input is raw sequencing reads without a quality-controlled alignment (SAM/BAM with alignment quality >= 20); the upstream QC and image-generation steps must run first

## Inputs

- 100x100-pixel high-frequency deletion-region images generated from CIGAR fields (sliding window 100, stride 50)
- Encoded base sequences of the sample genome, bases A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100
- Binary spatial-attention images marking CIGAR D regions (RGB 255,255,255) versus background (0,0,0)
- Labeled deletion/non-deletion image-sequence training pairs

## Outputs

- Trained multi-scale ResNet model with dual-attention fusion weights
- Per-region deletion-variant predictions for validation data (e.g. HG002)
- Precision, recall and F1 score versus the NIST benchmark set

## How to apply

Load the 100x100-pixel high-frequency deletion-region images (sliding window 100, stride 50; CIGAR D/M/S/I encoded as red/green/blue/black) together with the base sequences encoded A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100. Build the spatial-attention branch: convolve binary attention images marking CIGAR D regions (RGB 255,255,255 vs background 0,0,0) with the original images, multiply, and normalize the attention map with softmax to constrain values to 0-1. Build the channel-attention branch: downsample the spatial-attention feature map via Global Average Pooling (GAP) and Global Max Pooling (GMP), recalculate weight vectors with convolutional layers, sum and normalize with sigmoid, then multiply per-channel weights into the encoded sequence features. Train the assembled multi-scale network with a ResNet backbone and residual connections end-to-end on labeled deletion/non-deletion image-sequence pairs, then run inference on validation data (HG002, NIST Reference Material 8392 with 5464 deletion callings) and compare against the benchmark to report precision, recall and F1 score.

## Related tools

- **DASV** (Dual Attention Structural Variation method that converts gene alignment information into images, fuses them with genomic sequencing data via dual attention, and trains the multi-scale network for deletion calling) — https://github.com/deconvolution-w/DASV
- **ResNet** (Backbone network of the multi-scale architecture; residual connections enhance the model's expression ability for learning fused features)

## Evaluation signals

- Fused features must improve F1 score over sequence-only features, reproducing the pattern reported in Supplementary Table 3
- Reported precision/recall/F1 values must lie in [0,1] and be internally consistent against the benchmark comparison (Supplementary Table 4)
- Predictions on HG002 validation data (NIST Reference Material 8392, 5464 deletion callings) should balance precision and recall rather than trade one for the other
- Softmax-normalized spatial attention map values must fall in [0,1]
- Model should outperform cuteSV, SVIM, Sniffles and PBSV baselines, particularly on high-coverage data in F1 score

## Limitations

- The study covers only deletion variant calling; insertion and base substitution detection is future work
- Potential information loss during conversion into images, primarily due to the use of the 100-sized sliding window, is not quantified
- No versions are stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV), complicating exact reproduction of benchmark comparisons

## Evidence

- [methods] Softmax-normalized spatial attention values constrained to 0-1: "the softmax function is used to normalize the attention map and limit the values of the attention map to 0-1"
- [methods] GAP/GMP channel-attention downsampling of the spatial-attention feature map: "It is downsampled using Global Average Pooling(GAP) and Global Max Pooling(GMP) to obtain its global receptive field"
- [intro] ResNet backbone with residual connections: [evidence span withheld — anchor status: not_found]
- [readme] Overall performance claim of the trained model: "Our method achieved the best results on all datasets. And we have submitted the manuscript to [briefings in bioinformatics](https://academic.oup.com/bib/)."
- [discussion] Scope limited to deletion variants: "This study currently focuses exclusively on experiments related to deletion variants calling"

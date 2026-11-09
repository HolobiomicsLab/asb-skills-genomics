---
name: cnn-structural-variant-classification
description: Use when calling deletion SVs from long-read (third-generation) BAM/SAM
  alignments where alignments can be converted to CIGAR-derived images and encoded
  sequences, and a deep model must fuse both modalities to output per-region deletion
  predictions.
when_to_use_negative:
- The goal is insertion or base substitution calling — the article states 'This study
  currently focuses exclusively on experiments related to deletion variants calling'
- Input is short-read NGS (Illumina) data rather than third-generation long-read alignments,
  where the method's motivation and evaluation do not apply
- Only sequence data is available without aligned BAM/SAM files, since image generation
  and deletion framing depend on CIGAR fields of quality-controlled alignments
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: DASV
  role: The dual-attention feature-fusion method implementing image generation, sequence
    encoding, and ResNet-based deletion classification
  repo: https://github.com/deconvolution-w/DASV
- name: ResNet
  role: Multi-scale backbone network with residual connections that learns the fused
    image-sequence features for deletion region identification
  repo: https://github.com/deconvolution-w/DASV
- name: cuteSV
  role: Baseline deletion-calling tool used for precision/recall/F1 comparison
- name: SVIM
  role: Baseline deletion-calling tool used for precision/recall/F1 comparison
- name: Sniffles
  role: Baseline deletion-calling tool used for precision/recall/F1 comparison
- name: PBSV
  role: Baseline deletion-calling tool used for precision/recall/F1 comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae269
    title: Deletion variants calling in third-generation sequencing data based on
      a dual-attention mechanism
  source_content_hash: sha256:a5a600d0f30afe41443388ad0d354e24163c54f252bc81b46315ee1e3f12aab2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/deconvolution-w/DASV
  tools:
  - DASV
  - PBSV
  - ResNet
  - SVIM
  - Sniffles
  - cuteSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# cnn-structural-variant-classification

## Summary

Classify deletion structural variants from third-generation sequencing alignments by fusing generated deletion-region images (100x100 px, CIGAR-coded RGB) with hard-coded base sequences via a dual-attention mechanism (spatial softmax attention + GAP/GMP channel attention) on a multi-scale ResNet backbone (DASV).

## When to use

Use when calling deletion SVs from long-read (third-generation) BAM/SAM alignments where alignments can be converted to CIGAR-derived images and encoded sequences, and a deep model must fuse both modalities to output per-region deletion predictions.

## When NOT to use

- The goal is insertion or base substitution calling — the article states 'This study currently focuses exclusively on experiments related to deletion variants calling'
- Input is short-read NGS (Illumina) data rather than third-generation long-read alignments, where the method's motivation and evaluation do not apply
- Only sequence data is available without aligned BAM/SAM files, since image generation and deletion framing depend on CIGAR fields of quality-controlled alignments

## Inputs

- Quality-controlled SAM/BAM alignment files from third-generation sequencing (e.g., GIAB Ashkenazim Trio HG002/HG003/HG004)
- CIGAR fields of aligned reads framing deletion variation ranges
- Generated 100x100-pixel high-frequency deletion-region images (CIGAR D/M/S/I as red/green/blue/black)
- Hard-coded base sequences (A/C/T/G as 2/3/4/5, deletions as 9, zero-padded to length 100)
- Labeled deletion/non-deletion image-sequence training pairs
- Benchmark variant set (NIST Reference Material 8392, 5464 deletion callings)

## Outputs

- Per-region deletion-variant predictions for validation samples (e.g., HG002)
- Trained dual-attention ResNet model with fused spatial- and channel-attention features
- Precision, recall and F1 score per dataset compared against cuteSV, SVIM, Sniffles and PBSV

## How to apply

First pre-process quality-controlled SAM files (remove fragments with MAPQ <20 and FLAG 4 or >256), frame deletion regions from CIGAR D fields (sliding window 100, stride 50), and generate 100x100-pixel images with CIGAR D/M/S/I mapped to red/green/blue/black (padding/compressing to exactly 100x100 px). Encode the base sequence over the framed region with A/C/T/G as 2/3/4/5, deletions as 9, zero-padding sequences shorter than 100. Build the dual attention: a spatial branch multiplies the images by a binary attention map marking CIGAR D regions (RGB 255,255,255) and normalizes the attention map with softmax to 0-1; a channel branch downsamples the spatial-attention feature map via Global Average Pooling and Global Max Pooling, recomputes weight vectors with convolutions, sums and normalizes with sigmoid, then multiplies per-channel weights into the sequence features. Train the fused representation end-to-end on a ResNet backbone with residual connections on labeled deletion/non-deletion pairs, then infer per-region deletion calls and evaluate precision, recall and F1 against a benchmark set (e.g., NIST Reference Material 8392 with 5464 deletion callings).

## Related tools

- **DASV** (The dual-attention feature-fusion method implementing image generation, sequence encoding, and ResNet-based deletion classification) — https://github.com/deconvolution-w/DASV
- **ResNet** (Multi-scale backbone network with residual connections that learns the fused image-sequence features for deletion region identification) — https://github.com/deconvolution-w/DASV
- **cuteSV** (Baseline deletion-calling tool used for precision/recall/F1 comparison)
- **SVIM** (Baseline deletion-calling tool used for precision/recall/F1 comparison)
- **Sniffles** (Baseline deletion-calling tool used for precision/recall/F1 comparison)
- **PBSV** (Baseline deletion-calling tool used for precision/recall/F1 comparison)

## Examples

```
git clone https://github.com/deconvolution-w/DASV && python train_dasv.py --images del_regions_100x100 --sequences encoded_seqs_len100 --labels deletion_pairs
```

## Evaluation signals

- Spatial attention map values lie in [0,1] after softmax normalization
- All images are exactly 100x100 pixels (shorter padded with whitespace, longer compressed) and sequences zero-padded to length 100
- Predictions on HG002 (NIST Reference Material 8392, 5464 deletion callings) yield precision/recall/F1 values in [0,1] internally consistent against the benchmark comparison
- Feature fusion improves F1 over sequence-only features, reproducing the Supplementary Table 3 pattern
- DASV balances precision and recall, enhancing F1 across datasets relative to cuteSV, SVIM, Sniffles and PBSV

## Limitations

- Only deletion calling is covered; insertion and base substitution detection is future work
- Potential information loss from the 100-sized sliding window during conversion into images is not quantified
- Method aims at high-coverage data; advantages are reported 'for data with high coverage depth'
- No versions stated for the compared tools (cuteSV, SVIM, Sniffles, PBSV)

## Evidence

- [methods] spatial attention map over the generated images is normalized with softmax to values 0-1: [evidence span withheld — anchor status: not_found]
- [methods] downsampled using Global Average Pooling (GAP) and Global Max Pooling (GMP): [evidence span withheld — anchor status: not_found]
- [methods] Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256: "Sequenced fragments with an alignment quality number <20 were removed, as were those with a FLAG identifier of 4 and >256"
- [methods] the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels: "the sliding window was set to 100 and the stride length was set to 50. Canvas length and width were set to 100 pixels"
- [abstract] the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets: "the results demonstrate that DASV consistently achieves a balance between precision and recall, enhancing the F1 score across various datasets"

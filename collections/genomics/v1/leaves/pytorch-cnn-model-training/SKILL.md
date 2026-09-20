---
name: pytorch-cnn-model-training
description: Use when you have 2D image renderings of N x N gene-by-gene matrices
  (e.g., CCA matrices) with binary group labels (e.g., G2 vs G4 survival) and need
  a CNN classifier to test preliminary discrimination.
when_to_use_negative:
- Your input is already a numeric feature table rather than rendered N x N CCA matrix
  images — DenseNet-121 on images is specific to the image-based formulation
- Your survival outcome is continuous or multi-group rather than the binary shortened
  (G4) versus longer (G2) comparison
- You need de novo mutational signature extraction itself (use RNMF denovoNMF instead);
  this skill covers only the downstream deep-learning classification of CCA images
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: PyTorch
  role: Deep learning framework in which the DenseNet-121 classifier is instantiated
    and trained on CCA images
- name: DenseNet
  role: Publicly available CNN architecture used for the G2-vs-G4 classification of
    500x500 CCA matrix images
- name: RNMF
  role: R package that generates the gene CCA matrices (via cumulativeCA / genePerMutSigs)
    that are rendered as the classifier's input images
  repo: https://github.com/zhenzhang-li/RNMF
- name: hclust
  role: R hierarchical clustering used to order intersecting genes from top to bottom
    in the CCA matrix before image rendering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:1e79c5f4632284126d0a0c84bd32e93554b7ef5b90d7bba2ae7dece1218a11c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - DenseNet
  - PyTorch
  - R
  - RNMF
  - hclust
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# pytorch-cnn-model-training

## Summary

Train the publicly available DenseNet-121 convolutional architecture in PyTorch on gene CCA matrix images (resized to 500x500 pixels) to preliminarily discriminate shortened (G4) versus longer (G2) survival outcome in ESCC, as described in the RNMF framework paper.

## When to use

Use when you have 2D image renderings of N x N gene-by-gene matrices (e.g., CCA matrices) with binary group labels (e.g., G2 vs G4 survival) and need a CNN classifier to test preliminary discrimination.

## When NOT to use

- Your input is already a numeric feature table rather than rendered N x N CCA matrix images — DenseNet-121 on images is specific to the image-based formulation
- Your survival outcome is continuous or multi-group rather than the binary shortened (G4) versus longer (G2) comparison
- You need de novo mutational signature extraction itself (use RNMF denovoNMF instead); this skill covers only the downstream deep-learning classification of CCA images

## Inputs

- N x N gene CCA matrix images (N = number of intersecting genes) rendered from RNMF genePerMutSigs output, uniformly resized to 500x500 pixels
- Survival outcome labels for each sample image (G2 = longer survival, G4 = shortened survival)
- Per-signature sub-feature image sets for New, SBS3*, and SBS17b*
- Intersected gene list (non-silent mutated genes ∩ 717-gene list) used for matrix construction

## Outputs

- Trained DenseNet-121 classifier (PyTorch) per signature sub-feature
- Classification accuracies for the G2 vs G4 comparison (expected: SBS17b* ~77.500%, SBS3* ~72.500%, New ~71.429%)
- Preliminary discrimination of shortened versus longer survival outcome

## How to apply

First ensure the inputs are uniformly resized CCA matrix images (500x500 pixels) for each signature sub-feature (New, SBS3*, SBS17b*), where N equals the number of genes from intersecting the non-silent mutated gene list with the 717-gene list, and genes are ordered top-to-bottom via R's hclust hierarchical clustering. Instantiate DenseNet-121 from PyTorch (publicly available architecture) and train it on images labeled by the G2 versus G4 survival groups, adjusting the output head for the two-class outcome. Evaluate the classifier per signature sub-feature and compare against the reported accuracies (SBS17b* 77.500%, SBS3* 72.500%, New 71.429%) to judge preliminary separation. Accept the run when the trained model and images are produced and the G4-vs-G2 discrimination is consistent with the finding that CCA image data of these signatures help preliminary evaluation of shortened survival.

## Related tools

- **PyTorch** (Deep learning framework in which the DenseNet-121 classifier is instantiated and trained on CCA images)
- **DenseNet** (Publicly available CNN architecture used for the G2-vs-G4 classification of 500x500 CCA matrix images)
- **RNMF** (R package that generates the gene CCA matrices (via cumulativeCA / genePerMutSigs) that are rendered as the classifier's input images) — https://github.com/zhenzhang-li/RNMF
- **hclust** (R hierarchical clustering used to order intersecting genes from top to bottom in the CCA matrix before image rendering)

## Examples

```
import torchvision.models as models; model = models.densenet121(pretrained=False, num_classes=2)
```

## Evaluation signals

- All input images are uniformly 500x500 pixels and each is a square N x N matrix rendering with N equal to the intersecting gene count
- Genes are ordered top-to-bottom by R hclust applied to the gene CCA matrix before rendering
- Per-sub-feature accuracies are in the reported range: SBS17b* 77.500%, SBS3* 72.500%, New 71.429%
- The classifier achieves a preliminary G4-vs-G2 separation consistent with the paper's finding for these three signatures
- The CCA input images derive from RNMF genePerMutSigs output (geneCumulativeContributionAbundance txt per signature)

## Limitations

- Relatively small number of individuals studied, a shortcoming for model learning
- More excellent learning models are needed to obtain accurate results; requires further research in the future
- Due to inconsistent sequencing background, only coding regions were uniformly analyzed, which introduces limitations
- No software version numbers, installation instructions, or code availability details are stated for DenseNet-121 in the paper

## Evidence

- [intro] deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done: "deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done"
- [abstract] CCA matrix image data of genes in signatures New, SBS3*, and SBS17b* were helpful for preliminary evaluation of shortened survival outcome: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile, ... Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile,
                 sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', 
                 Mutfreq"

---
name: gene-cca-matrix-to-image-conversion
description: Use when RNMF has produced gene CCA abundance matrices for genes under
  fixed mutational signatures (e.g. New, SBS3*, SBS17b*) and the goal is to render
  them as images for CNN-based classification or prognostic prediction.
when_to_use_negative:
- You already have a flat gene-feature table suitable for classical ML — image conversion
  adds no information and complicates the pipeline
- Your mutational signatures have not been fixed/estimated (no P and S matrices from
  RNMF), so no CCA matrices exist to render
- The cohort is too small to train a CNN — the article notes the relatively small
  number of individuals was a shortcoming for model learning
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0199
tools:
- name: RNMF
  role: Generates the gene cumulative contribution abundance (CCA) matrices via cumulativeCA/genePerMutSigs
    that are rendered as images
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Environment for RNMF and for 'hclust' hierarchical clustering used to order
    intersecting genes in the CCA matrix
- name: PyTorch
  role: Deep learning framework in which the 500x500 CCA images are used to train
    the DenseNet-121 classifier
- name: DenseNet-121
  role: Publicly available CNN architecture trained on the CCA images labeled by G2
    vs G4 survival outcome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:cfc9ea9dfb117f80ace3f9841662d7a39b03079e39b164a2dd6012d5138fbbd5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - DenseNet
  - DenseNet-121
  - PyTorch
  - R
  - RNMF
  - hclust
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# gene-cca-matrix-to-image-conversion

## Summary

Converts RNMF gene cumulative contribution abundance (CCA) matrices into N x N gene-by-gene images (resized to 500x500 pixels) so deep learning classifiers such as DenseNet-121 can be trained on mutational-signature–gene association data.

## When to use

Use when RNMF has produced gene CCA abundance matrices for genes under fixed mutational signatures (e.g. New, SBS3*, SBS17b*) and the goal is to render them as images for CNN-based classification or prognostic prediction.

## When NOT to use

- You already have a flat gene-feature table suitable for classical ML — image conversion adds no information and complicates the pipeline
- Your mutational signatures have not been fixed/estimated (no P and S matrices from RNMF), so no CCA matrices exist to render
- The cohort is too small to train a CNN — the article notes the relatively small number of individuals was a shortcoming for model learning

## Inputs

- RNMF gene CCA matrix files (e.g. samplesResults/<sig>.SBS.geneCumulativeContributionAbundance.txt per signature)
- Non-silent mutated gene list from MAF file (Regular.ESCC.exon.WES.maf-input-nonsilent.txt)
- 717 genes list used for intersection processing
- Group file assigning samples to survival outcome groups (G2 vs G4)

## Outputs

- N x N CCA matrix images (N = number of intersecting genes) uniformly resized to 500x500 pixels
- Image dataset labeled by G2/G4 survival outcome for DenseNet-121 training
- Trained DenseNet-121 classifier with accuracy per signature sub-feature (e.g. SBS17b* 77.500%, SBS3* 72.500%, New 71.429%)

## How to apply

First obtain gene CCA matrices from RNMF's cumulativeCA/genePerMutSigs outputs for genes under each fixed signature, intersecting the non-silent mutated gene list with the 717 genes list to define the final gene set. Order the intersecting genes using R's 'hclust' hierarchical clustering applied to the gene CCA matrix, arranging genes from top to bottom so the matrix layout is deterministic. Organize each sample's data into an N x N matrix (N = number of intersecting genes), render it as an image, and uniformly resize all pictures to 500x500 pixels. Label each image with the sample's survival outcome group (shortened G4 vs longer G2) and feed the images to a DenseNet-121 classifier in PyTorch. The image representation lets the CNN exploit the 2D structure of gene-by-gene CCA abundances rather than treating genes as flat features.

## Related tools

- **RNMF** (Generates the gene cumulative contribution abundance (CCA) matrices via cumulativeCA/genePerMutSigs that are rendered as images) — https://github.com/zhenzhang-li/RNMF
- **R** (Environment for RNMF and for 'hclust' hierarchical clustering used to order intersecting genes in the CCA matrix)
- **PyTorch** (Deep learning framework in which the 500x500 CCA images are used to train the DenseNet-121 classifier)
- **DenseNet-121** (Publicly available CNN architecture trained on the CCA images labeled by G2 vs G4 survival outcome)

## Examples

```
Gfile = paste('samplesResults/', isig, '.SBS.geneCumulativeContributionAbundance.txt', sep=""); genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile, Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1)
```

## Evaluation signals

- All rendered images are uniformly 500x500 pixels and square N x N (N equals the number of genes after intersecting the non-silent mutated gene list with the 717-gene list)
- Gene ordering within each image is reproducible, derived from 'hclust' applied to the gene CCA matrix (top to bottom)
- Trained DenseNet-121 achieves preliminary G4 vs G2 separation with accuracies consistent with the reported finding (SBS17b* ~77.5%, SBS3* ~72.5%, New ~71.4%)
- Each input CCA text file (per signature) yields a corresponding image set, and labels trace back to the survival Group file

## Limitations

- The number of individuals studied is relatively small, a shortcoming for model learning; more excellent learning models are needed to obtain accurate results
- Only the G2 vs G4 comparison is preliminary discrimination — no software version numbers or code availability details are stated for DenseNet-121
- Incomplete clinical information (treatment, medication, follow-up) limits further prognostic exploration

## Evidence

- [abstract] the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile,: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile,"
- [intro] deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done: "deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done"
- [discussion] the number of individuals studied in this paper is relatively small, which is one of the shortcomings of model learning: "the number of individuals studied in this paper is relatively small, which is one of the shortcomings of model learning"

---
name: survival-outcome-image-classification
description: Use when non-image tabular data (gene-by-signature CCA matrices from
  RNMF) must be converted to N x N images and a deep CNN should be evaluated for binary
  survival-outcome discrimination.
when_to_use_negative:
- When the goal is de novo extraction of mutational signatures from raw mutation data
  rather than survival prediction from precomputed CCA matrices (use denovoNMF instead).
- When clinical information such as treatment methods, medication, or follow-up time
  is required for a robust prognostic model — the article notes such information was
  incomplete and the cohort relatively small for model learning.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_2640
tools:
- name: RNMF
  role: R package that generates the gene cumulative contribution abundance (CCA)
    matrices and per-signature gene files used as classifier inputs
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language of the RNMF package; used for hclust ordering of intersecting genes
    before image rendering
  repo: https://github.com/zhenzhang-li/RNMF
- name: PyTorch
  role: Deep learning framework in which the DenseNet-121 classifier is trained on
    500x500 CCA images
- name: DenseNet
  role: Publicly available DenseNet-121 architecture used for the G2 versus G4 survival-outcome
    image classification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:04de594428510c5bde53bbe2e30ceca3171b161b6a020a8bfae960dc9c807402
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# survival-outcome-image-classification

## Summary

Trains a DenseNet-121 convolutional classifier (PyTorch) on gene cumulative contribution abundance (CCA) matrix images rendered at 500x500 pixels to preliminarily discriminate shortened (G4) versus longer (G2) survival outcome in ESCC.

## When to use

Use when non-image tabular data (gene-by-signature CCA matrices from RNMF) must be converted to N x N images and a deep CNN should be evaluated for binary survival-outcome discrimination.

## When NOT to use

- When the goal is de novo extraction of mutational signatures from raw mutation data rather than survival prediction from precomputed CCA matrices (use denovoNMF instead).
- When clinical information such as treatment methods, medication, or follow-up time is required for a robust prognostic model — the article notes such information was incomplete and the cohort relatively small for model learning.

## Inputs

- RNMF gene CCA matrices (geneCumulativeContributionAbundance .txt files) for signatures New, SBS3*, SBS17b*
- Non-silent mutated gene list (MAF-derived, e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt)
- 717-gene list for intersection
- Survival outcome group labels (G2 vs G4 Groupfile)

## Outputs

- Gene-ordered N x N CCA matrix images resized to 500x500 pixels
- Trained DenseNet-121 classifier (PyTorch) for G2 vs G4 discrimination
- Per-signature classification accuracies (reported: SBS17b* 77.500%, SBS3* 72.500%, New 71.429%)

## How to apply

First obtain RNMF-generated CCA matrices for genes under the fixed signatures New, SBS3*, and SBS17b*, intersecting the non-silent mutated gene list with the 717-gene list. Order the intersecting genes using R's 'hclust' hierarchical clustering applied to the gene CCA matrix, arranging genes top to bottom. Organize the data into N x N matrices (N = number of intersecting genes), render them as images, and uniformly resize all pictures to 500x500 pixels. Train the publicly available DenseNet-121 architecture in PyTorch on the CCA images labeled by the G2 versus G4 survival outcome groups. Judge success by preliminary separation of G4 versus G2 groups consistent with reported accuracies: SBS17b* 77.500%, SBS3* 72.500%, New 71.429%.

## Related tools

- **RNMF** (R package that generates the gene cumulative contribution abundance (CCA) matrices and per-signature gene files used as classifier inputs) — https://github.com/zhenzhang-li/RNMF
- **R** (Language of the RNMF package; used for hclust ordering of intersecting genes before image rendering) — https://github.com/zhenzhang-li/RNMF
- **PyTorch** (Deep learning framework in which the DenseNet-121 classifier is trained on 500x500 CCA images)
- **DenseNet** (Publicly available DenseNet-121 architecture used for the G2 versus G4 survival-outcome image classification)

## Examples

```
genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- 500x500 resized CCA images are produced for each of the three signatures (New, SBS3*, SBS17b*) from the intersected gene set.
- Trained DenseNet-121 model achieves preliminary G4-vs-G2 separation; SBS17b* accuracy should approach ~77.500%, SBS3* ~72.500%, New ~71.429%.
- Gene ordering matches R 'hclust' hierarchical clustering of the gene CCA matrix, genes arranged top to bottom.
- Gene set equals the intersection of the non-silent mutated gene list with the 717-gene list (N x N matrix dimension consistency).

## Limitations

- The number of individuals studied is relatively small, which is a shortcoming for model learning.
- More excellent learning models are needed to obtain accurate results; further research is required.
- Complete clinical information (treatment methods, medication, extended follow-up time) was incomplete, limiting deeper prognostic exploration.
- Due to inconsistent sequencing background, only coding-region data were uniformly analyzed, introducing limitations.

## Evidence

- [abstract] the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [intro] deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done: "deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done"
- [methods] do intersection processing with 717 genes: "do intersection processing with 717 genes"
- [readme] Gfile = paste('samplesResults/',isig,'.',AnalCOSMICSigType,'.geneCumulativeContributionAbundance.txt',sep=""): "Gfile = paste('samplesResults/',isig,'.',AnalCOSMICSigType,'.geneCumulativeContributionAbundance.txt',sep="")"

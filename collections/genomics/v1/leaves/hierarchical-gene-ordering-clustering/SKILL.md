---
name: hierarchical-gene-ordering-clustering
description: Use when gene rows/columns of RNMF CCA matrices must be given a stable,
  biologically-informed order (via hierarchical clustering) before converting them
  into N x N image data for image-based deep learning.
when_to_use_negative:
- You already have a gene expression or feature table — this skill is specific to
  ordering RNMF gene CCA matrices derived from mutation data
- The signature of interest is not among those with CCA output (the study's useful
  signatures were New, SBS3*, and SBS17b*)
- The downstream model does not require a fixed image grid, since arbitrary gene ordering
  would suffice
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2648
tools:
- name: RNMF
  role: R package that generates the gene cumulative contribution abundance (CCA)
    matrices and per-signature gene files (genePerMutSigs) that the clustering step
    operates on
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF and the hclust hierarchical clustering gene-ordering
    step are implemented
  repo: https://github.com/zhenzhang-li/RNMF
- name: DenseNet-121
  role: PyTorch image classifier trained on the clustered, 500x500-resized CCA matrix
    images to discriminate G2 vs G4 survival outcome
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:b92c562a0671b5e1a09995095fc890c18e05ea1b21b96ac13bad3a7518a56049
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhenzhang-li/RNMF
  tools:
  - DenseNet
  - DenseNet-121
  - R
  - RNMF
  - hclust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2648
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Hierarchical gene ordering of CCA matrices

## Summary

Orders the intersecting non-silent mutated genes (against a 717-gene list) using R's hclust hierarchical clustering applied to the gene cumulative contribution abundance (CCA) matrix, so that genes are arranged top-to-bottom before rendering N x N CCA matrix images (resized to 500x500 pixels) for DenseNet-121 classification of G2 vs G4 survival outcome.

## When to use

Use when gene rows/columns of RNMF CCA matrices must be given a stable, biologically-informed order (via hierarchical clustering) before converting them into N x N image data for image-based deep learning.

## When NOT to use

- You already have a gene expression or feature table — this skill is specific to ordering RNMF gene CCA matrices derived from mutation data
- The signature of interest is not among those with CCA output (the study's useful signatures were New, SBS3*, and SBS17b*)
- The downstream model does not require a fixed image grid, since arbitrary gene ordering would suffice

## Inputs

- RNMF gene CCA matrix files (.geneCumulativeContributionAbundance.txt) for signatures New, SBS3*, SBS17b*
- Non-silent mutated gene list derived from the ESCC MAF file (e.g. 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt')
- 717-gene list used for intersection processing
- RNMF P-matrix (Pfile) and S-matrix (Sfile) for the chosen SBS signatures
- Survival group labels (Groupfile) assigning samples to G2 (longer) vs G4 (shortened) outcome

## Outputs

- Ordered (clustered) gene list per signature, arranged top to bottom
- N x N gene CCA matrix images (N = number of intersecting genes), uniformly resized to 500x500 pixels
- Trained DenseNet-121 classifier and G2-vs-G4 discrimination accuracy per signature

## How to apply

First intersect the non-silent mutated gene list (extracted from the MAF, e.g. 'Regular.ESCC.exon.WES.maf-input-nonsilent.txt') with the 717-gene list to obtain the final gene set for the chosen signature (New, SBS3*, or SBS17b*). Take the per-signature gene CCA abundance output produced by genePerMutSigs (the .geneCumulativeContributionAbundance.txt Gfile) and apply R's hclust hierarchical clustering to the gene CCA matrix to define the gene ordering, arranging genes from top to bottom. Build the N x N matrix (N = number of intersecting genes) using the clustered gene order, render it as an image, and uniformly resize all pictures to 500x500 pixels so they can be fed to a fixed-size DenseNet-121 classifier in PyTorch. The rationale is that clustering places genes with similar cumulative contribution abundance profiles adjacent to one another, giving the convolutional network a spatially coherent input rather than an arbitrary gene order. Train/evaluate per signature (G2 vs G4 labels) and compare against the reported accuracies of 77.500% (SBS17b*), 72.500% (SBS3*), and 71.429% (New).

## Related tools

- **RNMF** (R package that generates the gene cumulative contribution abundance (CCA) matrices and per-signature gene files (genePerMutSigs) that the clustering step operates on) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF and the hclust hierarchical clustering gene-ordering step are implemented) — https://github.com/zhenzhang-li/RNMF
- **DenseNet-121** (PyTorch image classifier trained on the clustered, 500x500-resized CCA matrix images to discriminate G2 vs G4 survival outcome)

## Examples

```
genePerMutSigs(Sfile = Sfile, choose.Sigs = 'SBS17b', Gfile = 'samplesResults/SBS17b.SBS.geneCumulativeContributionAbundance.txt', file = maffile, sample.id = 'Tumor_Sample_Barcode', Hugo = 'Hugo_Symbol', Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1, plot = TRUE)
```

## Evaluation signals

- The gene set equals the intersection of the non-silent mutated gene list with the 717-gene list; N equals the number of intersecting genes and the rendered matrix is N x N
- All generated CCA images are uniformly 500x500 pixels before training
- Gene ordering is derived from hclust applied to the gene CCA matrix (deterministic given the same CCA input and linkage)
- Classifier accuracy for the G2-G4 comparison falls near the reported values: SBS17b* 77.500%, SBS3* 72.500%, New 71.429%
- genePerMutSigs was run with the documented thresholds (Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1) before clustering

## Limitations

- The relatively small number of individuals studied is a shortcoming for model learning
- Incomplete clinical information (treatment methods, medication information, extended follow-up time) limits further prognostic exploration
- The framework is slower than previously published software for de novo signature extraction, due to the initial-value correction and solution-space processes

## Evidence

- [other] Order the intersecting genes using R's 'hclust' hierarchical clustering applied to the gene CCA matrix, arranging genes from top to bottom.: [evidence span withheld — anchor status: not_found]
- [abstract] the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [methods] do intersection processing with 717 genes: "do intersection processing with 717 genes"
- [methods] The R package is publicly available at https://github.com/zhenzhang-li/RNMF: [evidence span withheld — anchor status: not_found]
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile,: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, Gfile = Gfile, file = maffile,"

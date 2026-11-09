---
name: mutational-signature-gene-cca-matrix-generation
description: Use when you have a non-silent mutation MAF file plus RNMF P/S signature
  matrices for an ESCC cohort and need per-signature gene CCA matrices as image inputs
  for survival-outcome classification.
when_to_use_negative:
- You only need de novo signature extraction — use denovoNMF directly; CCA matrix
  generation requires pre-computed P and S signature matrices.
- Your mutation set includes silent mutations — the workflow operates on the non-silent
  MAF input intersected with the 717 genes list.
- You need population-scale prognostic modeling with complete clinical covariates
  — the article notes incomplete clinical information (treatment, medication, follow-up)
  and a relatively small cohort limit model learning.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RNMF
  role: R package providing cumulativeCA and genePerMutSigs to compute gene cumulative
    contribution abundance matrices from MAF data and signature P/S matrices
  repo: https://github.com/zhenzhang-li/RNMF
- name: R
  role: Language in which RNMF is implemented; hclust is used to order intersecting
    genes top-to-bottom in the CCA matrix
- name: hclust
  role: R hierarchical clustering function applied to the gene CCA matrix to arrange
    genes before image rendering
- name: PyTorch
  role: Deep learning framework used to train DenseNet-121 on the 500x500 CCA images
    with G2/G4 survival labels
- name: DenseNet
  role: Publicly available image classifier architecture applied to CCA matrix images
    for survival outcome prediction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/cam4.4717
    title: A practical framework <scp>RNMF</scp> for exploring the association between
      mutational signatures and genes using gene cumulative contribution abundance
  source_content_hash: sha256:49e98640a8b7f6d0d618218d55beac99c2f5691f762cd1a13637206f4899bbbc
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# mutational-signature-gene-cca-matrix-generation

## Summary

Generates gene-level cumulative contribution abundance (CCA) matrices from RNMF signature results and non-silent MAF mutation data, then renders them as ordered N x N images (resized to 500x500) that serve as image features for deep-learning prognostic classification (e.g., DenseNet-121 G2 vs G4 ESCC survival discrimination).

## When to use

Use when you have a non-silent mutation MAF file plus RNMF P/S signature matrices for an ESCC cohort and need per-signature gene CCA matrices as image inputs for survival-outcome classification.

## When NOT to use

- You only need de novo signature extraction — use denovoNMF directly; CCA matrix generation requires pre-computed P and S signature matrices.
- Your mutation set includes silent mutations — the workflow operates on the non-silent MAF input intersected with the 717 genes list.
- You need population-scale prognostic modeling with complete clinical covariates — the article notes incomplete clinical information (treatment, medication, follow-up) and a relatively small cohort limit model learning.

## Inputs

- Non-silent mutation MAF file (e.g. Regular.ESCC.exon.WES.maf-input-nonsilent.txt) with Tumor_Sample_Barcode, Chromosome, Start_position, Reference_Allele, Tumor_Seq_Allele2, Hugo_Symbol columns
- RNMF P signature matrix file (SBS.Pmatrix.txt) and S abundance matrix file (SBS.Smatrix.txt)
- Group file with sample survival outcome labels (G2 vs G4)
- 717 genes list used for intersection filtering
- COSMIC mutational signature names to process (Sigstype: SBS3, SBS16, SBS18, New, SBS33, SBS13, SBS2, SBS5, SBS1, SBS17b, SBS22, SBS15)

## Outputs

- Per-signature gene CCA table: <signature>.SBS.geneCumulativeContributionAbundance.txt in samplesResults/
- CCA association plots per signature (plot = TRUE)
- N x N gene CCA matrix images with genes ordered by hclust, uniformly resized to 500x500 pixels
- Trained DenseNet-121 (PyTorch) classifier discriminating G4 vs G2 survival groups

## How to apply

First obtain the RNMF signature matrices (Pfile='SBS.Pmatrix.txt', Sfile='SBS.Smatrix.txt') and a non-silent mutation MAF file, then run cumulativeCA and, per signature (e.g. SBS3, New, SBS17b from the COSMIC v2.0-based list), run genePerMutSigs with the article's thresholds Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1 to emit '<signature>.SBS.geneCumulativeContributionAbundance.txt'. Intersect the non-silent mutated gene list with the 717 genes list to fix the final gene set. Order the intersecting genes with R's hclust hierarchical clustering applied to the gene CCA matrix, top to bottom. Build the N x N matrix (N = number of intersecting genes), render it as an image, and uniformly resize all pictures to 500x500 pixels so they can train DenseNet-121 in PyTorch with G2 (longer) vs G4 (shortened) survival labels.

## Related tools

- **RNMF** (R package providing cumulativeCA and genePerMutSigs to compute gene cumulative contribution abundance matrices from MAF data and signature P/S matrices) — https://github.com/zhenzhang-li/RNMF
- **R** (Language in which RNMF is implemented; hclust is used to order intersecting genes top-to-bottom in the CCA matrix)
- **hclust** (R hierarchical clustering function applied to the gene CCA matrix to arrange genes before image rendering)
- **PyTorch** (Deep learning framework used to train DenseNet-121 on the 500x500 CCA images with G2/G4 survival labels)
- **DenseNet** (Publicly available image classifier architecture applied to CCA matrix images for survival outcome prediction)

## Examples

```
genePerMutSigs(Sfile='SBS.Smatrix.txt', choose.Sigs='SBS3', Gfile='samplesResults/SBS3.SBS.geneCumulativeContributionAbundance.txt', file=maffile, sample.id='Tumor_Sample_Barcode', Hugo='Hugo_Symbol', Mutfreq=0.04, threshold=0.06, Qvalue=0.1, plot=TRUE)
```

## Evaluation signals

- For each signature in Sigstype, an output file named <signature>.SBS.geneCumulativeContributionAbundance.txt exists under samplesResults/.
- genePerMutSigs calls use the article thresholds Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1.
- The gene set equals the intersection of the non-silent mutated gene list with the 717 genes list, and gene rows are ordered by hclust applied to the gene CCA matrix.
- All rendered CCA images are N x N and uniformly resized to 500x500 pixels before DenseNet-121 training.
- Downstream validation: classifier achieves preliminary G4 vs G2 separation consistent with reported accuracies — SBS17b* 77.500%, SBS3* 72.500%, New 71.429%.

## Limitations

- Framework spends more time on de novo signature extraction than previously published software due to the 'Re-updated the initial value' correction and 'build a solution space' processes; needs future optimization.
- Inconsistent sequencing background in the data; only coding regions were uniformly analyzed, which introduces limitations.
- Relatively small number of individuals is a shortcoming for model learning, and incomplete clinical information (treatment, medication, extended follow-up) limits further exploration; better learning models are needed for accurate results.

## Evidence

- [abstract] the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome: "the CCA matrix image data of genes in the signatures New, SBS3*, and SBS17b* were helpful for the preliminary evaluation of shortened survival outcome"
- [methods] do intersection processing with 717 genes: "do intersection processing with 717 genes"
- [methods] Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1: "Mutfreq = 0.04, threshold = 0.06, Qvalue = 0.1"
- [intro] deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done: "deep learning method is used to explore the cumulative contribution abundance (CCA) matrix image data of gene under fixed features, and hierarchical learning of prognosis is done"
- [readme] genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile,: "genePerMutSigs(Sfile = Sfile, choose.Sigs = isig, 
                 Gfile = Gfile, file = maffile,"

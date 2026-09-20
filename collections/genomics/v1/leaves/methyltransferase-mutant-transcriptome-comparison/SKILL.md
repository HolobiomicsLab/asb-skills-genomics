---
name: methyltransferase-mutant-transcriptome-comparison
description: Use when RNA-seq reads are available for methyltransferase-deficient
  mutant strains plus several clonal wild-type isolates of the same bacterium, and
  the goal is to identify genes differentially expressed due to loss of a specific
  methylation motif.
when_to_use_negative:
- Input is single-molecule SMRT methylation data (aligned_reads.cmph5) rather than
  RNA-seq reads — use SMALR's SMsn/SMp protocols instead
- There are no clonal wild-type control isolates, so a mutant-vs-wild-type GLM contrast
  cannot be fit
- The aim is consensus-level methylome/motif discovery rather than quantifying methylation-loss
  effects on transcription
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SMALR
  role: Single-molecule methylation analysis used upstream to characterize the methylomes/motifs
    (5'-GWCAY, 5'-TCAN6TRG/5'-CYAN6TGA) whose loss is being tested by transcriptome
    comparison
  repo: github.com/fanglab/SMALR
- name: samtools
  role: Sort and index RNA-seq alignments against the H. pylori J99 reference before
    feature counting (version >= 0.1.18)
- name: R
  role: Environment (>= 3.0) in which the count matrix is filtered, TMM-normalized,
    and the negative-binomial GLM is fit with edgeR
- name: edgeR
  role: TMM normalization and likelihood-ratio tests of each mutant against wild-type
    isolates; DE calls at P<0.001 and FDR<0.05
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:ab42d895bc2ecee6da5719ed9e68dfac572f1dcc2f36c7937c51b86746234965
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SMALR
  - edgeR
  - samtools
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# methyltransferase-mutant-transcriptome-comparison

## Summary

A differential-expression workflow that quantifies how a genome-wide loss of a specific DNA methylation motif (e.g., 5'-GWCAY or 5'-TCAN6TRG/5'-CYAN6TGA in H. pylori J99) alters gene expression, by testing RNA-seq count data from methyltransferase mutants against multiple clonal wild-type isolates with edgeR's negative-binomial GLM framework.

## When to use

Use when RNA-seq reads are available for methyltransferase-deficient mutant strains plus several clonal wild-type isolates of the same bacterium, and the goal is to identify genes differentially expressed due to loss of a specific methylation motif.

## When NOT to use

- Input is single-molecule SMRT methylation data (aligned_reads.cmph5) rather than RNA-seq reads — use SMALR's SMsn/SMp protocols instead
- There are no clonal wild-type control isolates, so a mutant-vs-wild-type GLM contrast cannot be fit
- The aim is consensus-level methylome/motif discovery rather than quantifying methylation-loss effects on transcription

## Inputs

- Raw RNA-seq reads (FASTQ) for methyltransferase mutants and clonal wild-type isolates (e.g., HPXZ1401, HPXZ1398 and five wild-type H. pylori J99 isolates)
- Reference genome FASTA (e.g., H. pylori J99) with gene feature annotations for counting
- Aligned reads (BAM) for sorting/indexing with samtools
- Raw gene-by-sample count matrix loaded into R

## Outputs

- Sorted and indexed alignment files (BAM/BAI)
- Raw gene-by-sample count matrix
- edgeR_de_genes.csv containing per-gene log-fold-changes, P-values, and FDR values for each mutant-vs-wild-type contrast
- Counts of significantly differentially expressed genes per mutant at P<0.001 and FDR<0.05 (e.g., 38 genes for HPXZ1401, 41 for HPXZ1398)

## How to apply

Quality-check and align the deposited RNA-seq reads for each sample (e.g., five wild-type J99 isolates, HPXZ1401, HPXZ1398) against the reference FASTA, then sort and index the alignments with samtools. Count reads overlapping gene features per sample to build a raw gene-by-sample count matrix, load it into R (>= 3.0), filter low-count genes, and normalize library composition with edgeR's TMM normalization. Fit an edgeR negative-binomial generalized linear model and test each mutant against the wild-type isolates using likelihood-ratio tests. Call differentially expressed genes at P<0.001 and FDR<0.05, and write per-gene log-fold-changes, P-values, and FDR values to edgeR_de_genes.csv. Rationale: using multiple clonal wild-type isolates separates methylation-loss effects from clone-to-clone variation, since clonal populations can carry epigenetic heterogeneity.

## Related tools

- **SMALR** (Single-molecule methylation analysis used upstream to characterize the methylomes/motifs (5'-GWCAY, 5'-TCAN6TRG/5'-CYAN6TGA) whose loss is being tested by transcriptome comparison) — github.com/fanglab/SMALR
- **samtools** (Sort and index RNA-seq alignments against the H. pylori J99 reference before feature counting (version >= 0.1.18))
- **R** (Environment (>= 3.0) in which the count matrix is filtered, TMM-normalized, and the negative-binomial GLM is fit with edgeR)
- **edgeR** (TMM normalization and likelihood-ratio tests of each mutant against wild-type isolates; DE calls at P<0.001 and FDR<0.05)

## Examples

```
library(edgeR); y <- DGEList(counts=counts); keep <- rowSums(cpm(y)>1) >= 2; y <- y[keep,,keep.lib.sizes=FALSE]; y <- calcNormFactors(y, method="TMM"); fit <- glmFit(y, design); lrt <- glmLRT(fit, coef="mutant"); write.csv(topTags(lrt, n=Inf), "edgeR_de_genes.csv")
```

## Evaluation signals

- The HPXZ1401 contrast (loss of 5'-GWCAY methylation) reproduces 38 significantly DE genes
- The HPXZ1398 contrast (loss of 5'-TCAN6TRG/5'-CYAN6TGA methylation) reproduces 41 significantly DE genes
- Both contrasts use the same significance cutoffs: P<0.001 and FDR<0.05
- HPXZ1401 shows upregulation of flgE, flgB, flaG, groEL, and groES
- edgeR_de_genes.csv contains per-gene log-fold-change, P-value, and FDR columns for each mutant-vs-wild-type contrast

## Limitations

- Existing consensus-based methylation methods cannot provide the resolution necessary to survey complex MTase activity, so linking methylation loss to expression depends on accurate single-molecule methylation calls upstream
- The study focused on cultures of single bacterial strains; applying the design to mixed bacterial populations (clinical isolates, microbiome samples) remains future work
- Low-count genes must be filtered before TMM normalization; inadequate filtering or insufficient replication can undermine the negative-binomial GLM contrast

## Evidence

- [abstract] Bacterial cells in a clonal population can generate epigenetic heterogeneity to increase population-level phenotypic plasticity: "Bacterial cells in a clonal population can generate epigenetic heterogeneity to increase population-level phenotypic plasticity"
- [intro] SMALR framework description: "The SMALR package conducts single-molecule level interrogation of the methylation status of SMRT reads."
- [methods] R version dependency: "* samtools >= 0.1.18
* R >= 3.0"
- [readme] README SMALR summary: "SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads."
- [full_text] DE gene counts and thresholds for the two mutants: [evidence span withheld — anchor status: not_found]

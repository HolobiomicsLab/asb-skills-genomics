---
name: edger-negative-binomial-differential-expression-testing
description: Use when you have a raw gene-by-sample RNA-seq count matrix and must
  call differentially expressed genes between mutant and wild-type conditions using
  edgeR TMM normalization plus a negative-binomial GLM with likelihood-ratio tests.
when_to_use_negative:
- The question concerns methylation status of SMRT reads (methylated vs non-methylated
  motif sites) rather than gene expression — use SMALR's SMsn or SMp protocols instead.
- Input is already a normalized expression matrix or population-level consensus methylation
  calls rather than a raw gene-by-sample count matrix.
- There are no biological replicates (e.g., only a single wild-type sample), since
  the GLM contrast here depends on multiple clonal wild-type isolates.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: edgeR
  role: TMM normalization of library composition, negative-binomial GLM fitting, and
    likelihood-ratio testing of mutant vs wild-type contrasts; DE calls at P<0.001
    and FDR<0.05
- name: R
  role: Runtime environment (>= 3.0) in which the count matrix is loaded, low-count
    genes filtered, and edgeR models fit
- name: samtools
  role: Sort and index RNA-seq alignments (>= 0.1.18) against the H. pylori J99 reference
    before gene-level read counting
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:22b650e0551ea95bf8c16755ed904e678208c926deb79af52f976a51464be127
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# edger-negative-binomial-differential-expression-testing

## Summary

Differential-expression testing of RNA-seq gene count matrices with edgeR's negative-binomial generalized linear model, used here to quantify transcriptome changes in H. pylori J99 methyltransferase mutants (HPXZ1401, HPXZ1398) against five clonal wild-type isolates.

## When to use

Use when you have a raw gene-by-sample RNA-seq count matrix and must call differentially expressed genes between mutant and wild-type conditions using edgeR TMM normalization plus a negative-binomial GLM with likelihood-ratio tests.

## When NOT to use

- The question concerns methylation status of SMRT reads (methylated vs non-methylated motif sites) rather than gene expression — use SMALR's SMsn or SMp protocols instead.
- Input is already a normalized expression matrix or population-level consensus methylation calls rather than a raw gene-by-sample count matrix.
- There are no biological replicates (e.g., only a single wild-type sample), since the GLM contrast here depends on multiple clonal wild-type isolates.

## Inputs

- RNA-seq FASTQ reads for each sample (five wild-type J99 isolates, HPXZ1401, HPXZ1398)
- H. pylori J99 reference genome FASTA and gene-feature annotation (GFF/GTF) for read counting
- Raw gene-by-sample count matrix (one column per sample)

## Outputs

- edgeR_de_genes.csv with per-gene log-fold-changes, P-values, and FDR values
- Differentially expressed gene lists per mutant contrast (38 DE genes for HPXZ1401, 41 for HPXZ1398 at P<0.001, FDR<0.05)

## How to apply

Quality-check and align deposited RNA-seq reads for all samples (five wild-type J99 isolates, HPXZ1401, HPXZ1398) against the H. pylori J99 reference FASTA, then sort and index alignments with samtools (>= 0.1.18). Count reads overlapping gene features per sample to build a raw gene-by-sample count matrix. In R (>= 3.0), filter low-count genes and normalize library composition with edgeR's TMM normalization. Fit an edgeR negative-binomial GLM and test each mutant against the wild-type isolates using likelihood-ratio tests, calling DE genes at P<0.001 and FDR<0.05. Rationale: the negative-binomial GLM with TMM handles library-composition differences among clonal isolates, and the strict P<0.001/FDR<0.05 cutoffs reproduce the published mutant-versus-wildtype contrasts.

## Related tools

- **edgeR** (TMM normalization of library composition, negative-binomial GLM fitting, and likelihood-ratio testing of mutant vs wild-type contrasts; DE calls at P<0.001 and FDR<0.05)
- **R** (Runtime environment (>= 3.0) in which the count matrix is loaded, low-count genes filtered, and edgeR models fit)
- **samtools** (Sort and index RNA-seq alignments (>= 0.1.18) against the H. pylori J99 reference before gene-level read counting)

## Examples

```
Rscript -e 'library(edgeR); y<-DGEList(counts=cnt); y<-calcNormFactors(y,method="TMM"); fit<-glmFit(y,design); lrt<-glmLRT(fit,contrast=c); write.csv(topTags(lrt),"edgeR_de_genes.csv")'
```

## Evaluation signals

- HPXZ1401 contrast reproduces exactly 38 significantly differentially expressed genes at P<0.001, FDR<0.05.
- HPXZ1398 contrast reproduces exactly 41 significantly differentially expressed genes at P<0.001, FDR<0.05.
- HPXZ1401 shows upregulation of flgE, flgB, flaG, groEL, and groES.
- edgeR_de_genes.csv contains per-gene log-fold-changes, P-values, and FDR values for every tested gene.
- Library composition normalization is TMM (calcNormFactors), not raw-count or RPKM-based scaling.

## Limitations

- The validation is brittle to threshold choice: DE gene counts (38 and 41) must be reproduced exactly at P<0.001, FDR<0.05, so deviations in filtering or normalization will break the check.
- Contrasts require the five clonal wild-type J99 isolates as the reference group; other wild-type groupings may not reproduce the published counts.
- Differential expression alone cannot explain methylation function; the article notes integration with other single molecule- or single cell-level data (e.g., RNA and protein expression) is needed for a detailed functional understanding.

## Evidence

- [other] Load the count matrix into R (>= 3.0), filter low-count genes, and normalize library composition with edgeR's TMM normalization.: [evidence span withheld — anchor status: not_found]
- [other] Fit an edgeR negative-binomial generalized linear model and test each mutant against the wild-type isolates using likelihood-ratio tests.: [evidence span withheld — anchor status: not_found]
- [other] Call differentially expressed genes at P<0.001 and FDR<0.05, and write the per-gene log-fold-changes, P-values, and FDR values to edgeR_de_genes.csv.: [evidence span withheld — anchor status: not_found]
- [discussion] Integration with RNA/protein expression data is framed as needed to understand methylation function.: "The application of SMALR and its integration with other single molecule- or single cell-level data, such as RNA and protein expression will enable a more detailed understanding of the functions of"

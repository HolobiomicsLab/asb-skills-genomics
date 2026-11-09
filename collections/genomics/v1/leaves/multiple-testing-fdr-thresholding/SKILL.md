---
name: multiple-testing-fdr-thresholding
description: Use when thousands of paired hypothesis tests (e.g., per-gene edgeR likelihood-ratio
  tests on a gene-by-sample RNA-seq count matrix) must yield a defensible DE gene
  list via a raw-P plus FDR cutoff.
when_to_use_negative:
- Calling per-site methylation status with SMALR's SMsn protocol, which estimates
  methylated/non-methylated site fractions without the use of subjective thresholds
  rather than applying a per-test FDR cutoff
- A single pre-specified hypothesis test (no family of simultaneous tests), where
  a raw P-value threshold alone is appropriate
- Input is already an adjusted-P/FDR table from a prior statistical step, which only
  needs thresholding/filtering, not a new GLM fit
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: edgeR
  role: Performs TMM normalization, negative-binomial GLM fitting, likelihood-ratio
    tests, and FDR computation for the per-gene DE calls
- name: R
  role: Runtime environment (>= 3.0) in which the count matrix is loaded, filtered,
    and analyzed with edgeR
- name: samtools
  role: Sorts and indexes alignments used to produce the raw gene-by-sample count
    matrix upstream of the testing step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:91d2ad4ea1ae9adc0a0e32b6b0273ec226ab1ed0de0513c29b4bc1765b36a7eb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - samtools
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# multiple-testing-fdr-thresholding

## Summary

Applies a dual significance cutoff (raw P < 0.001 and FDR < 0.05) to a genome-scale family of simultaneous statistical tests, as done in the edgeR differential-expression comparison of H. pylori J99 methylation mutants versus five clonal wild-type isolates.

## When to use

Use when thousands of paired hypothesis tests (e.g., per-gene edgeR likelihood-ratio tests on a gene-by-sample RNA-seq count matrix) must yield a defensible DE gene list via a raw-P plus FDR cutoff.

## When NOT to use

- Calling per-site methylation status with SMALR's SMsn protocol, which estimates methylated/non-methylated site fractions without the use of subjective thresholds rather than applying a per-test FDR cutoff
- A single pre-specified hypothesis test (no family of simultaneous tests), where a raw P-value threshold alone is appropriate
- Input is already an adjusted-P/FDR table from a prior statistical step, which only needs thresholding/filtering, not a new GLM fit

## Inputs

- Raw gene-by-sample count matrix (RNA-seq counts over gene features, e.g., from samtools-sorted alignments of H. pylori J99 samples)
- Sample/group design matrix distinguishing five clonal wild-type J99 isolates from mutants HPXZ1401 and HPXZ1398
- Numeric threshold parameters: P < 0.001 and FDR < 0.05

## Outputs

- edgeR_de_genes.csv with per-gene log-fold-changes, P-values, and FDR values
- DE gene counts per mutant contrast (reported as 38 for HPXZ1401 and 41 for HPXZ1398)

## How to apply

Build the raw gene-by-sample count matrix, filter low-count genes, and normalize library composition with edgeR's TMM normalization in R (>= 3.0). Fit an edgeR negative-binomial generalized linear model and test each mutant (HPXZ1401, HPXZ1398) against the five wild-type J99 isolates using likelihood-ratio tests. Call differentially expressed genes at the joint threshold P < 0.001 and FDR < 0.05, so that only genes passing both the per-test and genome-wide-adjusted criteria are reported. Write the per-gene log-fold-changes, P-values, and FDR values to edgeR_de_genes.csv. The rationale for the dual cutoff is that P < 0.001 controls per-test noise while FDR < 0.05 bounds the expected fraction of false discoveries across the whole gene family.

## Related tools

- **edgeR** (Performs TMM normalization, negative-binomial GLM fitting, likelihood-ratio tests, and FDR computation for the per-gene DE calls)
- **R** (Runtime environment (>= 3.0) in which the count matrix is loaded, filtered, and analyzed with edgeR)
- **samtools** (Sorts and indexes alignments used to produce the raw gene-by-sample count matrix upstream of the testing step)

## Examples

```
Rscript -e 'library(edgeR); y <- calcNormFactors(DGEList(counts), method="TMM"); fit <- glmFit(y, design); lrt <- glmLRT(fit); res <- topTags(lrt, n=Inf); write.csv(res, "edgeR_de_genes.csv")'
```

## Evaluation signals

- The HPXZ1401 contrast must reproduce 38 significantly differentially expressed genes at P < 0.001 and FDR < 0.05
- The HPXZ1398 contrast must reproduce 41 significantly differentially expressed genes at the same thresholds
- The HPXZ1401 DE list must show upregulation of flgE, flgB, flaG, groEL, and groES
- edgeR_de_genes.csv must contain per-gene log-fold-changes, P-values, and FDR values, with FDR values monotonically consistent with the ranked P-values

## Limitations

- The P < 0.001 / FDR < 0.05 cutoffs are study-specific for the J99 mutant-vs-wildtype contrasts and are not transferable defaults for other count-matrix analyses
- SMALR's SMSN-based methylation estimation deliberately avoids subjective thresholds, so this FDR-thresholding workflow applies to the transcriptome (edgeR) contrast, not to the methylation-score interpretation
- FDR control depends on the filtered gene set; removing low-count genes changes the family of tests over which the FDR is computed

## Evidence

- [other] Call differentially expressed genes at P<0.001 and FDR<0.05, and write the per-gene log-fold-changes, P-values, and FDR values to edgeR_de_genes.csv.: [evidence span withheld — anchor status: not_found]
- [other] Fit an edgeR negative-binomial generalized linear model and test each mutant against the wild-type isolates using likelihood-ratio tests.: [evidence span withheld — anchor status: not_found]
- [discussion] The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds"

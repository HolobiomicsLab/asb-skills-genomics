---
name: fdr-multiple-testing-correction
description: Use when thousands of candidate hypotheses (e.g., isoform-pair GLM interaction
  terms from acorde coDIU testing) are tested simultaneously and per-test p-values
  must be converted to FDR-adjusted values before applying significance cutoffs.
when_to_use_negative:
- When only a handful of pre-specified hypotheses are tested and per-hypothesis error
  control (e.g., Bonferroni at the raw p-value level) is preferred over genome/transcriptome-wide
  FDR control.
- When the goal is differential gene expression testing rather than isoform-usage
  (DIU/coDIU) interaction testing — the acorde FDR filter is defined on cluster*cell-type
  and gene*cell-type GLM terms, not gene-level counts.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package implementing the DIU/coDIU analyses whose GLM interaction p-values
    undergo FDR correction; ships the isoform expression matrix from the manuscript
    study
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Language in which the negative-binomial GLMs are fit and in which FDR correction
    (e.g., p.adjust) is applied to interaction-term p-values
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:4e6449b49b01e6ac63e138cc04d102358c4f10f363faea6290aa1dd6cc1b85c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  - acorde (R package)
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# fdr-multiple-testing-correction

## Summary

Apply false discovery rate (FDR) correction to the p-values of GLM interaction terms tested across thousands of gene pairs in the acorde coDIU analysis, so that significant cluster*cell-type isoform-usage effects can be called while controlling the expected fraction of false positives among calls.

## When to use

Use when thousands of candidate hypotheses (e.g., isoform-pair GLM interaction terms from acorde coDIU testing) are tested simultaneously and per-test p-values must be converted to FDR-adjusted values before applying significance cutoffs.

## When NOT to use

- When only a handful of pre-specified hypotheses are tested and per-hypothesis error control (e.g., Bonferroni at the raw p-value level) is preferred over genome/transcriptome-wide FDR control.
- When the goal is differential gene expression testing rather than isoform-usage (DIU/coDIU) interaction testing — the acorde FDR filter is defined on cluster*cell-type and gene*cell-type GLM terms, not gene-level counts.

## Inputs

- Vector of raw p-values for GLM interaction terms (one per candidate same-gene isoform pair)
- ISO or transcript-level count/quantification matrix (acorde input)
- Isoform-to-gene mapping and cell-level cell-type/state labels

## Outputs

- FDR-adjusted p-values (adjusted interaction terms) per tested gene pair
- coDIU gene table (e.g., codiu_results.csv) listing genes with cluster*cell-type FDR < 0.05 and gene*cell-type FDR > 0.05, plus involved co-expressed isoforms

## How to apply

Fit negative-binomial GLMs for each candidate gene pair, modeling isoform usage as a function of cluster, cell type, and their interactions (cluster*cell-type and gene*cell-type terms), and collect the interaction-term p-values. Apply FDR multiple-testing correction over these interaction-term p-values (in R, e.g., via p.adjust with a Benjamini-Hochberg-type method) rather than testing each pair at raw p < 0.05, because the number of simultaneous tests would otherwise inflate false calls. Then apply the two-sided filter: retain pairs with significant cluster*cell-type interaction (FDR < 0.05) and non-significant gene*cell-type interaction (FDR > 0.05), which is the acorde definition of coDIU — isoform-level, cluster/cell-type-dependent co-usage not detectable at the gene level. Summarize the surviving genes and their co-expressed isoforms; in the Tasic et al. mouse visual cortex Smart-Seq2 dataset this yielded 1784 coDIU genes involving 5274 co-expressed isoforms.

## Related tools

- **acorde** (R package implementing the DIU/coDIU analyses whose GLM interaction p-values undergo FDR correction; ships the isoform expression matrix from the manuscript study) — https://github.com/ConesaLab/acorde
- **R** (Language in which the negative-binomial GLMs are fit and in which FDR correction (e.g., p.adjust) is applied to interaction-term p-values)

## Examples

```
adj_p <- p.adjust(glm_interaction_pvals, method = "BH"); codiu <- adj_p_cluster_celltype < 0.05 & adj_p_gene_celltype > 0.05
```

## Evaluation signals

- Number of retained coDIU genes matches the reported acorde result for the Tasic et al. mouse visual cortex Smart-Seq2 dataset: 1784 genes.
- Total co-expressed isoforms among retained coDIU genes matches the reported 5274.
- Every gene in codiu_results.csv satisfies both criteria: cluster*cell-type interaction FDR < 0.05 AND gene*cell-type interaction FDR > 0.05.
- Adjusted p-values are monotonically consistent with raw p-values (adjusted >= raw for the same test) and all fall within [0,1].
- The set of tested pairs equals the candidate set of gene pairs with at least two same-gene isoforms sharing isoform clusters, i.e., correction was applied over the full candidate hypothesis space, not a pre-filtered subset.

## Limitations

- FDR significance of the cluster*cell-type interaction alone is not sufficient: coDIU requires the gene*cell-type interaction to be non-significant (FDR > 0.05), so a significant gene-level interaction disqualifies a pair even if the isoform-level term passes.
- Cutoffs are calibrated to the acorde workflow (FDR < 0.05) applied to Smart-Seq2 data with hybrid bulk PacBio/ENCODE long-read transcriptomes; different scRNA-seq protocols or transcriptomes may need recalibration.
- FDR-corrected calls are statistical only; the article notes reported functional insights require further experimental validation and are currently hypothesis-generating.

## Evidence

- [methods] coDIU definition and thresholds from the acorde task: [evidence span withheld — anchor status: not_found]
- [intro] coDIU analyses implemented in acorde: "We additionally defined and implemented Differential Isoform Usage (DIU) and co-Differential Isoform Usage (coDIU) analyses in order to leverage the multiple cell types contained in single-cell"
- [abstract] coDIU detection from same-gene isoforms between clusters: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [readme] acorde provides DIU/coDIU functions with an isoform-level matrix input: "*acorde* contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."

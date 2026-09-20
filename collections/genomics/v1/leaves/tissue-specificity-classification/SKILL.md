---
name: tissue-specificity-classification
description: Use when testing whether circRNA isoforms flagged as tissue-specific
  by per-tissue read-count criteria are enriched in tissue-specific parental genes,
  or when computing tissue-specificity scores (tau) from multi-tissue gene TPM tables.
when_to_use_negative:
- Input data covers fewer than the 12 tissues used in the isoCirc catalog — tau computed
  on n tissues uses the (n-1) normalization and yields non-comparable classifications
- The analysis goal is circRNA isoform detection or BSJ consensus calling itself —
  use the isoCirc pipeline (pip install isocirc; bedtools, minimap2) instead
- Expression is provided as counts without normalization to comparable scales across
  tissues — convert to TPM first, as tau depends on relative per-gene expression profiles
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Implementation language for tau computation, contingency-table construction,
    and scipy.stats.fisher_exact enrichment testing
  repo: https://github.com/Xinglab/isoCirc
- name: isoCirc
  role: Upstream pipeline whose output isoform-level read-count table provides the
    12-tissue circRNA isoform catalog analyzed here
  repo: https://github.com/Xinglab/isoCirc
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:9fa3abd12f748283abfb582b39fb2ebe3c75e498e08c1571387eed18bead5aad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - Python
  - isoCirc
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# tissue-specificity-classification

## Summary

Classifies parental genes as tissue-specific using Yanai's tau index computed on GTEx V8 median gene TPM values across 12 human tissues, then tests whether isoCirc-identified tissue-specific circRNA isoforms are enriched among these genes via a one-sided Fisher's exact test on a 2x2 contingency table (p < 2.2e-16).

## When to use

Use when testing whether circRNA isoforms flagged as tissue-specific by per-tissue read-count criteria are enriched in tissue-specific parental genes, or when computing tissue-specificity scores (tau) from multi-tissue gene TPM tables.

## When NOT to use

- Input data covers fewer than the 12 tissues used in the isoCirc catalog — tau computed on n tissues uses the (n-1) normalization and yields non-comparable classifications
- The analysis goal is circRNA isoform detection or BSJ consensus calling itself — use the isoCirc pipeline (pip install isocirc; bedtools, minimap2) instead
- Expression is provided as counts without normalization to comparable scales across tissues — convert to TPM first, as tau depends on relative per-gene expression profiles

## Inputs

- 12-tissue circRNA isoform read-count table (columns: isoformID, chrom, coordinates, readCount per tissue)
- GTEx V8 median gene TPM values for the same 12 human tissues

## Outputs

- Per-gene Yanai's tau values with tissue-specific gene labels
- 2x2 contingency table (tissue-specific isoform yes/no vs tissue-specific gene yes/no)
- fisher_test_results.tsv containing the contingency table, odds ratio, and one-sided p-value

## How to apply

Load the 12-tissue circRNA isoform read-count table (isoformID, chrom, coordinates, readCount per tissue) into a pandas DataFrame and apply the paper's chi-square/binomial criteria per isoform to flag tissue-specific isoforms (the confirmed set totals 3,653 isoforms). Load GTEx V8 median gene TPM values for the same 12 tissues and compute Yanai's tau per parental gene: tau = 1 - sum_i(1 - x_i/max(x)) / (n-1), where x_i is the TPM in tissue i; genes with high tau are classified as tissue-specific. Cross-tabulate tissue-specific isoform status (yes/no) against tissue-specific parental gene status (yes/no) into a 2x2 contingency table. Run a one-sided Fisher's exact test with scipy.stats.fisher_exact to test enrichment of tissue-specific isoforms in tissue-specific genes. Write the contingency table, odds ratio, and p-value to fisher_test_results.tsv and verify the p-value satisfies p < 2.2e-16 as reported in the isoCirc paper (Nat. Commun., DOI 10.1038/s41467-020-20459-8).

## Related tools

- **Python** (Implementation language for tau computation, contingency-table construction, and scipy.stats.fisher_exact enrichment testing) — https://github.com/Xinglab/isoCirc
- **isoCirc** (Upstream pipeline whose output isoform-level read-count table provides the 12-tissue circRNA isoform catalog analyzed here) — https://github.com/Xinglab/isoCirc

## Examples

```
tau = 1 - np.sum(1 - tpm / tpm.max()) / (len(tpm) - 1); from scipy.stats import fisher_exact; odds, p = fisher_exact(tab, alternative='greater')
```

## Evaluation signals

- Chi-square/binomial flagging of the isoform read-count table yields exactly 3,653 tissue-specific circRNA isoforms
- The 2x2 contingency table is fully populated (all four cells) and row/column marginals match the input sets
- One-sided scipy.stats.fisher_exact p-value is < 2.2e-16, matching the paper's reported enrichment
- fisher_test_results.tsv exists with contingency table, odds ratio, and p-value columns, and odds ratio > 1
- tau values are bounded in [0, 1] and n-1 = 11 denominator is used for the 12 GTEx tissues

## Limitations

- isoCirc is far from saturation at current sequencing depth; additional circRNA isoforms are expected with deeper sequencing, which could alter the isoform set
- isoCirc was applied only to whole cells and tissues, so subcellular-fraction expression patterns are not captured
- Tissue-specific isoform calling depends on the chi-square/binomial per-tissue count criteria; isoforms with low read counts may be misclassified

## Evidence

- [results] Tissue-specific circRNA isoforms were significantly enriched for tissue-specific parental genes compared to random expectation, with a Fisher's exact test p-value < 2.2e-16.: [evidence span withheld — anchor status: not_found]
- [methods] Run a one-sided Fisher's exact test on the 2x2 table with scipy.stats.fisher_exact to test enrichment of tissue-specific isoforms in tissue-specific genes.: [evidence span withheld — anchor status: not_found]
- [methods] compute Yanai's tau index per parental gene: tau = 1 - sum_i(1 - x_i/max(x)) / (n-1): [evidence span withheld — anchor status: not_found]
- [abstract] we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293): "we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293), including 40,628 isoforms ≥500 nt in length"
- [readme] isoCirc is a long-read sequencing strategy coupled with an integrated computational pipeline to characterize full-length circRNA isoforms: "isoCirc is a long-read sequencing strategy coupled with an integrated computational pipeline to characterize full-length circRNA isoforms"

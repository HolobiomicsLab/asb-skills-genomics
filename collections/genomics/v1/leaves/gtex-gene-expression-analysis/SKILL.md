---
name: gtex-gene-expression-analysis
description: Use when you have a per-tissue circRNA isoform read-count table (e.g.,
  isoCirc's 12 human tissues) plus multi-tissue parental-gene expression (GTEx V8
  TPM) and need to test isoform-to-gene specificity enrichment.
when_to_use_negative:
- When the goal is to identify or call full-length circRNA isoforms from raw nanopore
  reads — that is the upstream isoCirc pipeline, not this enrichment test
- When parental-gene expression is available for only one tissue or condition — Yanai's
  tau across multiple tissues cannot be computed
- When you need a full-length circRNA catalog de novo rather than reanalysis of the
  published 12-tissue isoCirc read-count table
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: isoCirc
  role: Source of the full-length circRNA isoform catalog across 12 human tissues
    and HEK293 whose per-tissue read counts form the input table
  repo: https://github.com/Xinglab/isoCirc
- name: scipy.stats.fisher_exact
  role: One-sided Fisher's exact test on the 2x2 contingency table of tissue-specific
    isoforms vs tissue-specific parental genes
- name: GTEx V8
  role: Provides median gene TPM values across the 12 tissues used to compute Yanai's
    tau for tissue-specific parental-gene classification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-20459-8
    title: isoCirc catalogs full-length circular RNA isoforms in human transcriptomes
  source_content_hash: sha256:33f60dfedfdab71aa514b8ae77b5f17fb0aba1185a276bc09b40a0d1e5261ea5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Xinglab/isoCirc
  tools:
  - GTEx V8
  - Python
  - isoCirc
  - scipy.stats.fisher_exact
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gtex-gene-expression-analysis

## Summary

Quantifies parental-gene tissue specificity using GTEx V8 median gene TPM values and Yanai's tau index, then tests whether tissue-specific circRNA isoforms from the isoCirc 12-tissue catalog are enriched in tissue-specific parental genes via a one-sided Fisher's exact test.

## When to use

Use when you have a per-tissue circRNA isoform read-count table (e.g., isoCirc's 12 human tissues) plus multi-tissue parental-gene expression (GTEx V8 TPM) and need to test isoform-to-gene specificity enrichment.

## When NOT to use

- When the goal is to identify or call full-length circRNA isoforms from raw nanopore reads — that is the upstream isoCirc pipeline, not this enrichment test
- When parental-gene expression is available for only one tissue or condition — Yanai's tau across multiple tissues cannot be computed
- When you need a full-length circRNA catalog de novo rather than reanalysis of the published 12-tissue isoCirc read-count table

## Inputs

- 12-tissue circRNA isoform read-count table from isoCirc (columns: isoformID, chrom, coordinates, readCount per tissue)
- GTEx V8 median gene TPM values for the same 12 human tissues
- Mapping of circRNA isoform IDs to parental gene IDs

## Outputs

- Set of tissue-specific circRNA isoforms (expected total: 3,653)
- Yanai's tau index per parental gene (values in [0,1])
- 2x2 contingency table (tissue-specific isoform yes/no vs tissue-specific gene yes/no)
- Odds ratio and one-sided Fisher's exact test p-value written to fisher_test_results.tsv (expected p < 2.2e-16)

## How to apply

Load the 12-tissue circRNA isoform read-count table (isoformID, chrom, coordinates, readCount per tissue) into a pandas DataFrame and apply the paper's chi-square/binomial criteria to each isoform's per-tissue read counts to flag tissue-specific isoforms, confirming the set totals 3,653 isoforms. Load GTEx V8 median gene TPM values for the same 12 tissues and compute Yanai's tau per parental gene: tau = 1 - sum_i(1 - x_i/max(x)) / (n-1), with high tau defining tissue-specific genes. Cross-tabulate tissue-specific isoforms (yes/no) against tissue-specific parental genes (yes/no) into a 2x2 contingency table. Run a one-sided Fisher's exact test with scipy.stats.fisher_exact on that table; the rationale is that full-length circRNA isoform calls from isoCirc's RCA + nanopore catalog can be tied to parental genes whose expression breadth is independently measured by GTEx, so a significant odds ratio (>1) indicates isoform specificity follows host-gene specificity. Validate by writing the contingency table, odds ratio, and p-value to fisher_test_results.tsv and checking that p < 2.2e-16 is reproduced.

## Related tools

- **isoCirc** (Source of the full-length circRNA isoform catalog across 12 human tissues and HEK293 whose per-tissue read counts form the input table) — https://github.com/Xinglab/isoCirc
- **scipy.stats.fisher_exact** (One-sided Fisher's exact test on the 2x2 contingency table of tissue-specific isoforms vs tissue-specific parental genes)
- **GTEx V8** (Provides median gene TPM values across the 12 tissues used to compute Yanai's tau for tissue-specific parental-gene classification)

## Examples

```
from scipy.stats import fisher_exact; odds_ratio, p = fisher_exact([[a,b],[c,d]], alternative='greater')
```

## Evaluation signals

- The chi-square/binomial tissue-specificity criteria flag exactly 3,653 tissue-specific circRNA isoforms from the 12-tissue read-count table
- Computed Yanai's tau values all fall in [0,1] and high-tau genes are classified as tissue-specific
- The 2x2 contingency table row/column totals match the classified isoform and gene counts before testing
- The one-sided Fisher's exact test reproduces the reported odds ratio with p < 2.2e-16 in fisher_test_results.tsv

## Limitations

- isoCirc is far from saturation at the current sequencing depth, so the isoform read-count input undercounts true tissue-specific isoforms
- The catalog covers only 12 human tissues plus the HEK293 cell line; tissue-specificity conclusions are bounded by those samples
- Tissue specificity of isoforms is defined by per-tissue read-count shifts (chi-square/binomial criteria), not experimentally determined isoform-proportion replicates

## Evidence

- [abstract] we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293), including 40,628 isoforms ≥500 nt in length: "we generate a comprehensive catalog of 107,147 full-length circRNA isoforms across 12 human tissues and one human cell line (HEK293), including 40,628 isoforms ≥500 nt in length"
- [other] Tissue-specific circRNA isoforms were significantly enriched for tissue-specific parental genes compared to random expectation, with a Fisher's exact test p-value < 2.2e-16.: [evidence span withheld — anchor status: not_found]
- [discussion] the catalog of full-length circRNA isoforms is available at https://genome.ucsc.edu/s/xinglab_chop/isoCirc: "the catalog of full-length circRNA isoforms is available at https://genome.ucsc.edu/s/xinglab_chop/isoCirc"
- [readme] isoCirc is a long-read sequencing strategy coupled with an integrated computational pipeline to characterize full-length circRNA isoforms: "isoCirc is a long-read sequencing strategy coupled with an integrated computational pipeline to characterize full-length circRNA isoforms"

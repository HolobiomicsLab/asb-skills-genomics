---
name: minor-isoform-expression-filtering
description: Use when a consensus DE isoform set must be pruned before co-usage/coDIU
  analysis so that low-contribution (minor) isoforms and single-DE-isoform genes do
  not enter downstream isoform co-expression modeling.
when_to_use_negative:
- When the goal is gene-level DE rather than isoform co-usage — the 10% minor-isoform
  cutoff and single-DE-isoform-gene removal would discard valid gene-level signals.
- When the input is an unfiltered isoform quantification matrix that has not yet gone
  through consensus DIU significance selection; filtering must follow DE isoform selection.
- When isoform proportions cannot be reliably computed (e.g. transcriptome without
  isoform-to-gene relationships or per-isoform expression estimates).
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: acorde
  role: R package providing the minor-isoform filtering and DIU/coDIU workflow functions
    applied to the isoform expression matrix
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Environment in which the acorde pipeline and the filtering/export steps (consensus_diu_isoforms.csv)
    are executed
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:2cf9899657bb09c6482e3f55b04c7042e2cf84a3d72bbf6c01d615bcd08b8ece
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# minor-isoform-expression-filtering

## Summary

A post-DIU-selection filtering step from the acorde pipeline that removes isoforms whose expression is <10% of their gene's total expression, followed by exclusion of genes left with only a single DE isoform, to isolate genes with meaningful isoform co-usage.

## When to use

Use when a consensus DE isoform set must be pruned before co-usage/coDIU analysis so that low-contribution (minor) isoforms and single-DE-isoform genes do not enter downstream isoform co-expression modeling.

## When NOT to use

- When the goal is gene-level DE rather than isoform co-usage — the 10% minor-isoform cutoff and single-DE-isoform-gene removal would discard valid gene-level signals.
- When the input is an unfiltered isoform quantification matrix that has not yet gone through consensus DIU significance selection; filtering must follow DE isoform selection.
- When isoform proportions cannot be reliably computed (e.g. transcriptome without isoform-to-gene relationships or per-isoform expression estimates).

## Inputs

- Consensus DIU isoform table (isoform-to-gene mapping with per-method FDR significance across downsampling runs)
- Isoform-level single-cell expression matrix (e.g. TPM counts from the Tasic et al. Smart-Seq2 visual cortex dataset)

## Outputs

- Filtered isoform-gene table (consensus_diu_isoforms.csv) with 6794 isoforms from 2696 genes
- Count of isoforms removed by the minor-isoform step (expected: 969 from the 9393-isoform consensus)

## How to apply

After building a consensus DIU set (e.g. isoforms called significant with FDR<0.05 in ≥50% of 50 downsampling runs, giving 9393 isoforms from 4223 genes), compute each isoform's expression as a fraction of its gene's total per-cell expression. Remove isoforms contributing <10% of the gene's total — acorde's minor-isoform filter, which in this study removed 969 isoforms. Then exclude genes left with only one significant DE isoform, since no isoform co-usage pattern can be defined for them, yielding the final 6794 isoforms from 2696 genes exported as consensus_diu_isoforms.csv. The rationale is that minor isoforms carry negligible usage signal and single-isoform DE genes cannot exhibit co-differential isoform usage.

## Related tools

- **acorde** (R package providing the minor-isoform filtering and DIU/coDIU workflow functions applied to the isoform expression matrix) — https://github.com/ConesaLab/acorde
- **R** (Environment in which the acorde pipeline and the filtering/export steps (consensus_diu_isoforms.csv) are executed) — https://github.com/ConesaLab/acorde

## Examples

```
filtered <- iso_expr %>% group_by(gene_id) %>% mutate(prop = expr / sum(expr)) %>% filter(prop >= 0.10, n_distinct(isoform_id)) >= 2
```

## Evaluation signals

- The minor-isoform step removes exactly 969 isoforms from the 9393-isoform consensus set.
- The final exported table (consensus_diu_isoforms.csv) contains exactly 6794 isoforms from 2696 genes.
- Every retained isoform has expression ≥10% of its gene's total in the input expression matrix.
- No gene in the output has fewer than two retained DE isoforms.
- All output isoforms were present in the ≥50% run-frequency consensus DIU set.

## Limitations

- The 10% minor-isoform threshold is a fixed study-specific cutoff; isoforms just below the cutoff with real usage signal are lost.
- Genes with only one significant DE isoform are entirely excluded, so single-isoform differential usage events cannot be studied downstream.
- Functional annotation of long read-defined isoforms is limited to pre-annotated references via isoAnnotLite until a de novo annotation tool is available.

## Evidence

- [other] Apply acorde's minor-isoform filter, removing isoforms whose expression is <10% of their gene's total (removing 969 isoforms).: [evidence span withheld — anchor status: not_found]
- [other] Exclude genes left with only one significant DE isoform, since no isoform co-usage pattern can be defined for them.: [evidence span withheld — anchor status: not_found]
- [readme] acorde contains functions to obtain DIU and co-DIU genes from an isoform-level expression matrix.: "acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input."

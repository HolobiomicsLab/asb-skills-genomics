---
name: multiple-testing-benjamini-hochberg-correction
description: Use when a functional enrichment scan (e.g. tappAS partially-overlapping-samples
  z-test over positional features and GO terms) returns many unadjusted p-values that
  must be corrected for multiple testing before calling significant features.
when_to_use_negative:
- Only a single hypothesis is being tested (no family of enrichment p-values to correct)
- The enrichment tool already reports FDR-adjusted values, so a second correction
  would double-shrink significance
- The goal is genome-wide DE gene correction rather than correcting across functional
  feature/GO term tests in tappAS
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: tappAS
  role: Runs the functional enrichment analysis (partially overlapping samples z-test)
    whose raw p-value family is corrected with BH
  repo: https://github.com/ConesaLab/tappAS
- name: R
  role: Environment in which BH correction of the enrichment p-values is performed
    and results are written to codiu_functional_enrichment.tsv
  repo: https://github.com/ConesaLab/acorde
- name: Revigo
  role: Downstream redundancy removal applied AFTER BH-FDR filtering, reducing the
    59 enriched GO terms to 26 unique terms
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:4dd6bab311f6799d9c4d09363bcc6cca42b53112cb196683bf75a124c2c9a9cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/tappAS
  tools:
  - R
  - Revigo
  - acorde
  - tappAS
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multiple-testing-benjamini-hochberg-correction

## Summary

Apply the Benjamini-Hochberg (BH) false discovery rate procedure to the family of p-values produced by a tappAS functional enrichment comparison of coDIU vs DIU genes, retaining features significant at FDR < 0.05 before downstream redundancy filtering (Revigo).

## When to use

Use when a functional enrichment scan (e.g. tappAS partially-overlapping-samples z-test over positional features and GO terms) returns many unadjusted p-values that must be corrected for multiple testing before calling significant features.

## When NOT to use

- Only a single hypothesis is being tested (no family of enrichment p-values to correct)
- The enrichment tool already reports FDR-adjusted values, so a second correction would double-shrink significance
- The goal is genome-wide DE gene correction rather than correcting across functional feature/GO term tests in tappAS

## Inputs

- Vector of raw p-values from the tappAS functional enrichment analysis (coDIU vs DIU genes, partially overlapping samples z-test)
- Gene-to-condition table assigning genes to the coDIU and DIU sets
- tappAS Mus musculus RefSeq functional annotation with positional features and GO terms

## Outputs

- BH-adjusted p-values (FDR) for every tested functional feature
- Filtered set of significant features at FDR < 0.05 (91 positional functional features and 59 GO terms in this study)
- codiu_functional_enrichment.tsv with feature/GO term IDs and FDR values

## How to apply

Collect the raw p-values from the tappAS functional enrichment run that compared the coDIU gene set against the DIU gene set using the partially overlapping samples z-test. Apply the Benjamini-Hochberg procedure across ALL tested functional features (91 positional functional features and 59 GO terms were the retained survivors in this study). Threshold the adjusted p-values at FDR < 0.05 and carry only those features forward — in this workflow, the BH-FDR-filtered GO term list is subsequently submitted to Revigo, which reduced it to 26 unique non-redundant GO terms. Write the positional feature and GO term results together with their FDR values (e.g. to codiu_functional_enrichment.tsv) so the correction is traceable.

## Related tools

- **tappAS** (Runs the functional enrichment analysis (partially overlapping samples z-test) whose raw p-value family is corrected with BH) — https://github.com/ConesaLab/tappAS
- **R** (Environment in which BH correction of the enrichment p-values is performed and results are written to codiu_functional_enrichment.tsv) — https://github.com/ConesaLab/acorde
- **Revigo** (Downstream redundancy removal applied AFTER BH-FDR filtering, reducing the 59 enriched GO terms to 26 unique terms)

## Examples

```
padj <- p.adjust(enrichment_pvalues, method = "BH"); enriched <- names(padj)[padj < 0.05]
```

## Evaluation signals

- BH-FDR-filtered feature set reproduces 91 positional functional features and 59 GO terms enriched in coDIU vs DIU genes at FDR < 0.05
- After Revigo filtering of the significant GO list, exactly 26 unique GO terms remain
- Sanity check: every adjusted p-value is >= its raw p-value and adjusted values are monotone with raw p-value rank
- No uncorrected raw p-value is used for feature calls; all retained features carry an FDR value in the output table

## Limitations

- The acorde R package itself does not include the tappAS functional analyses — enrichment and its correction are run via external tools, per the README
- Revigo redundancy filtering is a separate post-hoc step; BH correction alone does not de-duplicate semantically similar GO terms
- Functional feature annotation depends on IsoAnnotLite transfer from pre-annotated tappAS references; features absent from the annotation cannot be tested or corrected

## Evidence

- [other] BH applied across all enrichment tests at FDR 0.05: [evidence span withheld — anchor status: not_found]
- [other] Expected recovery counts of significant features: [evidence span withheld — anchor status: not_found]
- [other] Revigo de-duplication downstream of BH filtering: [evidence span withheld — anchor status: not_found]
- [readme] Functional enrichment executed in tappAS, not in the acorde R package: [evidence span withheld — anchor status: external_doc]
- [readme] Annotation source for the features being enriched: [evidence span withheld — anchor status: external_doc]

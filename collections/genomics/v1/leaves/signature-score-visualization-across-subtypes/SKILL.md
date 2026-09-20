---
name: signature-score-visualization-across-subtypes
description: Use when a bulk RNA-seq expression matrix is annotated with disease/WHO
  subtypes and you need to compute per-sample gene-signature scores and identify the
  subtypes where a lineage signature peaks.
when_to_use_negative:
- Input is already a per-sample signature score table or a summarized subtype-level
  score table rather than a raw expression matrix
- The dataset is single-cell genotyping data (e.g., SNV/fusion/isoform detection from
  long-read cDNA libraries) rather than bulk gene-expression profiles with subtype
  annotations
- No sample-level subtype or diagnosis metadata exists, since scores cannot be summarized
  per subtype without it
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:fe2ecdb1783d832913f5f734e8274b26ccfc5455c04893f493c0864fcccad7f4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools: []
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# signature-score-visualization-across-subtypes

## Summary

Scores bulk RNA-seq samples with predefined gene-expression signatures (e.g., the 5-gene erythroid GATA1/CA1/HBA1/HBB/ALAD and 5-gene megakaryocytic GATA2/ZFPM1/FLI1/NFE2/PF4 signatures) by z-score normalizing each gene and averaging, then summarizing mean scores per disease subtype to reveal which WHO classes the signatures mark.

## When to use

Use when a bulk RNA-seq expression matrix is annotated with disease/WHO subtypes and you need to compute per-sample gene-signature scores and identify the subtypes where a lineage signature peaks.

## When NOT to use

- Input is already a per-sample signature score table or a summarized subtype-level score table rather than a raw expression matrix
- The dataset is single-cell genotyping data (e.g., SNV/fusion/isoform detection from long-read cDNA libraries) rather than bulk gene-expression profiles with subtype annotations
- No sample-level subtype or diagnosis metadata exists, since scores cannot be summarized per subtype without it

## Inputs

- Bulk RNA-seq expression matrix (e.g., 646 Beat AML profiles retrieved from vizome.org)
- Gene signature definitions (erythroid: GATA1, CA1, HBA1, HBB, ALAD; megakaryocytic: GATA2, ZFPM1, FLI1, NFE2, PF4)
- Per-sample WHO subtype and lesion-status annotations (CEBPA mutation, inv(3)/RPN1-EVI1)

## Outputs

- Per-sample erythroid and megakaryocytic signature scores (mean z-scored expression)
- Mean signature score per WHO subtype with subtype ranking
- Verification report comparing subtype-level score maxima to the expected lineage pattern

## How to apply

Retrieve the bulk RNA-seq expression matrix (for the Beat AML cohort, subset to the 646 profiles used in the paper). Z-score normalize expression of each signature gene across samples. Compute a per-sample erythroid score as the mean normalized expression of GATA1, CA1, HBA1, HBB and ALAD, and a megakaryocytic score as the mean normalized expression of GATA2, ZFPM1, FLI1, NFE2 and PF4. Annotate each sample with its WHO subtype and lesion status (e.g., CEBPA mutation, inv(3)/RPN1-EVI1). Finally, summarize mean signature scores per subtype and compare the ranking of subtypes against the expected pattern; the rationale is that lineage-defining signatures should peak in the corresponding malignancies (essential thrombocythemia, acute erythroid/megakaryoblastic leukemia) rather than being distributed uniformly.

## Examples

```
z <- t(scale(t(expr))); ery <- colMeans(z[c("GATA1","CA1","HBA1","HBB","ALAD"),]); mk <- colMeans(z[c("GATA2","ZFPM1","FLI1","NFE2","PF4"),]); aggregate(list(ery,mk), by=list(subtype), mean)
```

## Evaluation signals

- The number of scored samples matches the expected cohort size (646 profiles for the Beat AML subset)
- Highest erythroid and megakaryocytic scores occur in essential thrombocythemia and acute erythroid/megakaryoblastic leukemia, matching the paper-reported pattern
- CEBPA-mutated AML and inv(3)/RPN1-EVI1 cases show the characteristic score pattern reported in the paper
- Each per-sample score equals the arithmetic mean of the 5 z-scored signature genes (verify by recomputation)
- Subtype means are finite and orderable, with no missing-value genes in the z-scored matrix

## Limitations

- Scores based on gene-expression profiles alone may not unambiguously identify malignant versus physiologic populations, as noted for single-cell expression profiling
- Scores depend on the cohort used for z-score normalization; the workflow specifies z-scoring across the 646-profile Beat AML subset, so scores are not directly comparable to cohorts normalized differently

## Evidence

- [other] Z-score normalize expression of each signature gene and compute an erythroid score per sample as the mean normalized expression of GATA1, CA1, HBA1, HBB and ALAD.: [evidence span withheld — anchor status: not_found]
- [results] Signatures score highest in ET and acute erythroid/megakaryoblastic leukemia: [evidence span withheld — anchor status: not_found]
- [intro] Expression profiles alone often cannot identify malignant populations: "A crucial prerequisite of cancer single cell studies is the unambiguous identification of malignant versus physiologic populations, oftentimes impossible based on gene expression profiles alone."

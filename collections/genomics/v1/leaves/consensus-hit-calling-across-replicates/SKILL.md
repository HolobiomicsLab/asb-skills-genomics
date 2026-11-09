---
name: consensus-hit-calling-across-replicates
description: Use when multiple replicate/downsampling DE runs with per-method FDR
  values are available and a robust consensus hit set is needed before isoform co-expression
  or coDIU analysis.
when_to_use_negative:
- When the goal is differential expression of single isoforms per gene rather than
  isoform co-usage — the single-DE-isoform-gene exclusion would discard exactly those
  genes
- When only one DE run or replicate is available, since the ≥50% run-frequency consensus
  requires multiple downsampling runs
- When isoform expression proportions are not available, since the <10% minor-isoform
  filter cannot be computed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: acorde
  role: R package used to run the per-run DIU tests whose FDR values feed the consensus
    selection, and source of the minor-isoform filter
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Language in which the downsampling DE runs, run-frequency consensus computation,
    and filtering/export script are implemented
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:949474c891f0043707e99d343ae16bdcacac036d34b212a01c4e797bce987b53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# consensus-hit-calling-across-replicates

## Summary

Derive a consensus set of differential isoform usage (DIU) hits by requiring each isoform to be called significant (FDR<0.05 by at least one acorde method) in ≥50% of 50 downsampling DE runs, then apply acorde's minor-isoform filter and single-DE-isoform-gene exclusion to obtain a robust isoform set for downstream co-usage analysis.

## When to use

Use when multiple replicate/downsampling DE runs with per-method FDR values are available and a robust consensus hit set is needed before isoform co-expression or coDIU analysis.

## When NOT to use

- When the goal is differential expression of single isoforms per gene rather than isoform co-usage — the single-DE-isoform-gene exclusion would discard exactly those genes
- When only one DE run or replicate is available, since the ≥50% run-frequency consensus requires multiple downsampling runs
- When isoform expression proportions are not available, since the <10% minor-isoform filter cannot be computed

## Inputs

- Per-method FDR values from DIU testing for each of the 50 downsampling DE runs (acorde pipeline output, R)
- Isoform-to-gene mapping and isoform expression matrix for minor-isoform filtering

## Outputs

- consensus_diu_isoforms.csv containing 6794 DIU isoforms from 2696 genes
- Intermediate consensus set of 9393 isoforms from 4223 genes (pre minor-isoform filtering)

## How to apply

Load the DIU significance results (per-method FDR values) from the 50 downsampling DE runs produced with the acorde pipeline in R. For each isoform, compute the fraction of runs in which it is called significant (FDR<0.05) by at least one method and retain isoforms meeting the ≥50% run-frequency threshold, yielding the consensus set of 9393 isoforms from 4223 genes. Next apply acorde's minor-isoform filter, removing isoforms whose expression is <10% of their gene's total (removing 969 isoforms). Finally, exclude genes left with only one significant DE isoform, since no isoform co-usage pattern can be defined for them, obtaining the final 6794 isoforms from 2696 genes; export as consensus_diu_isoforms.csv. The rationale is that requiring replicate-level (run-level) reproducibility plus same-gene multiplicity guarantees that retained isoforms support isoform co-usage rather than single-isoform differential expression.

## Related tools

- **acorde** (R package used to run the per-run DIU tests whose FDR values feed the consensus selection, and source of the minor-isoform filter) — https://github.com/ConesaLab/acorde
- **R** (Language in which the downsampling DE runs, run-frequency consensus computation, and filtering/export script are implemented) — https://github.com/ConesaLab/acorde

## Examples

```
consensus <- isoforms[isoforms$fraction_significant_runs >= 0.5, ]; minor <- filter_min_isoform(consensus, min_proportion = 0.10); write.csv(minor, "consensus_diu_isoforms.csv", row.names = FALSE)
```

## Evaluation signals

- Exported table contains exactly 6794 isoforms from 2696 genes
- Minor-isoform step removed exactly 969 isoforms from the 9393-isoform consensus set
- Consensus set before minor-isoform filtering equals 9393 isoforms from 4223 genes
- Every retained isoform is significant (FDR<0.05 by ≥1 method) in ≥50% of the 50 downsampling runs
- No gene in the final table has fewer than two retained DE isoforms

## Limitations

- Consensus is tied to the specific 50 downsampling runs; different run counts or thresholds (≥50% frequency, <10% minor isoform) will change the final gene/isoform counts
- Genes with only one significant DE isoform are excluded by design, so single-isoform DIU events are not represented in the final set
- Functional annotation downstream depends on long read-defined isoform models and isoAnnotLite-compatible references, which may not exist for all transcriptomes

## Evidence

- [results] consensus frequency threshold: [evidence span withheld — anchor status: not_found]
- [abstract] coDIU definition: "By selecting same-gene isoforms between these clusters, we subsequently detect and characterize genes with co-differential isoform usage (coDIU) across cell types"
- [readme] acorde availability: "acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input"

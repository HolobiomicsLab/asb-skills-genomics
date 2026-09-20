---
name: fdr-threshold-based-isoform-selection
description: Use when per-isoform significance calls are available from multiple differential
  expression runs or multiple statistical methods, and a robust consensus DIU isoform
  set is needed before acorde co-usage clustering.
when_to_use_negative:
- Input is a single DE run or single-method FDR table — the ≥50% run-frequency consensus
  cannot be computed without multiple replicate runs
- The goal is gene-level differential expression rather than isoform-level DIU/coDIU
  selection prior to isoform co-usage clustering
- Genes of interest contain only one isoform, since the single-DE-isoform-gene filter
  would exclude them by definition
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: acorde
  role: R package implementing the DIU/coDIU analyses whose FDR outputs are pooled
    for consensus selection, and providing the minor-isoform filter
  repo: https://github.com/ConesaLab/acorde
- name: R
  role: Environment in which the downsampling DE runs, FDR thresholding, and filtering
    steps are executed
  repo: https://github.com/ConesaLab/acorde
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-29497-w
    title: acorde unravels functionally interpretable networks of isoform co-usage
      from single cell data
  source_content_hash: sha256:fc9405732e0634d930cbc3cb9c6678fe44e8efd88f1efde9b53c537729f2ca85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ConesaLab/acorde
  tools:
  - R
  - acorde
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# fdr-threshold-based-isoform-selection

## Summary

Selects differentially-used isoforms by pooling per-method FDR values across replicate DE runs (e.g. 50 downsampling runs), calling an isoform significant when FDR<0.05 by at least one method in ≥50% of runs, then applying acorde's minor-isoform and single-DE-isoform-gene filters to yield a robust consensus isoform set for downstream DIU/coDIU analysis.

## When to use

Use when per-isoform significance calls are available from multiple differential expression runs or multiple statistical methods, and a robust consensus DIU isoform set is needed before acorde co-usage clustering.

## When NOT to use

- Input is a single DE run or single-method FDR table — the ≥50% run-frequency consensus cannot be computed without multiple replicate runs
- The goal is gene-level differential expression rather than isoform-level DIU/coDIU selection prior to isoform co-usage clustering
- Genes of interest contain only one isoform, since the single-DE-isoform-gene filter would exclude them by definition

## Inputs

- DIU significance results with per-method FDR values from 50 downsampling DE runs (acorde pipeline output, R)
- Isoform-level expression matrix with isoform-to-gene relationships and cell type labels

## Outputs

- consensus_diu_isoforms.csv: final filtered isoform-gene table (6794 isoforms from 2696 genes)
- Intermediate consensus set of 9393 isoforms from 4223 genes after the ≥50% run-frequency threshold
- Counts of isoforms removed by the minor-isoform filter (969) and by the single-DE-isoform-gene exclusion

## How to apply

Load the DIU significance results (per-method FDR values) produced by the acorde R pipeline across the 50 downsampling DE runs. For each isoform, compute the fraction of runs in which it is called significant (FDR<0.05) by at least one method and retain isoforms meeting the ≥50% run-frequency threshold, giving the consensus set of 9393 isoforms from 4223 genes. Apply acorde's minor-isoform filter, removing isoforms whose expression is <10% of their gene's total (removes 969 isoforms here). Finally exclude genes left with only one significant DE isoform, since no isoform co-usage pattern can be defined for them, yielding the final 6794 isoforms from 2696 genes exported as consensus_diu_isoforms.csv. The rationale is that run-frequency consensus suppresses instability from downsampling, while the minor-isoform and single-DE-isoform-gene filters guarantee that every retained gene admits an interpretable isoform co-usage pattern.

## Related tools

- **acorde** (R package implementing the DIU/coDIU analyses whose FDR outputs are pooled for consensus selection, and providing the minor-isoform filter) — https://github.com/ConesaLab/acorde
- **R** (Environment in which the downsampling DE runs, FDR thresholding, and filtering steps are executed) — https://github.com/ConesaLab/acorde

## Examples

```
devtools::install_github("ConesaLab/acorde", build_vignettes = TRUE)
```

## Evaluation signals

- Exported consensus_diu_isoforms.csv contains exactly 6794 isoforms from 2696 genes
- The minor-isoform filter removes exactly 969 isoforms from the 9393-isoform consensus set
- Consensus set before minor-isoform filtering contains 9393 isoforms from 4223 genes
- Every retained isoform satisfies expression ≥10% of its gene's total expression
- No gene in the final table has fewer than two significant DE isoforms

## Limitations

- The consensus depends on the ≥50% run-frequency and FDR<0.05 cutoffs; different thresholds will change the retained isoform count
- Isoforms from genes left with a single significant DE isoform are discarded even if significant, since no isoform co-usage pattern can be defined for them
- Acorde requires an isoform-level quantification matrix plus isoform-to-gene relationships and cell identity labels; functional analyses further require a functionally-annotated transcriptome

## Evidence

- [methods] consensus threshold on run frequency: [evidence span withheld — anchor status: not_found]
- [results] final counts after filters: [evidence span withheld — anchor status: not_found]
- [methods] single-DE-isoform-gene filter rationale: [evidence span withheld — anchor status: not_found]
- [readme] acorde input requirements: "acorde contains the necessary functions and documentation to obtain a set of DIU and co-DIU genes using an single-cell, isoform-level expression matrix as input"
- [intro] acorde package availability: "This pipeline has been implemented in the R package acorde (https://github.com/ConesaLab/acorde)."

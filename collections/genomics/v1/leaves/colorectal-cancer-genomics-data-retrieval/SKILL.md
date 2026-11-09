---
name: colorectal-cancer-genomics-data-retrieval
description: Use when quantifying METTL3 (or a similar gene) CNV deletion frequency
  and CNV–RNA expression correlation in colorectal cancer from GutUDB's CNV and RNA-seq
  profiles, or when cross-checking the reported deletion frequency of 0.04.
when_to_use_negative:
- The target gene/disease sits outside GutUDB's scope of 56 intestinal diseases across
  six species and 58,970 curated genes (e.g., a non-intestinal cancer).
- Input is already the assembled mettl3_cnv_results.csv; recomputing deletion frequency
  and correlation would duplicate the workflow.
- The question requires m6A/miCLIP-seq modification data (GutUDB's 1908 RNA m6A sites)
  or drug-disease interactions rather than CNV-vs-expression linkage.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0199
tools:
- name: GutUDB
  role: Source database queried via Browse/Query/Download for METTL3 CNV genomics
    profiles of colorectal cancer and matching RNA-seq transcriptomics profiles
  repo: github.com/Ansanqi/GutUDB
- name: manual expert review
  role: Validation step confirming the computed deletion frequency (0.04) and weak
    CNV–RNA correlation match the reported finding
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/imt2.195
    title: 'GutUDB: A comprehensive multiomics database for intestinal diseases'
  source_content_hash: sha256:6d1dd2d96ef360d37ec90e0014e1d70bca2efa4040668520ec82cb2e736bc623
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GutUDB
  - manual expert review
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# colorectal-cancer-genomics-data-retrieval

## Summary

Retrieves METTL3 CNV genomics and matching RNA-seq transcriptomics profiles for colorectal cancer from GutUDB (54,131 curated CNVs; 137 RNA-seq data sets with 7.9 million transcriptional profiles) and computes the deletion frequency and CNV–RNA expression correlation to reproduce the reported 0.04 deletion frequency and weak correlation.

## When to use

Use when quantifying METTL3 (or a similar gene) CNV deletion frequency and CNV–RNA expression correlation in colorectal cancer from GutUDB's CNV and RNA-seq profiles, or when cross-checking the reported deletion frequency of 0.04.

## When NOT to use

- The target gene/disease sits outside GutUDB's scope of 56 intestinal diseases across six species and 58,970 curated genes (e.g., a non-intestinal cancer).
- Input is already the assembled mettl3_cnv_results.csv; recomputing deletion frequency and correlation would duplicate the workflow.
- The question requires m6A/miCLIP-seq modification data (GutUDB's 1908 RNA m6A sites) or drug-disease interactions rather than CNV-vs-expression linkage.

## Inputs

- METTL3-focused CNV genomics profile for colorectal cancer retrieved from GutUDB (subset of its 54,131 curated CNVs)
- Matching METTL3 RNA-seq transcriptomics profile from GutUDB's 137 RNA-seq data sets / 7.9 million transcriptional profiles
- Per-sample METTL3 CNV call state and RNA expression values across the colorectal cancer sample cohort

## Outputs

- mettl3_cnv_results.csv — small tabular result file containing the METTL3 CNV deletion frequency (~0.04) and the CNV–RNA expression correlation coefficient

## How to apply

Query GutUDB (github.com/Ansanqi/GutUDB) through its Browse/Query/Download functionality for the METTL3-focused CNV genomics profile of colorectal cancer and the matching METTL3 RNA-seq transcriptomics profile; GutUDB's genomics layer holds 54,131 CNVs and its transcriptomics layer holds 137 RNA-seq data sets with 7.9 million transcriptional profiles. Compute the CNV deletion frequency as the fraction of colorectal cancer samples carrying a deletion call at METTL3 and compare it against the reported value of 0.04. Compute the Pearson or Spearman correlation between METTL3 CNV state and RNA expression level and confirm the coefficient falls in the weak range, consistent with the reported weak CNV–RNA correlation despite significantly high RNA expression. Choose Pearson for linear CNV–expression scaling or Spearman when treating CNV state as ordinal, and record which method was used. Assemble the deletion frequency and correlation coefficient into a small tabular result file written as mettl3_cnv_results.csv and route the result through the manual expert review step specified in the workflow.

## Related tools

- **GutUDB** (Source database queried via Browse/Query/Download for METTL3 CNV genomics profiles of colorectal cancer and matching RNA-seq transcriptomics profiles) — github.com/Ansanqi/GutUDB
- **manual expert review** (Validation step confirming the computed deletion frequency (0.04) and weak CNV–RNA correlation match the reported finding)

## Examples

```
import pandas as pd; from scipy.stats import pearsonr; d=pd.read_csv('mettl3_cnv.csv'); pd.DataFrame({'del_freq':[(d.CNV=='deletion').mean()],'r':[pearsonr(d.CNV,d.RNA)[0]]}).to_csv('mettl3_cnv_results.csv',index=False)
```

## Evaluation signals

- Computed deletion frequency reproduces the reported METTL3 value of 0.04 in colorectal cancer.
- Correlation coefficient (Pearson or Spearman) lands in the weak range, matching the reported weak CNV–RNA expression correlation.
- mettl3_cnv_results.csv exists and contains both metrics (deletion frequency and correlation coefficient) in a small tabular schema.
- Sample set is restricted to GutUDB colorectal cancer profiles drawn from the 54,131 CNVs and 137 RNA-seq data sets, not the full 56-disease database.
- Output passes the manual expert review step named in the source workflow.

## Limitations

- Deletion frequency is very low (0.04), so small shifts in the CRC sample subset change the estimate; the exact cohort must be reported.
- The CNV–RNA correlation is weak, so the Pearson-vs-Spearman decision point can move the reported coefficient; the chosen method must be stated.
- The article abstract does not document per-gene deletion-call thresholds or which CRC data sets define the cohort, so exact reproduction depends on GutUDB's current profile release.
- Findings rest on manual expert review; no automated validation of the 0.04 value or the weak-correlation range is described.

## Evidence

- [abstract] GutUDB supplies the CNV genomics layer queried for METTL3 profiles: "GutUDB currently includes 72,248 single‐nucleotide polymorphisms (SNPs), 54,131 CNVs, 1097 structural variations of chromosome (SVs), and 92,888 mutated genes"
- [abstract] GutUDB supplies the matching RNA-seq transcriptomics layer for the CNV–expression correlation: [evidence span withheld — anchor status: not_found]
- [abstract] Retrieval is performed through GutUDB's Query and Download functionality: "four core functionalities—Browse, Query, Visualization, and Download—were integrated into GutUDB"
- [abstract] Database scope bounds the applicability of the CRC retrieval workflow: "GutUDB has accumulated approximately 9 million generated profiles from eight types of omics data sets, which encompasses 56 intestinal diseases across six species"
- [other] Reported result the workflow must reproduce: METTL3 deletion frequency 0.04 and weak CNV–RNA correlation in CRC: [evidence span withheld — anchor status: not_found]

---
name: source-data-spreadsheet-parsing
description: Use when a task requires reproducing or verifying quantitative results
  (coverage, fold-enrichment) published in Source Data files accompanying a Nature
  Communications article rather than re-running the underlying wet-lab or sequencing
  pipeline.
when_to_use_negative:
- The goal is to regenerate coverage/enrichment from raw nanopore reads — use the
  FUDGE wet-lab protocol plus the NanoFG pipeline instead of parsing pre-computed
  Source Data.
- The publication does not deposit Source Data files or the figures of interest lack
  per-sample numeric panels.
- The task needs fusion detection from sequencing data rather than extraction of already-reported
  summary metrics.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: NanoFG
  role: Origin of the published results; the Source Data files record the coverage
    and enrichment metrics produced by the NanoFG pipeline that this skill parses
    and validates
  repo: https://github.com/SdeBlank/NanoFG
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16641-7
    title: Partner independent fusion gene detection by multiplexed CRISPR-Cas9 enrichment
      and long read nanopore sequencing
  source_content_hash: sha256:1c379d7dddb067c82fc78601cf9251cda9b923320dc28ff8e4f5a30dfa7035b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SdeBlank/NanoFG
  tools:
  - FUDGE
  - NanoFG
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# source-data-spreadsheet-parsing

## Summary

Reproduces the per-sample coverage and enrichment metrics reported in the FUDGE paper (DOI 10.1038/s41467-020-16641-7) by programmatically parsing the published Source Data spreadsheets with pandas and validating the extracted numbers against the stated summary statistics (68x average breakpoint-spanning coverage, 665x average enrichment).

## When to use

Use when a task requires reproducing or verifying quantitative results (coverage, fold-enrichment) published in Source Data files accompanying a Nature Communications article rather than re-running the underlying wet-lab or sequencing pipeline.

## When NOT to use

- The goal is to regenerate coverage/enrichment from raw nanopore reads — use the FUDGE wet-lab protocol plus the NanoFG pipeline instead of parsing pre-computed Source Data.
- The publication does not deposit Source Data files or the figures of interest lack per-sample numeric panels.
- The task needs fusion detection from sequencing data rather than extraction of already-reported summary metrics.

## Inputs

- Source Data spreadsheet files (.xlsx) downloaded from https://www.nature.com/articles/s41467-020-16641-7 for Figs. 3-5
- Reported reference values from the article text: 68x average breakpoint-spanning coverage, 665x average breakpoint-site enrichment, per-sample on-target enrichment list

## Outputs

- enrichment_metrics.csv with one row per sample/fusion and columns for coverage and enrichment
- Validation report showing computed averages (expected: 68x coverage, 665x enrichment) matching the published values

## How to apply

Download the Source Data files linked to Figs. 3-5 from the article page at https://www.nature.com/articles/s41467-020-16641-7. Load each spreadsheet with pandas and extract the breakpoint-spanning coverage and on-target enrichment columns for each sample. Compute the average breakpoint-spanning coverage across samples and confirm it equals the reported 68x, then compute the average breakpoint-site enrichment and confirm it equals the reported 665x fold enrichment. Extract the per-sample on-target enrichments (e.g., 498x ES1, 930x RH1, 611x CML, 347x BL, 679x ALL1, 3492x ALL2; 342x A4573, 443x CHP-100, 735x HS-SYII; 938x and 313x blinded diagnostic samples; 349x multiplexed run) and fusion-specific enrichments (118x, 121x, 68x for cell lines; 61x-3382x for tumors). Write all metrics to enrichment_metrics.csv with one row per sample/fusion and columns for coverage and enrichment so the values are diffable against the paper.

## Related tools

- **NanoFG** (Origin of the published results; the Source Data files record the coverage and enrichment metrics produced by the NanoFG pipeline that this skill parses and validates) — https://github.com/SdeBlank/NanoFG

## Examples

```
import pandas as pd; df = pd.read_excel('SourceData_Fig3.xlsx'); df[['sample','coverage','enrichment']].to_csv('enrichment_metrics.csv', index=False)
```

## Evaluation signals

- Computed average breakpoint-spanning coverage equals the reported 68x
- Computed average breakpoint-site enrichment equals the reported 665x
- Each per-sample on-target enrichment matches the published figure (e.g., 498x ES1, 930x RH1, 611x CML, 347x BL, 679x ALL1, 3492x ALL2)
- Cell-line fusion-specific enrichments match 118x, 121x, 68x and tumor values fall in the 61x-3382x reported range
- enrichment_metrics.csv schema contains one row per sample/fusion with coverage and enrichment columns populated and non-null

## Limitations

- Reproduces reported numbers only; it cannot validate the underlying sequencing runs or FUDGE enrichments themselves.
- Supplementary Data 1 is referenced in the article but its content was not provided, so some referenced inputs may be unavailable.
- Values for distal breakpoints and low-throughput runs (Supplementary Fig. 8) show decreased on-target coverage and are not in the main-text Source Data.

## Evidence

- [other] per-sample on-target enrichments: [evidence span withheld — anchor status: not_found]
- [intro] reported averages: "we achieve an average breakpoint-spanning coverage of 68x — resulting in an average enrichment of 665x"
- [abstract] abstract enrichment finding: "We observe on-average 665 fold breakpoint-site enrichment"
- [other] validation target: [evidence span withheld — anchor status: not_found]

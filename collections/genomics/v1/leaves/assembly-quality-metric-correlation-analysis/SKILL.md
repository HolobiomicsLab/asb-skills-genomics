---
name: assembly-quality-metric-correlation-analysis
description: Use when you have a per-assembly metrics table (AQI scores plus LAI,
  QV, BUSCO, N50) for multiple assemblies and need to test whether CRAQ's R-AQI/S-AQI
  measure the same quality dimensions as existing metrics.
when_to_use_negative:
- Input is raw CRAQ output (CRE/CSE breakpoints, CRHs/CSHs coordinates) rather than
  a per-assembly metrics table — this skill analyzes aggregate index scores, not error
  coordinates.
- The goal is single-nucleotide detection of assembly errors in one draft assembly
  (run CRAQ directly) rather than cross-assembly benchmarking of quality metrics.
- Assemblies are polyploid — the article notes CRAQ's evaluation may be inaccurate
  for higher-ploidy genomes, undermining the AQI scores being correlated.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0092
tools:
- name: CRAQ
  role: Source of the R-AQI and S-AQI scores being correlated; scores are derived
    from normalized Clip-based Regional Error (CRE) and Structural Error (CSE) counts
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: LTR Assembly Index (LAI)
  role: Existing reference-free metric compared against R-AQI/S-AQI; expected to show
    the best correlation (r2 = 0.419 with R-AQI)
- name: BUSCO
  role: Existing completeness metric included in the correlation comparison; shows
    poor correlation with S-AQI
- name: Merqury
  role: k-mer-based accuracy evaluation family (with QV) included among the existing
    metrics compared against S-AQI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:d959cdcb5af3bbe7b558202b02bec0d68296a43b55740d2b5d0644eb9534a42c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - BUSCO
  - CRAQ
  - LTR Assembly Index (LAI)
  - Merqury
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# assembly-quality-metric-correlation-analysis

## Summary

Quantifies how CRAQ's clipping-derived assembly quality indexes (R-AQI, S-AQI) agree with established assembly metrics (LAI, QV, BUSCO completeness, N50) across 40 public genome assemblies by computing Pearson correlations (reported as r2) and rendering scatter panels, revealing which quality dimensions each metric uniquely captures.

## When to use

Use when you have a per-assembly metrics table (AQI scores plus LAI, QV, BUSCO, N50) for multiple assemblies and need to test whether CRAQ's R-AQI/S-AQI measure the same quality dimensions as existing metrics.

## When NOT to use

- Input is raw CRAQ output (CRE/CSE breakpoints, CRHs/CSHs coordinates) rather than a per-assembly metrics table — this skill analyzes aggregate index scores, not error coordinates.
- The goal is single-nucleotide detection of assembly errors in one draft assembly (run CRAQ directly) rather than cross-assembly benchmarking of quality metrics.
- Assemblies are polyploid — the article notes CRAQ's evaluation may be inaccurate for higher-ploidy genomes, undermining the AQI scores being correlated.

## Inputs

- Supplementary Data 1 metrics table for the 40 publicly-available assemblies: per-assembly R-AQI, S-AQI, LAI, QV, BUSCO completeness, and N50 contig length (Mbp)
- AQI scores computed by CRAQ (normalized CRE/CSE counts transformed into R-AQI and S-AQI)

## Outputs

- aqi_metric_correlations.csv: correlation table of r2 values for R-AQI and S-AQI vs. LAI, QV, BUSCO, and N50
- Scatter panel PNGs: (a) each metric vs. R-AQI and (b) each metric vs. S-AQI, styled after Supplementary Fig. 11

## How to apply

Load Supplementary Data 1 — the table of sequencing/assembly sources giving R-AQI, S-AQI, LAI, QV, BUSCO completeness, and N50 contig length (in Mbp) for the 40 publicly-available assemblies — into a pandas DataFrame. Compute Pearson correlation coefficients (reported as r2) between R-AQI and each of LAI, QV, BUSCO, and N50 with scipy.stats.pearsonr, then repeat the computation for S-AQI against the same four metrics. Generate scatter plots of each metric against R-AQI (panel a) and S-AQI (panel b), styled after Supplementary Fig. 11, and write the correlation table to aqi_metric_correlations.csv with scatter panels as PNG files. Interpretation follows the paper's framing: LAI is expected to correlate best with R-AQI (r2 ≈ 0.419), while QV, BUSCO, and N50 correlate poorly with S-AQI because S-AQI, derived from normalized Clip-based Structural Error (CSE) counts, captures structural quality that existing completeness/continuity metrics do not measure.

## Related tools

- **CRAQ** (Source of the R-AQI and S-AQI scores being correlated; scores are derived from normalized Clip-based Regional Error (CRE) and Structural Error (CSE) counts) — https://github.com/JiaoLaboratory/CRAQ
- **LTR Assembly Index (LAI)** (Existing reference-free metric compared against R-AQI/S-AQI; expected to show the best correlation (r2 = 0.419 with R-AQI))
- **BUSCO** (Existing completeness metric included in the correlation comparison; shows poor correlation with S-AQI)
- **Merqury** (k-mer-based accuracy evaluation family (with QV) included among the existing metrics compared against S-AQI)

## Examples

```
from scipy.stats import pearsonr; r2 = pearsonr(df['R-AQI'], df['LAI'])[0]**2
```

## Evaluation signals

- LAI-vs-R-AQI Pearson correlation reproduces the reported r2 = 0.419 within reasonable numerical tolerance
- QV, BUSCO, and N50 all exhibit poor correlation with S-AQI, consistent with S-AQI capturing structural quality not reflected in other metrics
- aqi_metric_correlations.csv contains r2 values for all 8 metric–index pairs (4 metrics × R-AQI/S-AQI) over the full set of 40 assemblies
- Scatter panels match the layout of Supplementary Fig. 11: panel a for R-AQI, panel b for S-AQI, one scatter per metric

## Limitations

- LAI explains only ~42% of R-AQI variance (r2 = 0.419); the majority of variance in AQI scores is not explained by any existing metric, so correlations are only partially informative.
- Key benchmarking results supporting the comparison are deferred to supplementary materials (Table 1, Supplementary Fig. 13, Supplementary Data 1), which are not in the main text.
- The comparison is restricted to 40 publicly-available assemblies with sequencing/assembly sources summarized only in Supplementary Data 1.

## Evidence

- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [intro] A reference-free program, long terminal repeat (LTR) Assembly Index (LAI)21, gauges assembly quality by estimating the percentage of fully-assembled LTR retroelements: "A reference-free program, long terminal repeat (LTR) Assembly Index (LAI)21, gauges assembly quality by estimating the percentage of fully-assembled LTR retroelements"
- [discussion] Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1.: "Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1."
- [readme] It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly quality (R-AQI & S-AQI).: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly"

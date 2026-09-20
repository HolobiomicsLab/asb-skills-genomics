---
name: genome-assembly-benchmarking-statistics
description: Use when a new genome assembly quality index (e.g., R-AQI/S-AQI) must
  be compared against existing metrics (LAI, QV, BUSCO, N50) over a panel of assemblies
  to prove redundancy or complementary information.
when_to_use_negative:
- Assemblies lack the raw NGS short-reads and SMS long-reads needed to run CRAQ, so
  no R-AQI/S-AQI scores exist to correlate
- Genome assemblies of species with higher ploidy levels (polyploids), where CRAQ's
  evaluation may be less accurate than the benchmarked diploid cases
- The goal is de novo error detection or correction (CRE/CSE breakpoints, chimeric
  contig splitting) rather than aggregate metric benchmarking
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: CRAQ
  role: Supplies the R-AQI and S-AQI scores being benchmarked, derived from normalized
    CRE and CSE error counts
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: LTR Assembly Index (LAI)
  role: Reference-free comparison metric gauging assembly quality via percentage of
    fully-assembled LTR retroelements
- name: BUSCO
  role: Provides genome completeness values used as the comparison metric for correlation
    with R-AQI/S-AQI
- name: Merqury
  role: k-mer based approach supplying the QV accuracy values used in the correlation
    analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:08f9889220850802bba1a5e0c2dd3d0b54f1ef35420ca756a30de67b173d0fdb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - BUSCO
  - CRAQ
  - LTR Assembly Index (LAI)
  - Merqury
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# genome-assembly-benchmarking-statistics

## Summary

Benchmarks CRAQ's R-AQI and S-AQI assembly-quality indexes against established metrics (LAI, QV, BUSCO completeness, N50) by computing Pearson correlations (r2) across 40 publicly-available genome assemblies, demonstrating which quality dimensions each metric uniquely captures.

## When to use

Use when a new genome assembly quality index (e.g., R-AQI/S-AQI) must be compared against existing metrics (LAI, QV, BUSCO, N50) over a panel of assemblies to prove redundancy or complementary information.

## When NOT to use

- Assemblies lack the raw NGS short-reads and SMS long-reads needed to run CRAQ, so no R-AQI/S-AQI scores exist to correlate
- Genome assemblies of species with higher ploidy levels (polyploids), where CRAQ's evaluation may be less accurate than the benchmarked diploid cases
- The goal is de novo error detection or correction (CRE/CSE breakpoints, chimeric contig splitting) rather than aggregate metric benchmarking

## Inputs

- Supplementary Data 1 metrics table with per-assembly R-AQI, S-AQI, LAI, QV, BUSCO completeness, and N50 contig length (Mbp) for 40 publicly-available assemblies
- CRAQ AQI outputs (R-AQI and S-AQI scores derived from normalized CRE/CSE counts) for each assembly
- Existing per-assembly metric values: LAI, Merqury QV, BUSCO completeness, N50

## Outputs

- aqi_metric_correlations.csv correlation table (R-AQI and S-AQI vs. LAI, QV, BUSCO, N50)
- Scatter panel PNG files styled after Supplementary Fig. 11 (panel a: metrics vs. R-AQI; panel b: metrics vs. S-AQI)

## How to apply

Load the Supplementary Data 1 metrics table (R-AQI, S-AQI, LAI, QV, BUSCO completeness, N50 contig length in Mbp) for the 40 publicly-available assemblies into a pandas DataFrame. Compute Pearson correlation coefficients, reported as r2, between R-AQI and each of LAI, QV, BUSCO, and N50 using pandas and scipy.stats.pearsonr, then repeat for S-AQI against the same four metrics. The rationale is that CRAQ transforms normalized CRE/CSE error counts into R-AQI and S-AQI scores capturing regional vs. structural quality, so correlations reveal overlap with LTR-retroelement-based (LAI), k-mer-based (QV via Merqury), completeness (BUSCO), and contiguity (N50) measures. Generate scatter plots of each metric against R-AQI (panel a) and S-AQI (panel b) styled after Supplementary Fig. 11 with matplotlib, and write the correlation table to aqi_metric_correlations.csv. Validate by confirming the LAI-vs-R-AQI correlation reproduces the reported r2 = 0.419 within numerical tolerance.

## Related tools

- **CRAQ** (Supplies the R-AQI and S-AQI scores being benchmarked, derived from normalized CRE and CSE error counts) — https://github.com/JiaoLaboratory/CRAQ
- **LTR Assembly Index (LAI)** (Reference-free comparison metric gauging assembly quality via percentage of fully-assembled LTR retroelements)
- **BUSCO** (Provides genome completeness values used as the comparison metric for correlation with R-AQI/S-AQI)
- **Merqury** (k-mer based approach supplying the QV accuracy values used in the correlation analysis)

## Examples

```
python -c "import pandas as pd; from scipy.stats import pearsonr; df=pd.read_csv('Supp_Data_1.tsv', sep='\t'); print(pearsonr(df['LAI'], df['R-AQI'])[0]**2)"
```

## Evaluation signals

- LAI-vs-R-AQI Pearson r2 reproduces the reported value of 0.419 within reasonable numerical tolerance
- QV, BUSCO, and N50 all show poor correlations with S-AQI, consistent with the finding that S-AQI captures structural quality not measured by existing metrics
- Correlation table aqi_metric_correlations.csv contains entries for both AQI scores against all four comparison metrics (8 correlations total)
- Scatter panels include all 40 publicly-available assemblies and visually match the styling of Supplementary Fig. 11
- N50 values are confirmed to be in contig-length Mbp units before correlation to avoid unit-mismatched r2 values

## Limitations

- CRAQ's accuracy (and hence the AQI scores being benchmarked) may be lower for species with higher ploidy levels than for the benchmarked cases
- Key benchmarking results are deferred to supplementary materials (Table 1, Supplementary Fig. 13, Supplementary Data 1), so per-assembly values must be sourced from the supplement
- Correlations are computed over only 40 assemblies, so r2 estimates (e.g., LAI-vs-R-AQI 0.419) carry sampling uncertainty and no per-correlation confidence intervals are specified

## Evidence

- [readme] overall AQI metrics in regional and structural assembly quality (R-AQI & S-AQI): "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly"
- [intro] A reference-free program, long terminal repeat (LTR) Assembly Index (LAI)21, gauges assembly quality: "A reference-free program, long terminal repeat (LTR) Assembly Index (LAI)21, gauges assembly quality by estimating the percentage of fully-assembled LTR retroelements"
- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [discussion] Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1.: "Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1."

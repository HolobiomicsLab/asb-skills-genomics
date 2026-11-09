---
name: scatter-plot-visualization-of-quality-metrics
description: Use when CRAQ R-AQI/S-AQI scores must be compared against LAI, QV, BUSCO,
  or N50 across multiple assemblies to test whether AQI captures assembly quality
  not measured by existing metrics.
when_to_use_negative:
- The goal is per-assembly error localization (CRH/CSH coordinates, misjoin splitting)
  rather than cross-assembly metric comparison — use CRAQ's error-output workflow
  instead
- The input is raw read alignments (SAM/BAM) or clipped-read extractions rather than
  a summary metrics table with both AQI and comparison metrics per assembly
- The assemblies are polyploid, where the article states CRAQ evaluation may not be
  as accurate as the benchmarked diploid cases
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0091
tools:
- name: CRAQ
  role: Produces the R-AQI and S-AQI scores (from normalized CRE/CSE counts) that
    are the dependent variables in the correlation scatter panels
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: LAI
  role: Supplies the LTR Assembly Index values used as the comparison metric that
    best correlates with R-AQI (r2 = 0.419)
- name: BUSCO
  role: Supplies completeness values used as a comparison metric showing poor correlation
    with S-AQI
- name: Merqury
  role: k-mer based accuracy evaluation listed among the existing metrics context
    against which AQI is contrasted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:bda2685d5645c5bfb92a4732c2bc36d9517cf43bcc292bd99adf7fba3619470e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - BUSCO
  - CRAQ
  - LAI
  - Merqury
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# scatter-plot-visualization-of-quality-metrics

## Summary

Computes Pearson correlations (r2) between CRAQ's R-AQI/S-AQI scores and existing assembly quality metrics (LAI, QV, BUSCO completeness, N50) across a set of genome assemblies, then renders scatter panels showing how the new AQI metrics relate to established ones.

## When to use

Use when CRAQ R-AQI/S-AQI scores must be compared against LAI, QV, BUSCO, or N50 across multiple assemblies to test whether AQI captures assembly quality not measured by existing metrics.

## When NOT to use

- The goal is per-assembly error localization (CRH/CSH coordinates, misjoin splitting) rather than cross-assembly metric comparison — use CRAQ's error-output workflow instead
- The input is raw read alignments (SAM/BAM) or clipped-read extractions rather than a summary metrics table with both AQI and comparison metrics per assembly
- The assemblies are polyploid, where the article states CRAQ evaluation may not be as accurate as the benchmarked diploid cases

## Inputs

- Supplementary Data 1 metrics table with R-AQI, S-AQI, LAI, QV, BUSCO completeness, and N50 (Mbp) for 40 publicly-available genome assemblies
- CRAQ output AQI scores (R-AQI, S-AQI) derived from normalized CRE and CSE counts per assembly

## Outputs

- aqi_metric_correlations.csv: Pearson r2 for R-AQI vs LAI/QV/BUSCO/N50 and S-AQI vs LAI/QV/BUSCO/N50
- PNG scatter panels (panel a: metrics vs R-AQI; panel b: metrics vs S-AQI) styled after Supplementary Fig. 11

## How to apply

Load the per-assembly metrics table from Supplementary Data 1 — R-AQI, S-AQI, LAI, QV, BUSCO completeness, and N50 contig length in Mbp for the 40 publicly-available assemblies — into a pandas DataFrame. Use scipy.stats.pearsonr (r reported as r2) to correlate R-AQI with each of LAI, QV, BUSCO, and N50, then repeat for S-AQI; the decision point is the reported result that LAI best correlates with R-AQI (r2 = 0.419) while QV, BUSCO, and N50 correlate poorly with S-AQI, indicating S-AQI captures structural quality not measured by existing metrics. Render scatter panels styled after Supplementary Fig. 11: panel (a) plots each metric against R-AQI, panel (b) against S-AQI, using matplotlib. Write the correlation table to aqi_metric_correlations.csv and the panels to PNG files. Rationale: R-AQI and S-AQI derive from normalized CRE/CSE counts, so correlation against LAI (fully-assembled LTR retroelements) and continuity/accuracy metrics contextualizes what the clipping-based indexes add.

## Related tools

- **CRAQ** (Produces the R-AQI and S-AQI scores (from normalized CRE/CSE counts) that are the dependent variables in the correlation scatter panels) — https://github.com/JiaoLaboratory/CRAQ
- **LAI** (Supplies the LTR Assembly Index values used as the comparison metric that best correlates with R-AQI (r2 = 0.419))
- **BUSCO** (Supplies completeness values used as a comparison metric showing poor correlation with S-AQI)
- **Merqury** (k-mer based accuracy evaluation listed among the existing metrics context against which AQI is contrasted)

## Examples

```
from scipy.stats import pearsonr; import pandas as pd; df = pd.read_csv('supplementary_data_1_metrics.csv'); r2_lai = pearsonr(df['R-AQI'], df['LAI'])[0]**2
```

## Evaluation signals

- The LAI-vs-R-AQI Pearson r2 matches the reported 0.419 within numerical tolerance
- All metrics other than LAI (QV, BUSCO, N50) show poor correlation with S-AQI, consistent with S-AQI capturing structural quality not reflected in other metrics
- aqi_metric_correlations.csv contains 8 correlation values (4 metrics x 2 AQI scores) with r2 in [0, 1]
- Each scatter panel renders one point per assembly (40 points) for each of the four metrics
- AQI values fall within the paper's classification bands (AQI > 90 reference quality; 80-90 high quality; 60-80 draft; < 60 low quality), sanity-checking the plotted axis ranges

## Limitations

- Correlation support data (sequencing/assembly sources) is only in Supplementary Data 1, not the main text, so the input table must be obtained from supplementary materials
- CRAQ accuracy — and hence AQI values being correlated — is not established for polyploid genomes; the article notes evaluation 'may not be as accurate as the benchmarked cases presented here'
- Key benchmarking results (Table 1, Supplementary Fig. 13, Supplementary Data 1) are deferred to supplementary materials, so exact per-assembly values may be hard to reproduce outside the SI

## Evidence

- [abstract] Error counts are transformed into corresponding assembly evaluation indexes to reflect the assembly quality at single-nucleotide resolution: "Error counts are transformed into corresponding assembly evaluation indexes to reflect the assembly quality at single-nucleotide resolution"
- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [discussion] Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1.: "Sources of the sequencing and assembly data used in this study are summarized in Supplementary Data 1."
- [readme] It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly quality (R-AQI & S-AQI).: "as well as overall AQI metrics in regional and structural assembly quality (R-AQI & S-AQI)"

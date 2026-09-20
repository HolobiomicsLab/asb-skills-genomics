---
name: aqi-score-interpretation
description: Use when a CRAQ run has produced AQI, R-AQI, or S-AQI scores for one
  or more assemblies and you need to assign a quality class, compare against LAI/QV/BUSCO/N50,
  or decide whether to split misjoined scaffolds.
when_to_use_negative:
- 'Assessing assemblies of polyploid species: the article states ''evaluation of genome
  assemblies for species with higher ploidy levels may not be as accurate as the benchmarked
  cases presented here'''
- Judging genome completeness alone (use BUSCO) — S-AQI reflects structural quality
  that QV, BUSCO, and N50 do not capture, so a high S-AQI does not certify completeness
- Input is raw SAM/BAM read-mapping data rather than computed AQI scores — run the
  CRAQ pipeline (clip extraction, CRE/CSE classification, normalization) first
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Generates the AQI, R-AQI, and S-AQI scores being interpreted, from normalized
    CRE (regional) and CSE (structural) error counts
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads and SMS long reads back to the assembly ('-ax sr', '-ax
    map-hifi/map-pb/map-ont'); the clipping signal in these alignments is the basis
    of the AQI metrics
  repo: https://github.com/lh3/minimap2
- name: LAI (LTR Assembly Index)
  role: External reference-free metric used to cross-validate R-AQI interpretation
    (best correlation, r2 = 0.419)
- name: BUSCO
  role: Completeness metric compared against R-AQI/S-AQI to demonstrate S-AQI captures
    structural quality not measured by existing metrics
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:24f51929d163804c79782713d8801f4a6547a3c0020593f8d75b9aa99fe8c238
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - BUSCO
  - CRAQ
  - LAI (LTR Assembly Index)
  - Merqury
  - Minimap2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# aqi-score-interpretation

## Summary

Interprets CRAQ's assembly quality indicators, where normalized Clip-based Regional Error (CRE) counts become R-AQI and normalized Clip-based Structural Error (CSE) counts become S-AQI, and maps scores to published quality classes (AQI > 90 reference quality; 80-90 high; 60-80 draft; < 60 low). It also frames R-AQI/S-AQI against existing metrics (LAI, QV, BUSCO, N50), since S-AQI captures structural quality these metrics do not.

## When to use

Use when a CRAQ run has produced AQI, R-AQI, or S-AQI scores for one or more assemblies and you need to assign a quality class, compare against LAI/QV/BUSCO/N50, or decide whether to split misjoined scaffolds.

## When NOT to use

- Assessing assemblies of polyploid species: the article states 'evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'
- Judging genome completeness alone (use BUSCO) — S-AQI reflects structural quality that QV, BUSCO, and N50 do not capture, so a high S-AQI does not certify completeness
- Input is raw SAM/BAM read-mapping data rather than computed AQI scores — run the CRAQ pipeline (clip extraction, CRE/CSE classification, normalization) first

## Inputs

- CRAQ output metrics table with R-AQI, S-AQI (or overall AQI) per assembly, derived from normalized CRE and CSE counts
- Supplementary Data 1 metrics for the 40 publicly-available assemblies (R-AQI, S-AQI, LAI, QV, BUSCO completeness, N50 contig length in Mbp)
- CRAQ CSH/CSH coordinate files listing structural heterozygous variants and Clip-based Structural Error breakpoints

## Outputs

- Quality classification label per assembly (reference / high / draft / low) from the AQI thresholds
- Correlation table of R-AQI and S-AQI vs LAI, QV, BUSCO, and N50 (e.g. aqi_metric_correlations.csv)
- Scatter panels of each metric against R-AQI (panel a) and S-AQI (panel b), styled after Supplementary Fig. 11

## How to apply

Read the CRAQ output metrics (R-AQI, S-AQI, or overall AQI) and recall their provenance: CRE and CSE counts from clipped-alignments of Minimap2-mapped NGS short reads and SMS long reads (mapped with '-ax sr' and '-ax map-hifi/map-pb/map-ont', filtered with '-F 1796 -q 20') are normalized into R-AQI and S-AQI scores, respectively. Classify each assembly with the published thresholds: AQI > 90 reference quality; AQI 80-90 high quality; AQI 60-80 draft quality; AQI < 60 low quality. Interpret R-AQI as regional (small-scale error) quality and S-AQI as structural (misjoin) quality as independent dimensions: LAI is the best-correlating external metric for R-AQI (r2 = 0.419 across 40 public assemblies), while QV, BUSCO, and N50 correlate poorly with S-AQI, so low S-AQI will not be flagged by completeness or consensus metrics. To quantify this, load the Supplementary Data 1 metrics table into pandas, compute Pearson correlations (reported as r2) between R-AQI/S-AQI and each of LAI, QV, BUSCO, and N50 with scipy.stats.pearsonr, and validate against the reported LAI-vs-R-AQI r2 = 0.419. Treat low S-AQI (high CSE burden) as a signal that misjoined regions should be split at CSE breakpoints for further scaffolding with Hi-C or optical maps, as done for the A. oxysepala CRAQ-corrected contigs.

## Related tools

- **CRAQ** (Generates the AQI, R-AQI, and S-AQI scores being interpreted, from normalized CRE (regional) and CSE (structural) error counts) — https://github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads and SMS long reads back to the assembly ('-ax sr', '-ax map-hifi/map-pb/map-ont'); the clipping signal in these alignments is the basis of the AQI metrics) — https://github.com/lh3/minimap2
- **LAI (LTR Assembly Index)** (External reference-free metric used to cross-validate R-AQI interpretation (best correlation, r2 = 0.419))
- **BUSCO** (Completeness metric compared against R-AQI/S-AQI to demonstrate S-AQI captures structural quality not measured by existing metrics)

## Examples

```
from scipy.stats import pearsonr; r2 = pearsonr(df['R-AQI'], df['LAI'])[0]**2  # expect ~0.419 across the 40 assemblies
```

## Evaluation signals

- LAI-vs-R-AQI Pearson r2 reproduces the reported value 0.419 (within numerical tolerance) when computed over the 40 public assemblies
- All other external metrics (QV, BUSCO, N50) show poor correlation with S-AQI, matching the paper's finding that S-AQI captures structural quality not measured by existing metrics
- Threshold check: a score of 85 maps to 'high quality', 95 to 'reference quality', 70 to 'draft quality', and 50 to 'low quality' per the published AQI classes
- R-AQI and S-AQI values are traceable to normalized CRE and CSE counts respectively, not used interchangeably
- Output correlation table matches the expected schema (metrics × R-AQI/S-AQI r2 values) and scatter panels render for both panels a and b

## Limitations

- CRAQ accuracy for polyploid genomes is not established; the benchmarked cases are diploid (evaluation 'may not be as accurate' for higher ploidy)
- Ambiguous heterozygous-or-error regions (coverage above h and below f) are reported without definitive classification, so AQI-relevant calls can be ambiguous
- The error breakpoint ratio can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating the error counts feeding R-AQI/S-AQI
- Key benchmarking results (Table 1, Supplementary Fig. 13, Supplementary Data 1) are deferred to supplementary materials, limiting main-text verification

## Evidence

- [abstract] AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [readme] It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs),: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs),"

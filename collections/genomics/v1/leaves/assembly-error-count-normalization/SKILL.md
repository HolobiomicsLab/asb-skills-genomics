---
name: assembly-error-count-normalization
description: Use when you have CRAQ output files (locER_out/out_final.CRE.bed, strER_out/out_final.CSE.bed)
  plus assembly size and need to compute normalized error densities, R-AQI/S-AQI scores,
  and reference/high/draft/low quality classes.
when_to_use_negative:
- Input error coordinates have not yet been classified as CREs/CSEs — run CRAQ's clipping-based
  breakpoint detection and heterozygous filtering (MAPQ >= 20, '-F 1796 -q 20') first
- Regions flagged as heterozygous variants (CRHs/CSHs) or ambiguous heterozygous-or-error
  regions should be excluded rather than scored as errors
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0196
tools:
- name: CRAQ
  role: Provides the CRE/CSE breakpoint files and implements the normalization and
    AQI transformation pipeline
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Perl
  role: Scripting language required to run CRAQ's normalization/scoring scripts (version
    >= 5)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:10f1c2a9b98fcc69a773d23856878dcd19bc77169c4e1d26b3269d4dd87b1f5f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Perl
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# assembly-error-count-normalization

## Summary

Implements CRAQ's transformation of raw CRE/CSE assembly-error counts into normalized sliding-window values (Nw = sum of 1/i) and Assembly Quality Index scores (AQI = 100 * e^(-0.1*N/L)), producing R-AQI and S-AQI with published quality-class thresholds.

## When to use

Use when you have CRAQ output files (locER_out/out_final.CRE.bed, strER_out/out_final.CSE.bed) plus assembly size and need to compute normalized error densities, R-AQI/S-AQI scores, and reference/high/draft/low quality classes.

## When NOT to use

- Input error coordinates have not yet been classified as CREs/CSEs — run CRAQ's clipping-based breakpoint detection and heterozygous filtering (MAPQ >= 20, '-F 1796 -q 20') first
- Regions flagged as heterozygous variants (CRHs/CSHs) or ambiguous heterozygous-or-error regions should be excluded rather than scored as errors

## Inputs

- CRE breakpoint BED file (locER_out/out_final.CRE.bed) from CRAQ clipping-error detection
- CSE breakpoint BED file (strER_out/out_final.CSE.bed) from CRAQ clipping-error detection
- Total assembly size in megabases (from the draft assembly FASTA)

## Outputs

- aqi_scores.csv with per-fragment R-AQI, S-AQI, and quality-class columns
- Whole-assembly R-AQI and S-AQI summary values with quality classes (reference/high/draft/low)

## How to apply

Parse the CRE breakpoint file (locER_out/out_final.CRE.bed) and CSE breakpoint file (strER_out/out_final.CSE.bed) and read the total assembly size in megabases. Partition CRE and CSE counts separately into sliding windows of width 0.0001 * total assembly size and compute Nw = sum_{i=1..m} (1/i) for each window with m errors (e.g., three errors give Nw = 1/1 + 1/2 + 1/3 = 1.83). Penalize CSE-window Nw values at 10 times the rate of CRE Nw values before aggregation. Compute AQI = 100 * e^(-0.1*N/L), where N is the cumulative normalized CRE count (for R-AQI) or penalized CSE count (for S-AQI) and L is assembly length in Mb, reporting per-fragment regional scores and whole-assembly summary values. Classify each score with the published thresholds: AQI > 90 reference quality, 80-90 high quality, 60-80 draft quality, AQI < 60 low quality, and write per-fragment R-AQI, S-AQI, and quality class to aqi_scores.csv.

## Related tools

- **CRAQ** (Provides the CRE/CSE breakpoint files and implements the normalization and AQI transformation pipeline) — https://github.com/JiaoLaboratory/CRAQ
- **Perl** (Scripting language required to run CRAQ's normalization/scoring scripts (version >= 5))

## Examples

```
perl scripts/aqi_scoring.pl locER_out/out_final.CRE.bed strER_out/out_final.CSE.bed assembly.fa > aqi_scores.csv
```

## Evaluation signals

- Normalization reproduces the paper's worked example: a window block containing three errors yields Nw = 1/1 + 1/2 + 1/3 = 1.83
- All output AQI values fall in the valid range (0, 100]
- Every AQI value has a non-empty quality-class label matching the thresholds (AQI > 90 reference; 80-90 high; 60-80 draft; < 60 low)
- S-AQI is strictly lower than R-AQI for assemblies with structural errors, since CSE Nw values are penalized 10x before aggregation
- aqi_scores.csv contains R-AQI, S-AQI, and quality-class columns for every input fragment

## Limitations

- CRAQ accuracy for polyploid genomes is not established — evaluation for species with higher ploidy may be less accurate than benchmarked diploid cases
- The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, so error counts feeding the AQI may be incomplete
- Heterozygous loci must be removed during assembly assessment, since CRAQ distinguishes assembly errors from heterozygous sites or haplotype structural differences

## Evidence

- [abstract] AQI thresholds and classes: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [abstract] Error-count to AQI transformation: "Error counts are transformed into corresponding assembly evaluation indexes to reflect the assembly quality at single-nucleotide resolution"
- [discussion] Normalization and R-AQI/S-AQI computation: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [readme] CRAQ summary from README: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly"
- [abstract] Heterozygous-site requirement: "CRAQ distinguishes assembly errors from heterozygous sites or structural differences between haplotypes"

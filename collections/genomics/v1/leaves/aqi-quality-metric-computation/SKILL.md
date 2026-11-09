---
name: aqi-quality-metric-computation
description: Use when a CRAQ run has produced CRE/CSE breakpoint BED files (out_final.CRE.bed,
  out_final.CSE.bed) and you must convert normalized error counts into R-AQI/S-AQI
  scores with quality-class labels for an assembly.
when_to_use_negative:
- 'Heterozygous loci have not been removed: CRAQ stresses ''the importance of removing
  heterozygous loci during assembly assessment'', and it outputs CRH/CSH coordinates
  so heterozygous signal can be excluded before scoring — applying AQI computation
  to raw, heterozygosity-contaminated error calls inflates error counts.'
- 'The genome is polyploid: the paper states ''evaluation of genome assemblies for
  species with higher ploidy levels may not be as accurate as the benchmarked cases
  presented here''.'
- You need to detect the CRE/CSE errors themselves (read mapping, clipping-ratio cutoffs
  f = 0.75, h = 0.4-0.6, coverage m = 2 / M = 5x) — that is a separate CRAQ detection
  step; this skill only consumes its BED outputs.
tools:
- name: CRAQ
  role: Reference-free tool whose CRE/CSE breakpoint outputs (out_final.CRE.bed, out_final.CSE.bed)
    are the direct inputs to AQI transformation; also emits R-AQI & S-AQI in out_final.Report
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Perl
  role: Language (version >= 5) in which the AQI scoring/normalization implementation
    runs
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:5a584e509af8f22413ade11c9dcd78a993222cab7568f3cd5be4b4df598a3778
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# aqi-quality-metric-computation

## Summary

Transforms CRAQ-detected Clip-based Regional Error (CRE) and Clip-based Structural Error (CSE) counts into regional (R-AQI) and structural (S-AQI) assembly quality indexes using harmonic sliding-window normalization, then classifies quality (AQI > 90 reference, 80-90 high, 60-80 draft, < 60 low).

## When to use

Use when a CRAQ run has produced CRE/CSE breakpoint BED files (out_final.CRE.bed, out_final.CSE.bed) and you must convert normalized error counts into R-AQI/S-AQI scores with quality-class labels for an assembly.

## When NOT to use

- Heterozygous loci have not been removed: CRAQ stresses 'the importance of removing heterozygous loci during assembly assessment', and it outputs CRH/CSH coordinates so heterozygous signal can be excluded before scoring — applying AQI computation to raw, heterozygosity-contaminated error calls inflates error counts.
- The genome is polyploid: the paper states 'evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'.
- You need to detect the CRE/CSE errors themselves (read mapping, clipping-ratio cutoffs f = 0.75, h = 0.4-0.6, coverage m = 2 / M = 5x) — that is a separate CRAQ detection step; this skill only consumes its BED outputs.

## Inputs

- CRE breakpoint file from CRAQ run: locER_out/out_final.CRE.bed
- CSE breakpoint file from CRAQ run: strER_out/out_final.CSE.bed
- Total assembly size in megabases (from the assembled FASTA, e.g. A. oxysepala Falcon draft contigs)
- CRAQ report out_final.Report (for extracting N50, R-AQI, S-AQI in correction comparisons)

## Outputs

- aqi_scores.csv with per-fragment R-AQI, S-AQI, and quality-class columns
- Whole-assembly summary R-AQI and S-AQI values
- Quality-class labels per AQI value (reference / high / draft / low)

## How to apply

Parse the CRE file (locER_out/out_final.CRE.bed) and CSE file (strER_out/out_final.CSE.bed) together with the total assembly size in megabases, and partition each error type into sliding windows of width 0.0001 * (total assembly size). Within each window compute the normalized count Nw = sum_{i=1..m} (1/i), where m is the number of errors in the block, so a block with three errors yields Nw = 1/1 + 1/2 + 1/3 = 1.83; this harmonic normalization prevents a single error-dense block from dominating the score. Penalize CSE-window Nw values at a rate 10 times higher than CRE Nw values, since CSEs (misjoins) are more consequential than regional errors. Then compute AQI = 100 * e^(-0.1*N/L), where N is the cumulative normalized CRE count for R-AQI (or the 10x-penalized cumulative CSE count for S-AQI) and L is the total assembly length in megabases, reporting regional scores per fragment plus whole-assembly summaries. Finally classify each score with the published thresholds (AQI > 90 reference quality; 80-90 high quality; 60-80 draft quality; AQI < 60 low quality) and write per-fragment R-AQI, S-AQI, and quality-class columns to aqi_scores.csv. The normalization is implemented in the Perl component of CRAQ (Perl >= 5).

## Related tools

- **CRAQ** (Reference-free tool whose CRE/CSE breakpoint outputs (out_final.CRE.bed, out_final.CSE.bed) are the direct inputs to AQI transformation; also emits R-AQI & S-AQI in out_final.Report) — https://github.com/JiaoLaboratory/CRAQ
- **Perl** (Language (version >= 5) in which the AQI scoring/normalization implementation runs)

## Evaluation signals

- Worked-example check: a block containing three CREs/CSEs must yield Nw = 1/1 + 1/2 + 1/3 = 1.83
- All output AQI values fall in the valid range (0, 100] with a non-empty quality-class label
- Classification matches published thresholds: AQI > 90 reference, 80-90 high, 60-80 draft, AQI < 60 low
- In the A. oxysepala benchmark, computed values should approximate the reported S-AQI 58.1 vs 39.0 and R-AQI 78.7 vs 79.1 for corrected vs original scaffolds, with ~117 detected CSEs
- Sliding-window width equals 0.0001 * total assembly size in megabases, and CSE Nw values are penalized 10x relative to CRE Nw values before aggregation

## Limitations

- CSE/CRE classification depends on a user-tunable coverage-discrepancy parameter (-d) comparing SMS read coverage, with no stated universal value
- The error-breakpoint clipping ratio can be lower than 100% in practice due to sequencing errors or inaccurate read mapping
- Regions with coverage above the upper h level but below f are reported as ambiguous heterozygous-or-error regions without definitive classification
- CRAQ accuracy for polyploid genomes is not established; benchmarks were for diploid/haploid cases

## Evidence

- [discussion] The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively.: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [abstract] AQI thresholds define quality classes: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [readme] CRAQ reports CREs, CSEs, and R-AQI & S-AQI metrics: "It can report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics in regional and structural assembly"
- [discussion] Heterozygous removal is important for assessment: "demonstrating the accuracy of this tool and the importance of removing heterozygous loci during assembly assessment"

---
name: genome-assembly-quality-classification
description: Use when CRAQ output exists as CRE breakpoint files (e.g., locER_out/out_final.CRE.bed)
  and CSE breakpoint files (strER_out/out_final.CSE.bed) plus total assembly size,
  and per-fragment or whole-assembly quality scores are needed.
when_to_use_negative:
- The target genome is polyploid — CRAQ's error detection and AQI accuracy are not
  established for higher ploidy levels beyond the benchmarked diploid cases.
- Only raw sequencing reads are available with no classified CRE/CSE breakpoints —
  read mapping, clipping-based error calling (Minimap2/SAMtools filtering), and CRE/CSE
  classification must be completed before scoring.
edam_topics:
- http://edamontology.org/topic_0193
tools:
- name: CRAQ
  role: Source of CRE/CSE breakpoints and of the AQI transformation (Nw normalization
    and R-AQI/S-AQI scoring) applied here; installed via git clone from the JiaoLaboratory/CRAQ
    repository.
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Perl
  role: Runtime dependency (version >= 5) for executing CRAQ's scoring scripts that
    parse the breakpoint BED files and emit AQI metrics.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:e068bc3f4697b0861f4dc620ff560195b81fdccfbe999111f5a4c996a88bd3bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Perl
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0193
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# genome-assembly-quality-classification

## Summary

Transforms CRAQ-detected CRE/CSE error counts into sliding-window normalized values and computes regional (R-AQI) and structural (S-AQI) assembly quality indexes, then classifies assembly quality into reference/high/draft/low classes using published AQI thresholds.

## When to use

Use when CRAQ output exists as CRE breakpoint files (e.g., locER_out/out_final.CRE.bed) and CSE breakpoint files (strER_out/out_final.CSE.bed) plus total assembly size, and per-fragment or whole-assembly quality scores are needed.

## When NOT to use

- The target genome is polyploid — CRAQ's error detection and AQI accuracy are not established for higher ploidy levels beyond the benchmarked diploid cases.
- Only raw sequencing reads are available with no classified CRE/CSE breakpoints — read mapping, clipping-based error calling (Minimap2/SAMtools filtering), and CRE/CSE classification must be completed before scoring.

## Inputs

- CRE breakpoint file from CRAQ regional error detection (locER_out/out_final.CRE.bed)
- CSE breakpoint file from CRAQ structural error detection (strER_out/out_final.CSE.bed)
- Total assembly size in megabases (from the assembled FASTA, e.g., draft contigs)

## Outputs

- aqi_scores.csv with per-fragment R-AQI, S-AQI, and quality-class columns
- Whole-assembly R-AQI and S-AQI summary values with quality-class labels (reference / high / draft / low)

## How to apply

Parse the CRE and CSE breakpoint BED files and read the total assembly size in megabases. Partition CRE and CSE counts separately into sliding windows of width 0.0001 × total assembly size and compute the normalized error count per window as Nw = Σ_{i=1..m} (1/i), where m is the number of errors in the block (three errors give Nw = 1/1 + 1/2 + 1/3 = 1.83). Penalize CSE-window Nw values at 10× the rate of CRE Nw values before aggregation, since structural errors (CSEs) indicate misjoins whereas regional errors (CREs) indicate local misassemblies. Compute AQI = 100 × e^(−0.1·N/L), using cumulative normalized CRE counts for R-AQI and penalized CSE counts for S-AQI, with L the assembly length in megabases. Classify each AQI with the published thresholds — AQI > 90 reference quality, 80–90 high quality, 60–80 draft quality, AQI < 60 low quality — and write per-fragment R-AQI, S-AQI, and quality-class columns to aqi_scores.csv.

## Related tools

- **CRAQ** (Source of CRE/CSE breakpoints and of the AQI transformation (Nw normalization and R-AQI/S-AQI scoring) applied here; installed via git clone from the JiaoLaboratory/CRAQ repository.) — https://github.com/JiaoLaboratory/CRAQ
- **Perl** (Runtime dependency (version >= 5) for executing CRAQ's scoring scripts that parse the breakpoint BED files and emit AQI metrics.)

## Evaluation signals

- The Nw normalization reproduces the paper's worked example: a block containing three CREs/CSEs yields Nw = 1/1 + 1/2 + 1/3 = 1.83.
- All output AQI values fall in the valid range (0, 100] given the AQI = 100 × e^(−0.1·N/L) formula.
- Every fragment carries a non-empty quality-class label consistent with the thresholds (AQI > 90 reference; 80–90 high; 60–80 draft; < 60 low).
- CSE-derived S-AQI scores are computed from 10× penalized Nw values, so a single CSE depresses S-AQI far more than a CRE depresses R-AQI at equal error counts.
- aqi_scores.csv contains per-fragment R-AQI and S-AQI columns plus a whole-assembly summary for every fragment in the assembly.

## Limitations

- CRAQ accuracy for polyploid genomes is not established — 'evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented here'.
- Ambiguous regions with clipping ratio above the h cutoff and below f are reported without definitive classification, so some breakpoints feeding the AQI may be unresolved heterozygous-or-error calls.
- CSE/CRE classification depends on the user-tunable coverage discrepancy parameter (-d), which has no stated universal value, affecting downstream AQI reproducibility.

## Evidence

- [abstract] AQI thresholds define quality classes: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [discussion] R-AQI and S-AQI transformation: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [discussion] CRE/CSE error classification feeds the index: "Assembly errors were classified as CREs or CSEs."
- [readme] CRAQ output scope includes AQI metrics: "report precise locations of small-scale Clip-based Regional Errors (CREs), large-scale Clip-based Structural Errors (CSEs), as well as overall AQI metrics"

---
name: cre-cse-error-tallying
description: Use when CRAQ has produced locER_out/out_final.CRE.bed and strER_out/out_final.CSE.bed
  breakpoint files for a draft assembly and you need normalized error counts (Nw)
  converted into R-AQI/S-AQI scores with quality-class labels.
when_to_use_negative:
- The input is still raw read alignments or filtered BAM files — mapping, MAPQ filtering
  (-F 1796 -q 20), and CRE/CSE breakpoint calling must happen first; this skill consumes
  the resulting BED files.
- Regions identified as heterozygous (clipping ratio within h = 0.4–0.6, i.e. CRHs/CSHs)
  rather than error breakpoints — applying the tally to unfiltered heterozygous loci
  conflates haplotype variation with misassembly.
- Polyploid genomes, where CRAQ's evaluation accuracy has not been benchmarked.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: CRAQ
  role: Produces the CRE/CSE breakpoint BED files consumed by the tallying/AQI model
    and defines the AQI classification scheme
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Perl
  role: Runtime (version >= 5) in which the CRAQ AQI scoring model is implemented
  repo: https://github.com/JiaoLaboratory/CRAQ
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:c786684892163675f932d28afdf80927f7807f0a3d97ebfc0e4742dc6e280d86
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JiaoLaboratory/CRAQ
  tools:
  - CRAQ
  - Perl
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cre-cse-error-tallying

## Summary

Tallies Clip-based Regional Errors (CREs) and Clip-based Structural Errors (CSEs) from CRAQ breakpoint BED files and transforms the harmonic-normalized error counts into R-AQI and S-AQI assembly quality indexes that classify genome assemblies into reference/high/draft/low quality classes.

## When to use

Use when CRAQ has produced locER_out/out_final.CRE.bed and strER_out/out_final.CSE.bed breakpoint files for a draft assembly and you need normalized error counts (Nw) converted into R-AQI/S-AQI scores with quality-class labels.

## When NOT to use

- The input is still raw read alignments or filtered BAM files — mapping, MAPQ filtering (-F 1796 -q 20), and CRE/CSE breakpoint calling must happen first; this skill consumes the resulting BED files.
- Regions identified as heterozygous (clipping ratio within h = 0.4–0.6, i.e. CRHs/CSHs) rather than error breakpoints — applying the tally to unfiltered heterozygous loci conflates haplotype variation with misassembly.
- Polyploid genomes, where CRAQ's evaluation accuracy has not been benchmarked.

## Inputs

- CRE breakpoint file: locER_out/out_final.CRE.bed (small-scale regional errors)
- CSE breakpoint file: strER_out/out_final.CSE.bed (large-scale structural errors)
- Total assembly size in megabases (from the draft assembly FASTA)

## Outputs

- aqi_scores.csv with per-fragment R-AQI, S-AQI, and quality-class columns
- Whole-assembly R-AQI and S-AQI summary values
- Quality class labels per AQI value (reference / high / draft / low quality)

## How to apply

Parse the CRE breakpoint file (locER_out/out_final.CRE.bed) and the CSE breakpoint file (strER_out/out_final.CSE.bed) along with the total assembly size in megabases. Partition CRE and CSE counts separately into sliding windows of width 0.0001 * (total assembly size) and compute the normalized error count per window as Nw = sum_{i=1..m} (1/i), so a block containing three errors yields Nw = 1/1 + 1/2 + 1/3 = 1.83 (the paper's worked example). Penalize CSE-window Nw values at a rate 10 times higher than CRE Nw values before aggregation. Compute AQI = 100 * e^(-0.1*N/L), where N is the cumulative normalized CRE count (for R-AQI) or the penalized CSE count (for S-AQI) and L is the total assembly length in megabases, reporting regional scores per fragment plus whole-assembly summary values. Classify each AQI value with the published thresholds (>90 reference quality; 80-90 high quality; 60-80 draft quality; <60 low quality) and write per-fragment R-AQI, S-AQI, and quality-class columns to aqi_scores.csv.

## Related tools

- **CRAQ** (Produces the CRE/CSE breakpoint BED files consumed by the tallying/AQI model and defines the AQI classification scheme) — https://github.com/JiaoLaboratory/CRAQ
- **Perl** (Runtime (version >= 5) in which the CRAQ AQI scoring model is implemented) — https://github.com/JiaoLaboratory/CRAQ

## Examples

```
Nw = sum(1.0 / i for i in range(1, m + 1))  # m errors per sliding window; m=3 -> Nw=1.83; AQI = 100 * exp(-0.1 * N / L)
```

## Evaluation signals

- Normalization reproduces the paper's worked example: a block containing three CREs/CSEs yields Nw = 1/1 + 1/2 + 1/3 = 1.83
- All output AQI values fall in the valid range (0, 100]
- Every AQI value receives a non-empty quality-class label (reference / high / draft / low quality)
- aqi_scores.csv contains R-AQI, S-AQI, and quality-class columns for every assembly fragment, plus whole-assembly summary values

## Limitations

- Evaluation of genome assemblies for species with higher ploidy levels may not be as accurate as the benchmarked cases presented in the paper.
- Regions with coverage over the upper level of h and below the f value are reported as ambiguous heterozygous-or-error regions without definitive classification, so they cannot be cleanly included in or excluded from the error tally.
- The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping, and CSE/CRE classification depends on the user-tunable coverage-discrepancy parameter '-d'.

## Evidence

- [abstract] AQI thresholds define quality classes: "AQI > 90, reference quality; AQI from 80-90, high quality; AQI from 60-80, draft quality; and AQI < 60, low quality"
- [discussion] Normalized counts feed R-AQI/S-AQI: "The normalized CRE and CSE numbers were then transformed into the R-AQI and S-AQI scores, respectively."
- [readme] CRAQ as reference-free evaluator: "CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator that can assess the accuracy of assembled genomic sequences"
- [discussion] Coverage-discrepancy parameter limitation: [evidence span withheld — anchor status: not_found]

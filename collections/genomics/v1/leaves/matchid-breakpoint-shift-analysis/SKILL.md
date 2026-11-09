---
name: matchid-breakpoint-shift-analysis
description: Use when benchmarking assembly-based or read-based SV caller output (VCFs)
  against the GIAB HG002 SV gold standard and you need to quantify how sensitive caller
  ranking (F1) and breakpoint/sequence accuracy are to Truvari matching thresholds.
when_to_use_negative:
- When evaluating genotyping accuracy for inversions or duplications only — genotyping
  reciprocal translocations is ambiguous, so only F1 and GT_F1 are calculated for
  INVs and DUPs
- When the benchmark callset is incomplete (e.g., the HCC1395 high-confidence set,
  where only recall should be plotted rather than full F1 sweeps)
- When the goal is de novo SV discovery rather than caller benchmarking against a
  gold standard
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3175
tools:
- name: Truvari
  role: 'Benchmarking engine (v4.0.0): run bench per caller at baseline parameters,
    use MatchID pairing for breakpoint-shift and sequence-similarity extraction, and
    sweep p/P/O/r parameter grid recomputing precision/recall/F1'
- name: VolcanoSV
  role: SV caller under evaluation; produces the called SV VCFs (merged per variant
    type and chromosome via Merge_VCF.py) that are benchmarked against the GIAB gold
    standard
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:8c643d62b5ee289e40b9e2e30e02384ba51b87aeae025cfb91b1612e1ce89982
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Truvari
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# matchid-breakpoint-shift-analysis

## Summary

A Truvari-based robustness evaluation protocol for structural variant callers that pairs true positives via MatchID to extract per-variant breakpoint-shift distances and alternate allele sequence similarity, then sweeps the Truvari evaluation parameter grid (pctstim, pctsize, pctovl, refdist) to recompute precision, recall, and F1 per caller.

## When to use

Use when benchmarking assembly-based or read-based SV caller output (VCFs) against the GIAB HG002 SV gold standard and you need to quantify how sensitive caller ranking (F1) and breakpoint/sequence accuracy are to Truvari matching thresholds.

## When NOT to use

- When evaluating genotyping accuracy for inversions or duplications only — genotyping reciprocal translocations is ambiguous, so only F1 and GT_F1 are calculated for INVs and DUPs
- When the benchmark callset is incomplete (e.g., the HCC1395 high-confidence set, where only recall should be plotted rather than full F1 sweeps)
- When the goal is de novo SV discovery rather than caller benchmarking against a gold standard

## Inputs

- Called SV VCFs per SV caller (VolcanoSV output merged per variant type/chromosome with Merge_VCF.py; comparison callers such as cuteSV, pbsv, Sniffles2, NanoVar, DeBreak, SVDSS, MAMnet)
- GIAB HG002 SV gold standard callset (VCF)
- High-confidence region BED file restricting the GIAB benchmark
- Reference genome FASTA used for alignment/assembly (e.g., hg19/GRCh38)

## Outputs

- Table of precision, recall, and F1 per caller for every (p, P, O, r) Truvari parameter combination
- Per-true-positive-pair breakpoint-shift distance distributions extracted via Truvari MatchID
- Per-true-positive-pair alternate allele sequence similarity distributions
- Baseline-setting benchmark summary (p=0.5, P=0.5, r=500, O=0.01) for validation against reported values

## How to apply

Load each caller's SV VCFs (for VolcanoSV, merged per variant type and chromosome via Merge_VCF.py) and restrict the GIAB HG002 SV gold standard to high-confidence regions. Run Truvari (v4.0.0) bench per caller with the baseline setting p=0.5, P=0.5, r=500, O=0.01. Use Truvari's MatchID pairing to extract breakpoint-shift distances and alternate allele sequence similarity distributions for true-positive pairs. Then sweep a grid varying p, P, and O from 0 to 1 in 0.1 increments and r from 0 to 1 kb in 100 bp increments, recomputing precision, recall, and F1 at every combination. Aggregate into a table of F1 per (p, P, O, r) setting per caller, including the breakpoint-shift and sequence-similarity distributions, to identify which caller is most robust to evaluation-parameter choices.

## Related tools

- **Truvari** (Benchmarking engine (v4.0.0): run bench per caller at baseline parameters, use MatchID pairing for breakpoint-shift and sequence-similarity extraction, and sweep p/P/O/r parameter grid recomputing precision/recall/F1)
- **VolcanoSV** (SV caller under evaluation; produces the called SV VCFs (merged per variant type and chromosome via Merge_VCF.py) that are benchmarked against the GIAB gold standard) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
truvari bench -b giab_hg002_sv.vcf.gz -c volcanosv_merged.vcf.gz -f reference.fa -o truvari_out/ --pctsim 0.5 --pctsize 0.5 --refdist 500 --pctovl 0.01
```

## Evaluation signals

- Baseline-setting (p=0.5, P=0.5, r=500, O=0.01) F1 on Hifi data matches reported values: ~94.19% for deletions and ~91.03% for insertions on HG002
- VolcanoSV attains the highest F1 across all swept (p, P, O, r) parameter combinations, consistent with the paper's robustness finding
- MatchID-paired true-positive breakpoint-shift distances cluster near zero and alternate allele sequence similarity near 1 for the top-performing caller
- The parameter sweep covers p, P, O in [0, 1] with 0.1 increments and r in [0, 1 kb] with 100 bp increments — verify grid completeness (11 values per p/P/O, 11 values per r)
- Precision and recall recomputed at every grid entry are internally consistent with the Truvari bench TP/FP/FN counts at that setting

## Limitations

- The GIAB gold standard SV callset is not a complete set and could also contain false positives, so benchmark results inherit its biases
- Genotyping reciprocal translocations is ambiguous, so this F1-sweep robustness analysis does not extend to translocation genotype accuracy
- Evaluation restricted to high-confidence GIAB regions; calls outside these regions are excluded from the sweep

## Evidence

- [abstract] VolcanoSV is robust to various evaluation parameters and accurately identifies breakpoints and SV sequences.: "VolcanoSV is robust to various evaluation parameters and accurately identifies breakpoints and SV sequences."
- [results] Truvari baseline parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Truvari parameter grid sweep: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [results] Truvari benchmarking against GIAB: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Expected baseline F1 values on Hifi: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."

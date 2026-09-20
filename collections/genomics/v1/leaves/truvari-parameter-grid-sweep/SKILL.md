---
name: truvari-parameter-grid-sweep
description: Use when benchmarking SV callers against a gold standard (e.g., GIAB
  HG002) and you need to show that caller rankings (precision/recall/F1) are robust
  to Truvari's matching-parameter choices rather than artifacts of a single setting.
when_to_use_negative:
- When the evaluation question is somatic SV detection against an incomplete high-confidence
  callset where only recall is meaningful
- When comparing against the CHM13 homozygous sample or T2T-CHM13 for false discovery
  rate estimation rather than GIAB gold-standard benchmarking
- When genotyping accuracy for reciprocal translocations is needed, since GT_F1 is
  ambiguous for TRAs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Benchmarking SV calls against the GIAB gold standard and sweeping matching
    parameters (p, P, O, r) to produce per-setting F1 and MatchID-based TP variant
    comparisons
- name: VolcanoSV
  role: SV caller under evaluation; its merged per-type, per-chromosome VCFs are compared
    against other callers across the Truvari parameter grid
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: cuteSV
  role: Comparison SV caller included in the grid sweep to contrast robustness with
    VolcanoSV
- name: Sniffles2
  role: Comparison SV caller included in the grid sweep
- name: SVIM-asm
  role: Assembly-based comparison SV caller included in the grid sweep
  repo: https://github.com/eldariont/svim-asm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:7c637e3eac239b33dc33b93f189a76b2bea496857ddee7e33d738ebc4efd1264
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VolcanoSV
  - cuteSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# truvari-parameter-grid-sweep

## Summary

A robustness evaluation protocol that sweeps Truvari bench comparison parameters (pctsim, pctsize, pctovl, refdist) across a dense grid to test how sensitive SV caller benchmark results are to evaluation settings, as done for VolcanoSV against assembly-based SV callers.

## When to use

Use when benchmarking SV callers against a gold standard (e.g., GIAB HG002) and you need to show that caller rankings (precision/recall/F1) are robust to Truvari's matching-parameter choices rather than artifacts of a single setting.

## When NOT to use

- When the evaluation question is somatic SV detection against an incomplete high-confidence callset where only recall is meaningful
- When comparing against the CHM13 homozygous sample or T2T-CHM13 for false discovery rate estimation rather than GIAB gold-standard benchmarking
- When genotyping accuracy for reciprocal translocations is needed, since GT_F1 is ambiguous for TRAs

## Inputs

- Called SV VCFs per SV caller (e.g., VolcanoSV merged per variant type and chromosome via Merge_VCF.py)
- GIAB HG002 SV gold standard callset restricted to high-confidence regions (VCF/BED)
- Reference genome FASTA
- Truvari v4.0.0 with bench and MatchID output

## Outputs

- Table of precision, recall, and F1 per (p, P, O, r) Truvari setting per caller
- Baseline Truvari bench results at p=0.5, P=0.5, r=500, O=0.01 (e.g., ~94.19% F1 for deletions and ~91.03% for insertions on HiFi data)
- Per-variant breakpoint-shift distance distributions for true-positive pairs
- Per-variant alternate allele sequence similarity distributions for true-positive pairs

## How to apply

First benchmark each SV caller's VCFs with Truvari v4.0.0 against the GIAB HG002 SV gold standard using baseline parameters p=0.5, P=0.5, r=500, O=0.01. Use Truvari's MatchID pairing to extract per-variant breakpoint-shift distances and alternate allele sequence similarity for true-positive pairs, which characterizes caller accuracy beyond set-level F1. Then sweep the grid: vary p, P, and O from 0 to 1 in increments of 0.1 and r from 0 to 1 kb in increments of 100 bp, recomputing precision, recall, and F1 at every (p, P, O, r) combination for each caller. Aggregate into a table of F1 per parameter setting per caller, plus breakpoint-shift and sequence-similarity distributions. The rationale is that a caller whose F1 is stable across the grid and remains highest at every entry has genuinely better performance rather than one tuned to the evaluator's default thresholds.

## Related tools

- **Truvari** (Benchmarking SV calls against the GIAB gold standard and sweeping matching parameters (p, P, O, r) to produce per-setting F1 and MatchID-based TP variant comparisons)
- **VolcanoSV** (SV caller under evaluation; its merged per-type, per-chromosome VCFs are compared against other callers across the Truvari parameter grid) — https://github.com/maiziezhoulab/VolcanoSV
- **cuteSV** (Comparison SV caller included in the grid sweep to contrast robustness with VolcanoSV)
- **Sniffles2** (Comparison SV caller included in the grid sweep)
- **SVIM-asm** (Assembly-based comparison SV caller included in the grid sweep) — https://github.com/eldariont/svim-asm

## Examples

```
truvari bench -b giab_hg002.vcf.gz -c volcanosv_merged.vcf.gz -f ref.fa -p 0.5 -P 0.5 -r 500 -O 0.01 --includebed confident.bed -o bench_out
```

## Evaluation signals

- Baseline-setting F1 matches reported values (e.g., ~94.19% for deletions and ~91.03% for insertions on HiFi data)
- VolcanoSV attains the highest F1 at every (p, P, O, r) grid entry, consistent with the paper's robustness finding
- F1 varies smoothly across the parameter grid without abrupt discontinuities for the top caller
- Breakpoint-shift distances and alternate allele sequence similarities of MatchID-paired true positives are tightly distributed near zero shift and near-perfect similarity
- Precision/recall trade-offs recompute consistently from the same MatchID TP/FP/FN assignments at each grid point

## Limitations

- The GIAB gold standard SV callset is not a complete set and could contain false positives, so robustness conclusions inherit its biases
- Only F1 and GT_F1 are calculated for INVs and DUPs since genotyping reciprocal TRAs is ambiguous
- Results are specific to the caller set and datasets (HG002 HiFi/CLR/ONT, CHM13); rankings could differ for other samples or callers

## Evidence

- [results] Grid search of Truvari parameters: p, P, O from 0 to 1 in 0.1 increments, r from 0 to 1 kb in 100 bp increments: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [results] Baseline Truvari parameter setting: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Truvari version and gold standard used for benchmarking: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Reported baseline F1 values on HiFi data: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [readme] Truvari evaluation section in the VolcanoSV README: "- [Truvari evaluation](#Truvari-evaluation)"

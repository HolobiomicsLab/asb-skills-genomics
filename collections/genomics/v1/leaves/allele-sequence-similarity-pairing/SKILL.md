---
name: allele-sequence-similarity-pairing
description: Use when you have called SV VCFs (e.g. from VolcanoSV, cuteSV, Sniffles2,
  pbsv, DeBreak) and a GIAB-style truth set, and need to measure how F1 depends on
  Truvari matching thresholds plus how accurately TP pairs recover breakpoints and
  allele sequences.
when_to_use_negative:
- Genotyping-accuracy evaluation of reciprocal translocations — the study only calculated
  F1 and GT_F1 for INVs and DUPs because genotyping reciprocal TRAs is ambiguous
- Evaluating somatic SVs against an incomplete high-confidence callset (e.g. HCC1395),
  where only recall should be plotted rather than precision/F1
- Input is not a compared pair of SV VCFs against a curated truth set — this skill
  is a benchmarking/robustness protocol, not a caller itself
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Bench tool (v4.0.0) comparing each caller's SV calls to the GIAB gold standard;
    its MatchID pairing supplies the breakpoint-shift and alternate-allele-sequence-similarity
    values for TP pairs, and its parameters (p, P, r, O) are swept across the grid
- name: VolcanoSV
  role: SV caller under evaluation; its per-type VCFs are merged via Merge_VCF.py
    before benchmarking, and it is compared for robustness against alignment- and
    assembly-based callers
  repo: https://github.com/maiziezhoulab/VolcanoSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:aab0ad547247c12c2404fd751711031a6aefd22972a9202883fbc1e3d299088f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Truvari
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# allele-sequence-similarity-pairing

## Summary

Benchmarks SV caller VCFs against the GIAB HG002 gold standard with Truvari bench, then uses Truvari's MatchID pairing to extract per-variant breakpoint-shift distances and alternate allele sequence similarity for true-positive pairs, and sweeps Truvari parameters (p, P, O, r) to test robustness of precision/recall/F1.

## When to use

Use when you have called SV VCFs (e.g. from VolcanoSV, cuteSV, Sniffles2, pbsv, DeBreak) and a GIAB-style truth set, and need to measure how F1 depends on Truvari matching thresholds plus how accurately TP pairs recover breakpoints and allele sequences.

## When NOT to use

- Genotyping-accuracy evaluation of reciprocal translocations — the study only calculated F1 and GT_F1 for INVs and DUPs because genotyping reciprocal TRAs is ambiguous
- Evaluating somatic SVs against an incomplete high-confidence callset (e.g. HCC1395), where only recall should be plotted rather than precision/F1
- Input is not a compared pair of SV VCFs against a curated truth set — this skill is a benchmarking/robustness protocol, not a caller itself

## Inputs

- Called SV VCFs per tool (VolcanoSV outputs merged per variant type and chromosome with Merge_VCF.py; also cuteSV, pbsv, Sniffles2, NanoVar, MAMnet, SVDSS, DeBreak)
- GIAB HG002 SV gold standard callset restricted to high-confidence regions
- Truvari v4.0.0 bench configuration (baseline p=0.5, P=0.5, r=500, O=0.01)

## Outputs

- Truvari bench results (precision, recall, F1) per caller at each parameter setting
- Table of F1 per (p, P, O, r) combination per caller over the parameter grid
- Per-variant breakpoint-shift distance distributions for TP pairs (via MatchID)
- Per-variant alternate allele sequence similarity distributions for TP pairs (via MatchID)

## How to apply

Load each caller's SV VCFs (for VolcanoSV, merged per variant type and chromosome via Merge_VCF.py) and restrict the GIAB HG002 SV gold standard to high-confidence regions. Run Truvari (v4.0.0) bench per caller with the baseline setting p=0.5, P=0.5, r=500, O=0.01 (pctsim, pctsize, pctovl, refdist). Use Truvari's MatchID pairing to extract, for each true-positive pair, the breakpoint-shift distance and alternate allele sequence similarity, yielding accuracy distributions rather than only counts. Sweep p, P, O from 0 to 1 in increments of 0.1 and r from 0 to 1 kb in increments of 100 bp, recomputing precision, recall, and F1 at every combination. Aggregate into a table of F1 per (p, P, O, r) setting per caller; robustness is judged by how little F1 degrades across the grid (in the source study VolcanoSV held the highest F1 across all grid entries).

## Related tools

- **Truvari** (Bench tool (v4.0.0) comparing each caller's SV calls to the GIAB gold standard; its MatchID pairing supplies the breakpoint-shift and alternate-allele-sequence-similarity values for TP pairs, and its parameters (p, P, r, O) are swept across the grid)
- **VolcanoSV** (SV caller under evaluation; its per-type VCFs are merged via Merge_VCF.py before benchmarking, and it is compared for robustness against alignment- and assembly-based callers) — https://github.com/maiziezhoulab/VolcanoSV

## Examples

```
truvari bench -b giab_hg002_sv.vcf.gz -c volcanosv_merged.vcf.gz -p 0.5 -P 0.5 -r 500 -O 0.01 -o bench_volcanosv_hifi
```

## Evaluation signals

- At the baseline setting (p=0.5, P=0.5, r=500, O=0.01), HiFi-data F1 should reproduce the reported values (~94.19% for deletions and ~91.03% for insertions on HG002)
- The grid sweep should yield 11 values per parameter for p, P, O (0–1 step 0.1) and 11 values for r (0–1 kb step 100 bp), i.e. a complete F1 table over all combinations
- Robustness check: the strongest caller (VolcanoSV in the source study) should retain the highest F1 across all swept parameter combinations rather than only at the baseline
- MatchID-extracted TP pairs must be one-to-one with bench TP entries, with non-degenerate breakpoint-shift and sequence-similarity distributions (not all zeros/ones)
- Recomputed F1 at each grid entry must equal the value derivable from that run's TP/FP/FN counts

## Limitations

- The GIAB gold standard SV callset is not complete and could contain false positives, capping achievable precision/recall
- Baseline F1 depends on the specific Truvari metric setting (p=0.5, P=0.5, r=500, O=0.01); conclusions can shift with different thresholds, which is why the grid sweep is performed
- Genotyping comparisons are ambiguous for reciprocal translocations, so only F1 (not GT_F1) is meaningful for INVs/DUPs and TRA genotyping is excluded

## Evidence

- [results] An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Baseline Truvari metric setting: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Parameter grid definition for the robustness sweep: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [abstract] Robustness and breakpoint/sequence accuracy claim supported by the pairing analysis: "VolcanoSV is robust to various evaluation parameters and accurately identifies breakpoints and SV sequences."
- [results] Reported baseline F1 values used to validate the benchmark run: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."

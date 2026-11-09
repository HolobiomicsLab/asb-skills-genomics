---
name: f1-score-across-tools-comparison
description: Use when comparing SV-calling tool performance (F1/precision/recall)
  against a benchmark callset, especially to test robustness of tool rankings to Truvari
  parameter choices (p, P, O, r).
when_to_use_negative:
- When the benchmark truth set is incomplete (e.g., HCC1395 high-confidence callset
  built only from alignment-based tools) — recall-only comparison is more appropriate
- When genotyping-based metrics are needed for reciprocal translocations, where GT
  comparison is ambiguous (only F1/GT_F1 computed for INVs and DUPs in this context)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: VolcanoSV
  role: SV caller under evaluation; its merged VCFs (per variant type and chromosome)
    are benchmarked against GIAB with Truvari
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Truvari
  role: Benchmarking tool (v4.0.0) used for bench runs, MatchID-based TP pairing,
    and the parameter grid sweep (p, P, O, r)
- name: Dipcall
  role: Assembly-based SV caller compared in the F1 benchmarking
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based SV caller compared in the F1 benchmarking
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based SV caller compared in the F1 benchmarking
  repo: https://github.com/EichlerLab/pav
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:08e92b0ddb0d7a1459db3c336022e680c8b2c78d3ef611e6f8b436d44b33eaca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Dipcall
  - PAV
  - SVIM-asm
  - Truvari
  - VolcanoSV
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# f1-score-across-tools-comparison

## Summary

Benchmark multiple SV callers against a gold standard (GIAB HG002) using Truvari, sweeping the evaluation parameters (pctstim, pctsize, pctovl, refdist) to compare precision, recall, and F1 robustness across tools, as done for VolcanoSV versus assembly-based callers.

## When to use

Use when comparing SV-calling tool performance (F1/precision/recall) against a benchmark callset, especially to test robustness of tool rankings to Truvari parameter choices (p, P, O, r).

## When NOT to use

- When the benchmark truth set is incomplete (e.g., HCC1395 high-confidence callset built only from alignment-based tools) — recall-only comparison is more appropriate
- When genotyping-based metrics are needed for reciprocal translocations, where GT comparison is ambiguous (only F1/GT_F1 computed for INVs and DUPs in this context)

## Inputs

- Called SV VCFs per tool (e.g., VolcanoSV VCFs merged per variant type and chromosome via Merge_VCF.py)
- GIAB HG002 SV gold standard callset (VCF) restricted to high-confidence regions
- Reference genome FASTA (e.g., hg19/GRCh38) with index

## Outputs

- Table of F1 (and precision/recall) per Truvari parameter combination (p, P, O, r) per caller
- Per-variant breakpoint-shift distance and alternate allele sequence similarity distributions for TP pairs
- Baseline-setting F1 values per SV type and dataset (e.g., ~94.19% DEL and ~91.03% INS for VolcanoSV on HiFi)

## How to apply

Run Truvari (v4.0.0) bench for each caller (e.g., VolcanoSV, Dipcall, SVIM-asm, PAV) against the GIAB HG002 SV gold standard restricted to high-confidence regions, using baseline settings p=0.5, P=0.5, r=500, O=0.01. Use Truvari's MatchID pairing to extract per-variant breakpoint-shift distances and alternate allele sequence similarity for true-positive pairs. Then sweep a parameter grid varying p, P, and O from 0 to 1 in increments of 0.1 and r from 0 to 1 kb in increments of 100 bp, recomputing precision, recall, and F1 at every combination. Aggregate results into a per-caller F1 table across all (p, P, O, r) settings to assess which tool's performance is most robust to evaluation-parameter changes.

## Related tools

- **VolcanoSV** (SV caller under evaluation; its merged VCFs (per variant type and chromosome) are benchmarked against GIAB with Truvari) — https://github.com/maiziezhoulab/VolcanoSV
- **Truvari** (Benchmarking tool (v4.0.0) used for bench runs, MatchID-based TP pairing, and the parameter grid sweep (p, P, O, r))
- **Dipcall** (Assembly-based SV caller compared in the F1 benchmarking) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based SV caller compared in the F1 benchmarking) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based SV caller compared in the F1 benchmarking) — https://github.com/EichlerLab/pav

## Examples

```
truvari bench -b giab_hg002.sv.vcf.gz -c volcanosv.del.merged.vcf.gz --pctstim 0.5 --pctsize 0.5 --refdist 500 --pctovl 0.01 -o truvari_del_bench
```

## Evaluation signals

- Baseline-setting F1 (p=0.5, P=0.5, r=500, O=0.01) matches reported values, e.g., ~94.19% for deletions and ~91.03% for insertions for VolcanoSV on HiFi data
- Grid sweep covers the full space: p, P, O from 0 to 1 in 0.1 increments and r from 0 to 1 kb in 100 bp increments, with F1 recomputed at every combination
- VolcanoSV attains the highest F1 across all swept parameter combinations, consistent with the paper's robustness finding
- Truvari MatchID pairing yields TP-pair breakpoint-shift and sequence-similarity distributions for each caller
- Precision, recall, and F1 are internally consistent (F1 = harmonic mean of precision and recall) at each grid point

## Limitations

- The GIAB gold standard SV callset is not a complete set and could also contain false positives
- High-confidence benchmark callsets built only from alignment-based tools can bias evaluation against assembly-based callers
- F1 comparison alone ignores genotyping accuracy; genotyping of reciprocal translocations is ambiguous and was excluded

## Evidence

- [results] An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Truvari parameter grid sweep: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [results] Baseline Truvari parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Reported F1 values: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [abstract] Robustness finding: "VolcanoSV is robust to various evaluation parameters and accurately identifies breakpoints and SV sequences."

---
name: sv-benchmarking-with-truvari
description: Use when SV caller VCFs (DEL/INS/INV/TRA/DUP) need quantitative evaluation
  against a GIAB gold standard, including low-coverage robustness tests and Truvari
  parameter-sensitivity sweeps.
when_to_use_negative:
- Benchmarking somatic SVs in the HCC1395 tumor where the high-confidence callset
  is incomplete — only recall is plottable, so full F1 evaluation is inappropriate
- Evaluating small indels from ONT assembly-based calling, where the article found
  assembly proved ineffective (low precision for all three tools)
- Comparing against a benchmark callset built exclusively from alignment-based tools,
  which the article notes may bias evaluation of hybrid/assembly methods
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Truvari
  role: Benchmarks caller VCFs against the GIAB gold standard with p=0.5, P=0.5, r=500,
    O=0.01; MatchID pairing extracts TP breakpoint shifts and sequence similarity;
    parameter grid swept for robustness
- name: VolcanoSV
  role: Caller under evaluation; VCFs merged per variant type and chromosome via Merge_VCF.py
    before benchmarking
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Assembly-based baseline caller whose DEL/INS VCFs are benchmarked on the same
    datasets
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based baseline caller benchmarked alongside VolcanoSV
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based baseline caller benchmarked alongside VolcanoSV
  repo: https://github.com/EichlerLab/pav
- name: rasusa
  role: Subsamples HG002 long-read BAMs to 40x/30x/20x/10x/5x coverage before calling,
    enabling low-coverage robustness benchmarking
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:0fc21e51477fe1fadf3adc328e175a2792122addc4817bd6c4c50958953c4c4e
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
  - rasusa
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# sv-benchmarking-with-truvari

## Summary

Benchmark structural variant caller outputs (VCFs) against the GIAB HG002 SV gold standard using Truvari v4.0.0, extracting recall, precision, F1, and genotype accuracy per SV type, dataset, and coverage level.

## When to use

Use when SV caller VCFs (DEL/INS/INV/TRA/DUP) need quantitative evaluation against a GIAB gold standard, including low-coverage robustness tests and Truvari parameter-sensitivity sweeps.

## When NOT to use

- Benchmarking somatic SVs in the HCC1395 tumor where the high-confidence callset is incomplete — only recall is plottable, so full F1 evaluation is inappropriate
- Evaluating small indels from ONT assembly-based calling, where the article found assembly proved ineffective (low precision for all three tools)
- Comparing against a benchmark callset built exclusively from alignment-based tools, which the article notes may bias evaluation of hybrid/assembly methods

## Inputs

- SV caller output VCFs (e.g., merged VolcanoSV VCFs from Merge_VCF.py, or VCFs from Dipcall, SVIM-asm, PAV)
- GIAB HG002 SV gold standard callset VCF
- GIAB high-confidence region BED
- Truvari parameter grid specification (p, P, r, O)

## Outputs

- Truvari bench summary with recall, precision, F1, and GT_F1 per caller/SV type/dataset/coverage
- Parameter-sweep table of F1 per (p, P, O, r) combination per caller
- Breakpoint-shift and alternate-allele sequence-similarity distributions for TP pairs
- Aggregated metrics CSV (e.g., sv_benchmark_metrics.csv)

## How to apply

Run Truvari v4.0.0 bench on each caller's merged VCF against the GIAB HG002 SV gold standard restricted to high-confidence regions, using the baseline parameters p=0.5, P=0.5, r=500, S=30, O=0.01. Extract recall, precision, F1, and GT_F1 per tool, dataset, and SV type; for INVs and DUPs only compute F1 and GT_F1 since genotyping reciprocal TRAs is ambiguous. To test robustness, sweep p, P, and O from 0 to 1 in 0.1 increments and r from 0 to 1 kb in 100 bp increments, recomputing F1 at every grid point. Optionally use Truvari's MatchID pairing to derive per-variant breakpoint-shift distances and alternate allele sequence similarity distributions for true-positive pairs. Compile all metrics into a single table (e.g., sv_benchmark_metrics.csv) and validate against reported values (e.g., ~94.19% DEL and ~91.03% INS F1 for VolcanoSV on Hifi data).

## Related tools

- **Truvari** (Benchmarks caller VCFs against the GIAB gold standard with p=0.5, P=0.5, r=500, O=0.01; MatchID pairing extracts TP breakpoint shifts and sequence similarity; parameter grid swept for robustness)
- **VolcanoSV** (Caller under evaluation; VCFs merged per variant type and chromosome via Merge_VCF.py before benchmarking) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Assembly-based baseline caller whose DEL/INS VCFs are benchmarked on the same datasets) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based baseline caller benchmarked alongside VolcanoSV) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based baseline caller benchmarked alongside VolcanoSV) — https://github.com/EichlerLab/pav
- **rasusa** (Subsamples HG002 long-read BAMs to 40x/30x/20x/10x/5x coverage before calling, enabling low-coverage robustness benchmarking)

## Examples

```
truvari bench -b giab_hg002.vcf.gz -c volcanosv_merged.vcf.gz -f GRCh38.fa --pctsim 0.5 --pctsize 0.5 --refdist 500 --sizemax 30 --pctovl 0.01 -o truvari_out
```

## Evaluation signals

- At baseline parameters (p=0.5, P=0.5, r=500, O=0.01), VolcanoSV Hifi DEL F1 ≈ 94.19% and INS F1 ≈ 91.03% match reported values
- VolcanoSV preserves high F1 for insertions and deletions at 10x subsampled coverage, exceeding PAV, SVIM-asm, and Dipcall
- VolcanoSV attains the highest F1 across all swept Truvari parameter combinations (p, P, O ∈ [0,1] step 0.1; r ∈ [0,1kb] step 100bp)
- Metrics table contains entries for every tool × dataset × SV type × coverage combination, explicitly including 10x runs
- GT_F1 reported separately from F1, with F1/GT_F1 only (no genotype accuracy) for INVs and DUPs

## Limitations

- The GIAB gold standard SV callset is not complete and could contain false positives
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing evaluation against assembly-based callers
- Genotyping reciprocal translocations is ambiguous, so genotype accuracy is not computed for TRAs
- Truvari metrics are sensitive to evaluation parameters, necessitating the robustness sweep

## Evidence

- [results] Truvari baseline parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Truvari version and gold standard: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Parameter grid sweep: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [results] Reported F1 values: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [results] Excluding TRA genotyping: "We thus only calculated F1 and GT_F1 for INVs and DUPs"

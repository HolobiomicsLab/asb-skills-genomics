---
name: giab-gold-standard-vcf-comparison
description: Use when evaluating DEL/INS SV calls from long-read SV callers against
  the GIAB HG002 gold standard to quantify recall, precision, F1, and genotype accuracy
  with Truvari.
when_to_use_negative:
- Evaluating somatic complex SVs against the HCC1395 high-confidence callset, which
  is incomplete and includes SV calls only from alignment-based tools, so only recall
  (not precision/F1) is meaningful there
- Genotyping assessment of reciprocal translocations (TRAs), where genotyping reciprocity
  is ambiguous — only F1 and GT_F1 are computed for INVs and DUPs instead
- Estimating false discovery rate against a different reference (e.g., CHM13 vs GRCh38),
  which uses a ratio-of-counts approach rather than gold-standard benchmarking
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: SV benchmarking tool comparing each caller's VCF to the GIAB gold standard;
    parameterized p=0.5, P=0.5, r=500, O=0.01 at v4.0.0
- name: VolcanoSV
  role: SV caller under evaluation; merged DEL/INS VCFs (Merge_VCF.py) serve as comparison
    input
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: Dipcall
  role: Assembly-based baseline caller benchmarked on the same datasets
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based baseline caller benchmarked on the same datasets
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based baseline caller benchmarked on the same datasets
  repo: https://github.com/EichlerLab/pav
- name: rasusa
  role: Subsamples read sets to 40x/30x/20x/10x/5x coverage for robustness benchmarking
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:651048af5abeb039b1445703d1b968783af3393f9e7b3aaecdf30cfd54869a52
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# giab-gold-standard-vcf-comparison

## Summary

Benchmark structural variant caller VCFs (e.g., VolcanoSV, Dipcall, SVIM-asm, PAV) against the GIAB HG002 SV gold standard using Truvari v4.0.0, computing recall, precision, F1, and genotype accuracy per SV type, dataset, and coverage level.

## When to use

Use when evaluating DEL/INS SV calls from long-read SV callers against the GIAB HG002 gold standard to quantify recall, precision, F1, and genotype accuracy with Truvari.

## When NOT to use

- Evaluating somatic complex SVs against the HCC1395 high-confidence callset, which is incomplete and includes SV calls only from alignment-based tools, so only recall (not precision/F1) is meaningful there
- Genotyping assessment of reciprocal translocations (TRAs), where genotyping reciprocity is ambiguous — only F1 and GT_F1 are computed for INVs and DUPs instead
- Estimating false discovery rate against a different reference (e.g., CHM13 vs GRCh38), which uses a ratio-of-counts approach rather than gold-standard benchmarking

## Inputs

- Per-caller SV VCFs for deletions and insertions (e.g., VolcanoSV outputs merged via Merge_VCF.py, plus Dipcall, SVIM-asm, PAV VCFs)
- GIAB HG002 SV gold standard VCF restricted to high-confidence regions
- Long-read BAM datasets (HG002 Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6; optionally subsampled to 40x/30x/20x/10x/5x with rasusa v0.6.0)
- Reference genome (e.g., hg19/GRCh38 FASTA with index)

## Outputs

- Truvari bench comparison reports (TP/FP/FN variant pairs) per caller and dataset
- Table of recall, precision, F1, and genotype accuracy per tool, SV type, dataset, and coverage level (e.g., sv_benchmark_metrics.csv)
- F1 grid over Truvari parameter combinations (p, P, O from 0–1 step 0.1; r from 0–1 kb step 100 bp) per caller
- Breakpoint-shift distance and alternate allele sequence similarity distributions for true-positive pairs

## How to apply

Run Truvari (v4.0.0) bench on each caller's merged DEL/INS VCF (e.g., VolcanoSV outputs merged per variant type and chromosome with Merge_VCF.py) against the GIAB HG002 SV gold standard callset restricted to high-confidence regions, using the baseline parameters p=0.5, P=0.5, r=500, O=0.01. Extract recall, precision, F1, and genotype accuracy per tool, dataset (Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6), SV type, and coverage level, including 10x subsampled runs produced with rasusa v0.6.0. Optionally sweep a parameter grid — p, P, and O from 0 to 1 in 0.1 increments and r from 0 to 1 kb in 100 bp increments — recomputing F1 at each combination to test robustness, and use Truvari MatchID pairing to extract breakpoint-shift distances and alternate allele sequence similarity for true-positive pairs. Validate by checking that baseline F1 reproduces reported values (≈94.19% deletions and ≈91.03% insertions on Hifi) and that VolcanoSV retains the highest F1 across the sweep. Compile all metrics into a single table such as sv_benchmark_metrics.csv.

## Related tools

- **Truvari** (SV benchmarking tool comparing each caller's VCF to the GIAB gold standard; parameterized p=0.5, P=0.5, r=500, O=0.01 at v4.0.0)
- **VolcanoSV** (SV caller under evaluation; merged DEL/INS VCFs (Merge_VCF.py) serve as comparison input) — https://github.com/maiziezhoulab/VolcanoSV
- **Dipcall** (Assembly-based baseline caller benchmarked on the same datasets) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based baseline caller benchmarked on the same datasets) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based baseline caller benchmarked on the same datasets) — https://github.com/EichlerLab/pav
- **rasusa** (Subsamples read sets to 40x/30x/20x/10x/5x coverage for robustness benchmarking)

## Examples

```
truvari bench -b giab_hg002_sv.vcf.gz -c volcano_sv_merged.vcf.gz -f hg19.fa -p 0.5 -P 0.5 -r 500 -O 0.01
```

## Evaluation signals

- Baseline-setting (p=0.5, P=0.5, r=500, O=0.01) F1 matches reported values: ~94.19% for deletions and ~91.03% for insertions on Hifi data
- Genotype accuracy reproduces reported averages (98.32% insertions, 99.01% deletions across Hifi datasets; 97.07% and 98.58% across CLR)
- VolcanoSV attains the highest F1 across all swept Truvari parameter combinations (p, P, O ∈ [0,1] step 0.1; r ∈ [0,1 kb] step 100 bp)
- High F1 is preserved for both insertions and deletions even at 10x subsampled coverage, exceeding the other three assembly-based tools
- Output table has one row per (tool, dataset, SV type, coverage level) with non-null recall/precision/F1/genotype accuracy columns

## Limitations

- The GIAB gold standard SV callset is not a complete set and could also contain false positives
- Genotyping reciprocal TRAs is ambiguous, so only F1 and GT_F1 are calculated for INVs and DUPs
- VolcanoSV achieved the best insertion/deletion recall in only 3–4 out of 6 ONT datasets, so ONT results are less uniformly favorable than Hifi/CLR

## Evidence

- [results] truvari_parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] truvari_giab_benchmark: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] hifi_f1_expectation: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [results] subsampled_coverage: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)37"
- [results] gold_standard_incomplete: "the GIAB gold standard SV callset is not a complete set and could also contain false positives"
- [results] grid_search: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"

---
name: genotype-accuracy-computation
description: Use when benchmarking diploid SV callers (e.g., VolcanoSV vs PAV, SVIM-asm,
  Dipcall) on long-read HG002 data and you need genotype accuracy per DEL/INS call
  at multiple coverage depths.
when_to_use_negative:
- Benchmarking genotypes for reciprocal translocations (TRAs), since genotyping is
  ambiguous — only F1/GT_F1 should be computed for INVs and DUPs in that context
- Evaluating against the HCC1395 somatic high-confidence callset, which includes SV
  calls only from alignment-based tools and is incomplete
- Assessing small indel genotype accuracy on ONT data, where assembly-based small
  indel detection showed low precision for all tools
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Benchmarks caller VCFs against the GIAB HG002 gold standard and reports genotype
    concordance metrics under p=0.5, P=0.5, r=500, S=30, O=0.01
- name: VolcanoSV
  role: SV caller under evaluation; WGS-mode pipeline (volcanosv-asm.py, volcanosv-vc-large-indel.py,
    volcanosv-vc-complex-sv.py, volcanosv-vc-small-indel.py, Merge_VCF.py) produces
    the phased DEL/INS VCFs being genotyped
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: rasusa
  role: Subsamples HG002 long-read BAMs to 40x, 30x, 20x, 10x, and 5x coverage to
    generate genotype-accuracy test points
- name: Dipcall
  role: Assembly-based baseline caller whose genotypes are compared in the same benchmark
  repo: https://github.com/lh3/dipcall
- name: SVIM-asm
  role: Assembly-based baseline caller whose genotypes are compared in the same benchmark
  repo: https://github.com/eldariont/svim-asm
- name: PAV
  role: Assembly-based baseline caller whose genotypes are compared in the same benchmark
  repo: https://github.com/EichlerLab/pav
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:7b27e5e7ee6d3748364eb5506117d6022182c3cf8d70f4314bdc760f18ce7a03
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7727272727272727
n_claims: 22
---

# genotype-accuracy-computation

## Summary

Computes genotype (GT) accuracy for structural variant callers by benchmarking caller VCFs against the GIAB HG002 SV gold standard with Truvari v4.0.0 under fixed parameters, extracting GT accuracy alongside recall, precision, and F1 per tool, SV type, and coverage level.

## When to use

Use when benchmarking diploid SV callers (e.g., VolcanoSV vs PAV, SVIM-asm, Dipcall) on long-read HG002 data and you need genotype accuracy per DEL/INS call at multiple coverage depths.

## When NOT to use

- Benchmarking genotypes for reciprocal translocations (TRAs), since genotyping is ambiguous — only F1/GT_F1 should be computed for INVs and DUPs in that context
- Evaluating against the HCC1395 somatic high-confidence callset, which includes SV calls only from alignment-based tools and is incomplete
- Assessing small indel genotype accuracy on ONT data, where assembly-based small indel detection showed low precision for all tools

## Inputs

- SV caller output VCFs (DEL/INS calls with GT genotypes, e.g., VolcanoSV merged VCFs, PAV, SVIM-asm, Dipcall)
- GIAB HG002 SV gold standard VCF
- Genome reference FASTA (hg19/GRCh38) and high-confidence region BED
- Subsampled long-read BAMs (Hifi_L1-L5, CLR_L1-L3, ONT_L1-L6) at 5x-40x coverage

## Outputs

- Per-tool, per-dataset, per-SV-type, per-coverage table of recall, precision, F1, and genotype accuracy (e.g., sv_benchmark_metrics.csv)
- Truvari bench summary and TP/FP/FN annotated VCFs

## How to apply

Run each SV caller on the subsampled long-read BAMs (e.g., rasusa v0.6.0 downsampling to 40x, 30x, 20x, 10x, and 5x), then compare each tool's DEL/INS VCF against the GIAB HG002 SV gold standard using Truvari v4.0.0 restricted to high-confidence regions with p=0.5, P=0.5, r=500, S=30, O=0.01. From the Truvari output, extract recall, precision, F1, and genotype accuracy (GT concordance) per tool, dataset, SV type, and coverage level. Compute GT accuracy as the fraction of true positive SVs whose genotype call (e.g., 0/1 vs 1/1) matches the gold standard. Note that reciprocal genotyping is ambiguous for TRAs, so GT accuracy is computed for INVs/DUPs only at the F1/GT_F1 level. Compile all metrics into a single table (e.g., sv_benchmark_metrics.csv) to compare robustness across platforms (Hifi, CLR, ONT).

## Related tools

- **Truvari** (Benchmarks caller VCFs against the GIAB HG002 gold standard and reports genotype concordance metrics under p=0.5, P=0.5, r=500, S=30, O=0.01)
- **VolcanoSV** (SV caller under evaluation; WGS-mode pipeline (volcanosv-asm.py, volcanosv-vc-large-indel.py, volcanosv-vc-complex-sv.py, volcanosv-vc-small-indel.py, Merge_VCF.py) produces the phased DEL/INS VCFs being genotyped) — https://github.com/maiziezhoulab/VolcanoSV
- **rasusa** (Subsamples HG002 long-read BAMs to 40x, 30x, 20x, 10x, and 5x coverage to generate genotype-accuracy test points)
- **Dipcall** (Assembly-based baseline caller whose genotypes are compared in the same benchmark) — https://github.com/lh3/dipcall
- **SVIM-asm** (Assembly-based baseline caller whose genotypes are compared in the same benchmark) — https://github.com/eldariont/svim-asm
- **PAV** (Assembly-based baseline caller whose genotypes are compared in the same benchmark) — https://github.com/EichlerLab/pav

## Examples

```
truvari bench -f ref.fa --base giab_hg002_sv.vcf.gz --comp volcanosv_merged.vcf.gz -p 0.5 -P 0.5 -r 500 -S 30 -O 0.01 --includebed giab_HC_regions.bed
```

## Evaluation signals

- Across Hifi datasets, VolcanoSV reference values: average genotyping accuracy of 98.32% (insertions) and 99.01% (deletions), with average F1 of 91.03% and 94.19% — comparable tools should land near or below these values on the same subsampled data
- Across CLR datasets, expected average genotyping accuracy of 97.07% (insertions) and 98.58% (deletions) with F1 of 89.72% and 93.70%
- Genotype accuracy should remain high at 10x coverage for robust tools — a sharp drop from 20x to 10x indicates sensitivity to subsampling
- Truvari benchmark run must use the exact parameter set p=0.5, P=0.5, r=500, S=30, O=0.01 and be restricted to GIAB high-confidence regions, or results are not comparable
- Output table must contain one row per (tool, dataset, SV type, coverage) combination, including all 10x runs, with recall/precision/F1/GT accuracy columns

## Limitations

- The GIAB gold standard SV callset is not a complete set and could contain false positives, biasing GT accuracy estimates
- Genotyping accuracy cannot be meaningfully computed for TRAs; only F1 and GT_F1 are computed for INVs and DUPs
- Assembly-based small indel genotyping on ONT data has low precision for all three tools, so GT accuracy there is unreliable

## Evidence

- [results] An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"
- [results] Truvari parameter setting: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Hifi genotyping accuracy reference values: "Across Hifi datasets, VolcanoSV achieved the best average F1 (91.03% and 94.19%) and genotyping accuracy (98.32% and 99.01%) for insertions and deletions."
- [results] Subsampling coverage levels: "All three datasets were subsampled to 40x, 30x, 20x, 10x, and 5x coverage using rasusa (v0.6.0)37"
- [results] INV/DUP genotyping restriction: "We thus only calculated F1 and GT_F1 for INVs and DUPs"

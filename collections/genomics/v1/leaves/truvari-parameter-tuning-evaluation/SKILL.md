---
name: truvari-parameter-tuning-evaluation
description: Use when comparing SV calls from long-read callers (e.g., VolcanoSV,
  SVIM-asm, Sniffles2) against a GIAB or tumor-normal truth set and you need to justify
  Truvari parameter settings and metric choice per SV type.
when_to_use_negative:
- You need genotype accuracy for translocations — reciprocal TRA genotyping is ambiguous,
  so only recall should be used, not GT_F1
- The truth callset is incomplete (e.g., HCC1395 somatic SVs) — report recall only
  rather than precision/F1, which would be misleadingly low
- You are detecting or calling SVs themselves rather than evaluating existing callsets
  against a benchmark
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Truvari
  role: Benchmarking SV calls against truth callsets; parameters p, P, r, S, O tuned
    via grid search
- name: SURVIVOR
  role: Merges paired normal-tumor VCFs to produce somatic SV callsets prior to Truvari
    evaluation
- name: VolcanoSV
  role: SV caller under evaluation, output VCFs benchmarked with Truvari
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: SVIM-asm
  role: Assembly-based SV comparison caller benchmarked on the same BAMs
  repo: https://github.com/eldariont/svim-asm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:903427a0ec6eb331663df09fd9b1fd3f04e11e558958c63f6bce39e22b66283f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - DeBreak
  - MAMnet
  - NanoVar
  - PBSIM
  - SURVIVOR
  - SVDSS
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VISOR
  - VolcanoSV
  - cuteSV
  - pbsv
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# truvari-parameter-tuning-evaluation

## Summary

A method for benchmarking long-read SV caller outputs against truth/high-confidence callsets with Truvari (v4.0.0), including grid-searching the match parameters (p, P, r, S, O) and choosing type-appropriate metrics (F1/GT_F1 for inversions and duplications, recall-only for translocations and somatic HCC1395 SVs).

## When to use

Use when comparing SV calls from long-read callers (e.g., VolcanoSV, SVIM-asm, Sniffles2) against a GIAB or tumor-normal truth set and you need to justify Truvari parameter settings and metric choice per SV type.

## When NOT to use

- You need genotype accuracy for translocations — reciprocal TRA genotyping is ambiguous, so only recall should be used, not GT_F1
- The truth callset is incomplete (e.g., HCC1395 somatic SVs) — report recall only rather than precision/F1, which would be misleadingly low
- You are detecting or calling SVs themselves rather than evaluating existing callsets against a benchmark

## Inputs

- SV call VCFs from each tool (VolcanoSV, NanoVar, cuteSV, pbsv, Sniffles2, MAMnet, SVDSS, DeBreak, SVIM-asm)
- Truth/high-confidence callset VCF (e.g., GIAB HG002 SV gold standard; HCC1395 somatic SV callset)
- Paired normal-tumor VCFs for SURVIVOR somatic merging
- Truvari parameter grid specification (p, P in [0,1] step 0.1; r in [0,1000] bp step 100; O in [0,1] step 0.1)

## Outputs

- Truvari benchmark output directories per tool/parameter setting
- Per-tool recall, precision, F1, and GT_F1 values
- Aggregated performance table (complex_sv_performance.csv)
- Parameter-sensitivity curves showing metric stability across p/P/O/r settings

## How to apply

Run Truvari (v4.0.0) with the base parameter setting p = 0.5, P = 0.5, r = 500, O = 0.01 (plus S = 30 for the complex SV/somatic analysis), comparing each tool's VCF to the truth/high-confidence callset. To test robustness rather than relying on a single cutoff, grid-search the sensitivity parameters: vary p, P, and O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp, confirming metric rankings hold across settings. Restrict metric computation by SV type: compute F1 and GT_F1 only for inversions and duplications (genotyping reciprocal translocations is ambiguous), and plot recall only for translocations and for HCC1395 somatic SVs because the high-confidence callset is incomplete. For somatic benchmarking, first call somatic variants with SURVIVOR (v1.0.6) from paired normal-tumor VCFs, then benchmark the somatic callset. Aggregate per-tool recall/precision/F1/GT_F1 values into a final performance table (e.g., complex_sv_performance.csv).

## Related tools

- **Truvari** (Benchmarking SV calls against truth callsets; parameters p, P, r, S, O tuned via grid search)
- **SURVIVOR** (Merges paired normal-tumor VCFs to produce somatic SV callsets prior to Truvari evaluation)
- **VolcanoSV** (SV caller under evaluation, output VCFs benchmarked with Truvari) — https://github.com/maiziezhoulab/VolcanoSV
- **SVIM-asm** (Assembly-based SV comparison caller benchmarked on the same BAMs) — https://github.com/eldariont/svim-asm

## Examples

```
truvari bench -b truth.vcf.gz -c tool_calls.vcf.gz -o truvari_bench -f ref.fa -p 0.5 -P 0.5 -r 500 -S 30 -O 0.01
```

## Evaluation signals

- Truvari run completed with the specified parameter set (p = 0.5, P = 0.5, r = 500, S = 30, O = 0.01)
- F1 and GT_F1 are computed only for inversions and duplications; recall-only rows appear for translocations and HCC1395 somatic SVs
- Reproduced metrics confirm VolcanoSV outperforms SVIM-asm in recall across all libraries and complex SV types, with ~12% higher duplication recall on PacBio CLR and ~17% on ONT
- Metric rankings are stable across the parameter grid (p, P, O in 0–1 step 0.1; r in 0–1 kb step 100 bp)
- Per-tool results are aggregated into a final table (complex_sv_performance.csv) with recall/precision/F1/GT_F1 columns

## Limitations

- The GIAB gold standard SV callset is not a complete set and could contain false positives
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing evaluation against VolcanoSV
- Since the HCC1395 high-confidence callset is incomplete, only recall (not precision or F1) is meaningful for somatic complex SVs

## Evidence

- [results] Truvari default parameters and sensitivity grid: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Parameter robustness grid search: "we varied p, P, O from 0 to 1 in increments of 0.1, and r from 0 to 1 kb in increments of 100 bp"
- [results] Metric choice per SV type: "We thus only calculated F1 and GT_F1 for INVs and DUPs"
- [results] Recall-only for incomplete truth set: "Since this high-confidence callset is incomplete, we only plotted recall"
- [results] Truvari role and version: "An SV benchmarking tool, Truvari (v4.0.0)30, was employed to compare the SV calls of each tool with the GIAB SV gold standard"

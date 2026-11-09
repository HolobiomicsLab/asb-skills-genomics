---
name: somatic-sv-calling-from-tumor-normal-pairs
description: Use when you have paired tumor-normal long-read BAMs (PacBio CLR/ONT,
  e.g. HCC1395) and need to detect or benchmark somatic complex SVs (TRA, INV, DUP)
  against a tool like SVIM-asm.
when_to_use_negative:
- The goal is germline SV genotyping on diploid non-cancer samples — use the full
  VolcanoSV indel pipeline benchmarked against GIAB HG002, not the somatic tumor-normal
  SURVIVOR merging step
- The caller must be run in single-chromosome mode and translocation detection is
  required — translocations demand WGS BAM support and are not meaningful at single-chromosome
  level
- Comparison against Dipcall is intended for complex SVs — Dipcall was excluded because
  it was not designed to detect complex SVs
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Hybrid SV detection pipeline (assembly + reference-based); its complex SV
    component (volcanosv-vc-complex-sv.py) is benchmarked for somatic TRA/INV/DUP
    calling
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: SVIM-asm
  role: Assembly-based comparison caller for somatic complex SVs; VolcanoSV's recall
    is compared against it across all libraries and SV types
  repo: https://github.com/eldariont/svim-asm
- name: SURVIVOR
  role: Merges paired normal-tumor VCFs (v1.0.6) to derive somatic SV calls
- name: Truvari
  role: Benchmarks SV calls against the high-confidence callset (v4.0.0) with parameters
    p=0.5, P=0.5, r=500, S=30, O=0.01
- name: VISOR
  role: Inserts SNPs and complex SVs (translocations, duplications, inversions) into
    the hg19 reference for simulated ground truth (v1.1.2)
- name: PBSIM
  role: Simulates long reads for each data type from the VISOR-mutated reference (v3.0.0)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:94dab3afd2fd413dd7fea8af906ac45342e4e3bd7ec001e330476e89b830f3a3
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
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# somatic-sv-calling-from-tumor-normal-pairs

## Summary

Benchmark somatic complex structural variant (translocation, inversion, duplication) detection from paired tumor-normal long-read (PacBio CLR or ONT) cancer datasets, using VolcanoSV against alignment-based and assembly-based callers, with somatic variants derived via SURVIVOR from paired normal-tumor VCFs and evaluated with Truvari.

## When to use

Use when you have paired tumor-normal long-read BAMs (PacBio CLR/ONT, e.g. HCC1395) and need to detect or benchmark somatic complex SVs (TRA, INV, DUP) against a tool like SVIM-asm.

## When NOT to use

- The goal is germline SV genotyping on diploid non-cancer samples — use the full VolcanoSV indel pipeline benchmarked against GIAB HG002, not the somatic tumor-normal SURVIVOR merging step
- The caller must be run in single-chromosome mode and translocation detection is required — translocations demand WGS BAM support and are not meaningful at single-chromosome level
- Comparison against Dipcall is intended for complex SVs — Dipcall was excluded because it was not designed to detect complex SVs

## Inputs

- Paired tumor-normal long-read BAM files (HCC1395 PacBio CLR and ONT WGS libraries)
- hg19 human reference FASTA (for simulation via VISOR)
- VolcanoSV large-indel VCF feeding the complex SV caller
- HCC1395 high-confidence somatic SV benchmark callset
- Per-tool germline/somatic VCFs from tumor and normal samples

## Outputs

- Somatic complex SV VCFs per tool per library (TRA, INV, DUP) from SURVIVOR normal-tumor merging
- Truvari benchmark reports (recall, precision, F1, GT_F1) per SV type and library
- Aggregated performance table (complex_sv_performance.csv)

## How to apply

Run each SV caller (VolcanoSV, NanoVar, cuteSV, pbsv, Sniffles2, MAMnet, SVDSS, DeBreak, SVIM-asm) per library on the tumor and normal BAMs; for VolcanoSV, execute volcanosv-asm.py followed by volcanosv-vc-large-indel.py in WGS mode (with -cl chrom_list.txt and -px prefix) so the large-indel VCF feeds volcanosv-vc-complex-sv.py — translocations require the whole-genome BAM, not single-chromosome mode. Call somatic variants with SURVIVOR (v1.0.6) from the paired normal-tumor VCFs. Benchmark against the HCC1395 high-confidence callset with Truvari (v4.0.0) using p=0.5, P=0.5, r=500, S=30, O=0.01, computing F1 and GT_F1 only for inversions and duplications (genotyping reciprocal TRAs is ambiguous) and recall only for somatic SVs since the benchmark callset is incomplete. For simulated validation, insert SNPs and complex SVs into hg19 with VISOR (v1.1.2), simulate reads with PBSIM (v3.0.0), and exclude Dipcall since it was not designed to detect complex SVs. Aggregate per-tool recall/precision/F1/GT_F1 into a performance table (complex_sv_performance.csv) and expect VolcanoSV to exceed SVIM-asm, with the largest duplication recall advantage (e.g. 12% higher on PacBio CLR, 17% higher on ONT).

## Related tools

- **VolcanoSV** (Hybrid SV detection pipeline (assembly + reference-based); its complex SV component (volcanosv-vc-complex-sv.py) is benchmarked for somatic TRA/INV/DUP calling) — https://github.com/maiziezhoulab/VolcanoSV
- **SVIM-asm** (Assembly-based comparison caller for somatic complex SVs; VolcanoSV's recall is compared against it across all libraries and SV types) — https://github.com/eldariont/svim-asm
- **SURVIVOR** (Merges paired normal-tumor VCFs (v1.0.6) to derive somatic SV calls)
- **Truvari** (Benchmarks SV calls against the high-confidence callset (v4.0.0) with parameters p=0.5, P=0.5, r=500, S=30, O=0.01)
- **VISOR** (Inserts SNPs and complex SVs (translocations, duplications, inversions) into the hg19 reference for simulated ground truth (v1.1.2))
- **PBSIM** (Simulates long reads for each data type from the VISOR-mutated reference (v3.0.0))

## Examples

```
chmod +x ${path_to_volcanosv}/bin/VolcanoSV-vc/Large_INDEL/volcanosv-vc-large-indel.py ${path_to_volcanosv}/bin/VolcanoSV-vc/Complex_SV/volcanosv-vc-complex-sv.py
```

## Evaluation signals

- Truvari run completed with the specified parameter set (p=0.5, P=0.5, r=500, S=30, O=0.01) on all libraries
- VolcanoSV recall exceeds SVIM-asm recall for every complex SV type (TRA, INV, DUP) on every PacBio CLR and ONT library, with the largest gap on duplications (~12% CLR, ~17% ONT)
- F1 and GT_F1 reported only for INVs and DUPs; only recall reported for translocations and HCC1395 somatic SVs
- Aggregated per-tool metrics written to complex_sv_performance.csv with recall/precision/F1/GT_F1 columns per tool
- On simulated data, VolcanoSV outperforms assembly-based tools for translocations, duplications, and inversions

## Limitations

- VolcanoSV's recall on real somatic complex SVs was relatively lower than on simulated data or indel SVs, and it failed to detect a sufficient number of unique somatic complex SVs beyond germline SVs in the tumor sample
- The HCC1395 high-confidence benchmark callset included SV calls only from alignment-based tools, potentially biasing evaluation against VolcanoSV, and is incomplete (hence recall-only reporting)
- Assembly-based SV calling may miss complex SVs because assembly algorithms prefer simplest paths through overlapping reads

## Evidence

- [abstract] VolcanoSV outperforms assembly-based tools in identifying complex SVs (translocations, duplications, inversions) in simulated and real cancer data.: "VolcanoSV outperforms assembly-based tools in the identification of complex SVs, including translocations, duplications, and inversions, in both simulated and real cancer data."
- [results] Somatic variants called with SURVIVOR from paired normal-tumor VCFs: "then used SURVIVOR (v1.0.6)36 to call somatic variants based on the paired normal-tumor VCFs"
- [results] Truvari benchmarking parameters: "The following metric/parameter setting was used in Truvari: p = 0.5, P = 0.5, r = 500, O = 0.01"
- [results] Recall-only reporting rationale for the incomplete HCC1395 callset: "Since this high-confidence callset is incomplete, we only plotted recall"
- [readme] Translocations require WGS BAM support, per the VolcanoSV README: "Note: since translocation detection requires WGS BAM file as support, it does not make sense to run it on single chromosome level."

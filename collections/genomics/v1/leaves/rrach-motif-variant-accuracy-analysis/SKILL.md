---
name: rrach-motif-variant-accuracy-analysis
description: Use when you already have per-tool TP/FP/FN bin calls from an m6A dRNA-seq
  benchmark and need to explain errors by motif variant or sequence context (e.g.
  reproducing Figure 5 feature characterization).
when_to_use_negative:
- Bin-level TP/FP/FN classifications from the benchmark step are not yet available
  (this skill consumes, not produces, them)
- You only need overall tool benchmark metrics (F1, AUPRC) without sequence-context
  stratification — the general benchmark workflow covers that
- The analysis targets non-m6A modifications or non-RRACH context motifs outside the
  12 RRACH variants
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3674
tools:
- name: NanOlympicsMod
  role: Provides the postprocessing and statistical_analysis R scripts used to annotate
    bins with RRACH variants and compute per-variant precision/recall/F1 and per-class
    sequence features
  repo: https://github.com/mfurla/NanOlympicsMod
- name: XSTREME (MEME Suite)
  role: De novo motif discovery and enrichment on false-positive bins to detect recurring
    sequence motifs driving FP calls
- name: RNAfold
  role: Predicts minimum free energy of the sequence context for each TP/FP/FN bin
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:4676b2893b8409422d47cac31417e054d19f965a0987de8e789975e5f7de65be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - RNAfold
  - XSTREME (MEME Suite)
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3674
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# rrach-motif-variant-accuracy-analysis

## Summary

Stratifies per-tool TP/FP/FN bin classifications from the m6A dRNA-seq benchmark by RRACH motif variant (R in {A,G}, A, C, H in {A,C,U} = 12 variants) and by sequence features (GC content, RNAfold minimum free energy, Shannon entropy) to reveal which sequence contexts make m6A easier or harder to detect.

## When to use

Use when you already have per-tool TP/FP/FN bin calls from an m6A dRNA-seq benchmark and need to explain errors by motif variant or sequence context (e.g. reproducing Figure 5 feature characterization).

## When NOT to use

- Bin-level TP/FP/FN classifications from the benchmark step are not yet available (this skill consumes, not produces, them)
- You only need overall tool benchmark metrics (F1, AUPRC) without sequence-context stratification — the general benchmark workflow covers that
- The analysis targets non-m6A modifications or non-RRACH context motifs outside the 12 RRACH variants

## Inputs

- Per-tool TP/FP/FN bin classifications from the m6A benchmark step
- Transcriptome FASTA file
- Gene/transcript BED annotation file
- False-positive bin sequences for XSTREME motif enrichment

## Outputs

- RRACH-variant accuracy table (precision, recall, F1 per tool per motif variant, 12 variants)
- Enriched false-positive motif list with significance values from XSTREME
- Per-bin-class (TP/FP/FN) sequence feature statistics table (GC content, RNAfold MFE, Shannon entropy) corresponding to Figure 5

## How to apply

Load the per-tool TP/FP/FN bin classifications from the preceding benchmark step and annotate each bin with its RRACH motif variant (R in {A,G}, A, C, H in {A,C,U} = 12 variants) using the transcriptome FASTA and gene/transcript BED annotation. Compute per-tool accuracy (precision, recall, F1) within each of the 12 RRACH variants using the NanOlympicsMod statistical_analysis R scripts from the Scripts folder of the repository. Run de novo motif discovery and enrichment on false-positive bins with XSTREME from the MEME Suite to detect recurring sequence motifs driving FP calls. For each bin class (TP, FP, FN), compute GC content, RNAfold predicted minimum free energy, and Shannon entropy of the sequence context using the NanOlympicsMod postprocessing R scripts. The rationale is that restricting and stratifying by RRACH motifs — one of the tested bin restrictions in the benchmark — exposes sequence-context biases common to all 14 detection tools.

## Related tools

- **NanOlympicsMod** (Provides the postprocessing and statistical_analysis R scripts used to annotate bins with RRACH variants and compute per-variant precision/recall/F1 and per-class sequence features) — https://github.com/mfurla/NanOlympicsMod
- **XSTREME (MEME Suite)** (De novo motif discovery and enrichment on false-positive bins to detect recurring sequence motifs driving FP calls)
- **RNAfold** (Predicts minimum free energy of the sequence context for each TP/FP/FN bin)

## Examples

```
git clone https://github.com/mfurla/NanOlympicsMod.git && Rscript Scripts/statistical_analysis.R --input per_tool_TP_FP_FN_bins.tsv --fasta transcriptome.fa --bed genes.bed
```

## Evaluation signals

- Accuracy table covers all 12 RRACH motif variants for every tool
- Enriched FP motifs are reported with significance from XSTREME
- Feature table reproduces the expected pattern: FN bins show lower GC content and higher free energy than TP bins
- FP bins show lower and more heterogeneous Shannon entropy than TP bins
- Per-variant precision/recall/F1 values are consistent with the overall per-tool benchmark metrics when aggregated

## Limitations

- Biological datasets (yeast, mouse, human) lack a ground truth for m6A positions; reference sets come from orthogonal short-read methods, so FP/FN labels may carry reference bias
- The pattern that all tools miss m6A hits in less structured (lower-GC, higher-free-energy) contexts is an aggregate observation; per-dataset behavior may differ
- Possible differences in bias between the orthogonal reference m6A sets and nanopore-based m6A techniques confound performance assessment

## Evidence

- [discussion] restricting to specific sets of bins, such as those associated with RRACH motifs: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"
- [readme] R scripts for post-processing and statistical analysis: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"
- [methods] cloning the NanOlympicsMod repository: "git clone https://github.com/mfurla/NanOlympicsMod.git"
- [intro] benchmarking against orthogonal reference m6A sets: "The results were benchmarked against reference m6A sets obtained from established orthogonal techniques."

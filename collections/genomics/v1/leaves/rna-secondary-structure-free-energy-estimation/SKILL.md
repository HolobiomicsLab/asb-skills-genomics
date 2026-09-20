---
name: rna-secondary-structure-free-energy-estimation
description: Use when per-tool TP/FP/FN m6A bin classifications from dRNA-seq benchmarking
  are available and the goal is to find sequence features (GC content, folding free
  energy, sequence entropy) that distinguish error classes.
when_to_use_negative:
- Input is raw read-level data (FAST5/FASTQ) without prior bin-level m6A tool calls
  — this skill starts from existing TP/FP/FN classifications
- The synthetic oligos dataset alone, which does not recapitulate in vivo m6A location/frequency
  and lacks confounding proximal RNA modifications, weakening feature-interpretation
- Evaluating tool performance vs. coverage depth or stringency cut-offs — those are
  separate benchmark analyses (saturation/AUPRC, default vs. relaxed thresholds)
edam_operation: http://edamontology.org/operation_0278
edam_topics:
- http://edamontology.org/topic_0097
tools:
- name: NanOlympicsMod
  role: Provides the postprocessing R scripts computing GC content, RNAfold minimum
    free energy and Shannon entropy per bin class, and the statistical_analysis R
    script computing per-RRACH-variant precision/recall/F1
  repo: https://github.com/mfurla/NanOlympicsMod
- name: XSTREME (MEME Suite)
  role: De novo motif discovery and enrichment on false-positive bins to detect recurring
    sequence motifs driving FP m6A calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:30ea912164392246df070a668dec751deab014b184c0a3da87c15ada3fec4c3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - XSTREME (MEME Suite)
  edam_operation: http://edamontology.org/operation_0278
  edam_topics:
  - http://edamontology.org/topic_0097
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# rna-secondary-structure-free-energy-estimation

## Summary

Characterizes m6A detection bins (true positive, false positive, false negative) from NanOlympicsMod dRNA-seq benchmarking by computing sequence-context features — GC content, RNAfold-predicted minimum free energy, and Shannon entropy — to identify sequence contexts where m6A marks are systematically easier or harder for tools to detect.

## When to use

Use when per-tool TP/FP/FN m6A bin classifications from dRNA-seq benchmarking are available and the goal is to find sequence features (GC content, folding free energy, sequence entropy) that distinguish error classes.

## When NOT to use

- Input is raw read-level data (FAST5/FASTQ) without prior bin-level m6A tool calls — this skill starts from existing TP/FP/FN classifications
- The synthetic oligos dataset alone, which does not recapitulate in vivo m6A location/frequency and lacks confounding proximal RNA modifications, weakening feature-interpretation
- Evaluating tool performance vs. coverage depth or stringency cut-offs — those are separate benchmark analyses (saturation/AUPRC, default vs. relaxed thresholds)

## Inputs

- Per-tool TP/FP/FN bin classification table from the NanOlympicsMod m6A benchmark step
- Transcriptome FASTA file
- Gene/transcript BED annotation file
- Bin-level sequence context windows for feature computation

## Outputs

- Per-bin-class feature statistics table (GC content, RNAfold minimum free energy, Shannon entropy for TP/FP/FN), corresponding to Figure 5
- RRACH-variant accuracy table (precision, recall, F1 per tool across the 12 RRACH motif variants)
- Enriched motif list for false-positive bins with significance (XSTREME / MEME Suite)

## How to apply

Load the per-tool TP/FP/FN bin classifications from the NanOlympicsMod benchmark step and annotate each bin with its RRACH motif variant (R in {A,G}, H in {A,C,U} = 12 variants) using the transcriptome FASTA and gene/transcript BED annotation. For each bin class, compute GC content, RNAfold minimum free energy of the sequence context, and Shannon entropy of the sequence context using the NanOlympicsMod postprocessing R scripts. Optionally run de novo motif discovery and enrichment on false-positive bins with XSTREME from the MEME Suite to detect recurring motifs driving FP calls. Assemble the per-bin-class feature statistics table (Figure 5): expect FN bins in lower-GC / higher-free-energy (less structured) contexts and FP bins with lower and more heterogeneous Shannon entropy.

## Related tools

- **NanOlympicsMod** (Provides the postprocessing R scripts computing GC content, RNAfold minimum free energy and Shannon entropy per bin class, and the statistical_analysis R script computing per-RRACH-variant precision/recall/F1) — https://github.com/mfurla/NanOlympicsMod
- **XSTREME (MEME Suite)** (De novo motif discovery and enrichment on false-positive bins to detect recurring sequence motifs driving FP m6A calls)

## Examples

```
git clone https://github.com/mfurla/NanOlympicsMod.git && Rscript Scripts/postprocessing/feature_stats.R --bins tp_fp_fn_bins.tsv --fasta transcriptome.fa
```

## Evaluation signals

- The RRACH-variant accuracy table covers all 12 RRACH variants (R in {A,G}, H in {A,C,U}) for every benchmarked tool
- Enriched false-positive motifs from XSTREME are reported with significance values
- The feature table reproduces the expected pattern: FN bins show lower GC content and higher free energy than TP bins
- FP bins show lower and more heterogeneous Shannon entropy than TP/FN bins
- Per-bin GC, free energy and entropy values are non-degenerate (vary across bins) and are computable for all three bin classes in every dataset

## Limitations

- The yeast, mouse and human datasets lack a ground truth for m6A positions; reference sets come from orthogonal short-read methods, so apparent sequence-feature biases may partly reflect bias differences between orthogonal references and nanopore-based techniques
- Low m6A stoichiometry in biological datasets only partially explains reduced F1, so feature-driven error patterns cannot be fully attributed to sequence context
- The oligos dataset lacks endogenous proximal RNA modifications that could confound sequence-feature associations in vivo

## Evidence

- [results] Across all tools, FN bins showed lower GC content and higher free energy than TP bins, indicating that all methods tend to miss m6A hits in less structured sequence contexts: [evidence span withheld — anchor status: not_found]
- [results] FP bins additionally showed lower and more heterogeneous Shannon entropy.: [evidence span withheld — anchor status: not_found]
- [results] Compute per-tool accuracy (precision, recall, F1) within each of the 12 RRACH motif variants: [evidence span withheld — anchor status: not_found]
- [readme] Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] differences in bias between the orthogonal reference m6A sets and nanopore based m6A techniques: "differences in bias between the orthogonal reference m6A sets and nanopore based m6A techniques"
- [methods] git clone https://github.com/mfurla/NanOlympicsMod.git: "git clone https://github.com/mfurla/NanOlympicsMod.git"

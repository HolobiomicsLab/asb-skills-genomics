---
name: rna-sequence-feature-computation
description: Use when per-tool TP/FP/FN bin classifications from an m6A dRNA-seq benchmark
  need to be explained by sequence context (motif variant, GC, structure, entropy).
when_to_use_negative:
- When no TP/FP/FN bin classifications from a benchmark run exist yet — features must
  be computed on already-classified bins
- When the goal is generic motif scanning of a transcriptome rather than explaining
  classifier error classes from the m6A benchmark
edam_operation: http://edamontology.org/operation_0236
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: NanOlympicsMod
  role: Provides the R postprocessing and statistical_analysis scripts that annotate
    RRACH variants and compute GC content, RNAfold MFE and Shannon entropy per bin
    class
  repo: https://github.com/mfurla/NanOlympicsMod
- name: XSTREME (MEME Suite)
  role: De novo motif discovery and enrichment on false-positive bins to detect recurring
    sequence motifs driving FP calls
- name: RNAfold
  role: Predicts the minimum free energy of each bin's sequence context as a structural
    feature
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:8f35ef5252b2b54cada41441e917e8830eb6c7d910f6832f0eec33585eec7f10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - RNAfold
  - XSTREME (MEME Suite)
  edam_operation: http://edamontology.org/operation_0236
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# rna-sequence-feature-computation

## Summary

Characterizes m6A benchmarking bins (true-positive, false-positive, false-negative) by sequence features — RRACH motif variant, GC content, RNAfold minimum free energy, and Shannon entropy — plus de novo motif discovery on FPs, to explain which sequence contexts make m6A easier or harder to detect with dRNA-seq tools.

## When to use

Use when per-tool TP/FP/FN bin classifications from an m6A dRNA-seq benchmark need to be explained by sequence context (motif variant, GC, structure, entropy).

## When NOT to use

- When no TP/FP/FN bin classifications from a benchmark run exist yet — features must be computed on already-classified bins
- When the goal is generic motif scanning of a transcriptome rather than explaining classifier error classes from the m6A benchmark

## Inputs

- Per-tool TP/FP/FN bin classification table from the m6A detection benchmark step
- Transcriptome FASTA file
- Gene/transcript BED annotation
- Per-bin m6A calls (e.g. R script-produced hit tables with 'default' or 'relaxed' thresholds)

## Outputs

- RRACH-variant accuracy table (precision, recall, F1 per tool across all 12 motif variants)
- List of motifs enriched in false-positive bins with significance values (XSTREME/MEME Suite)
- Per-bin-class sequence feature statistics table (GC content, RNAfold MFE, Shannon entropy) corresponding to Figure 5

## How to apply

Load per-tool TP/FP/FN bin classifications from the preceding NanOlympicsMod benchmark step and annotate each bin with its RRACH motif variant (R in {A,G}, A, C, H in {A,C,U} = 12 variants) using the transcriptome fasta and gene/transcript BED annotation. Compute per-tool precision, recall and F1 within each of the 12 RRACH variants with the NanOlympicsMod statistical_analysis R scripts. Run de novo motif discovery and enrichment on false-positive bins with XSTREME (MEME Suite) to find motifs driving FP calls. For each bin class, compute GC content, RNAfold-predicted minimum free energy, and Shannon entropy of the sequence context using the NanOlympicsMod postprocessing R scripts. The rationale is that tool errors may be concentrated in specific sequence contexts — e.g. low-stoichiometry, less structured regions — rather than distributed uniformly.

## Related tools

- **NanOlympicsMod** (Provides the R postprocessing and statistical_analysis scripts that annotate RRACH variants and compute GC content, RNAfold MFE and Shannon entropy per bin class) — https://github.com/mfurla/NanOlympicsMod
- **XSTREME (MEME Suite)** (De novo motif discovery and enrichment on false-positive bins to detect recurring sequence motifs driving FP calls)
- **RNAfold** (Predicts the minimum free energy of each bin's sequence context as a structural feature)

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- Accuracy table covers all 12 RRACH motif variants for every benchmarked tool
- Enriched FP motifs from XSTREME are reported together with significance values
- Feature table reproduces the expected pattern: FN bins in lower-GC / higher-minimum-free-energy contexts than TP bins
- FP bins show lower and more heterogeneous Shannon entropy than TP bins
- Every bin in the input classification table is annotated with exactly one RRACH variant from transcriptome/BED coordinates

## Limitations

- Biological datasets (yeast, mouse, human) lack a true ground truth; reference m6A sets come from orthogonal short-read methods, so 'FN' bins may reflect reference-set bias toward structured/high-GC contexts rather than pure tool failure
- Low m6A stoichiometry in biological datasets is only partially responsible for reduced F1 scores, so sequence features may not fully account for residuals
- Differences in bias between orthogonal reference m6A sets and nanopore-based techniques confound interpretation of sequence-feature patterns

## Evidence

- [results] Across all tools, FN bins showed lower GC content and higher free energy than TP bins: [evidence span withheld — anchor status: not_found]
- [results] FP bins showed lower and more heterogeneous Shannon entropy: [evidence span withheld — anchor status: not_found]
- [discussion] restricting to specific bins such as RRACH motifs: "restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones"
- [readme] README: R scripts for post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"

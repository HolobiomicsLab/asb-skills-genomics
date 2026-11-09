---
name: denovo-motif-enrichment-analysis
description: Use when benchmarking m6A detection tools on dRNA-seq bins and you need
  to explain systematic false-positive or false-negative calls through sequence-context
  features such as RRACH variants, GC content, structure, or entropy.
when_to_use_negative:
- When the goal is overall tool ranking or precision/recall comparison across datasets
  rather than explaining sequence-context-driven error patterns
- When evaluating performance on the synthetic oligos dataset where in vivo-like sequence
  contexts and confounding proximal modifications are absent
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_3524
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline whose statistical_analysis and postprocessing R scripts
    compute RRACH-variant accuracy and per-bin-class GC content, RNAfold free energy,
    and Shannon entropy
  repo: https://github.com/mfurla/NanOlympicsMod
- name: XSTREME (MEME Suite)
  role: De novo motif discovery and enrichment on false-positive bins to detect recurring
    sequence motifs driving FP calls
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:f86ec2af585dc26c48cc78846771170d7eccb11b652b6165627d653ba627bc1f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - XSTREME (MEME Suite)
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# denovo-motif-enrichment-analysis

## Summary

Characterizes which sequence features make m6A marks easier or harder to detect with dRNA-seq tools by annotating TP/FP/FN bins with RRACH motif variants, computing per-motif accuracy, running de novo motif discovery on false positives with XSTREME, and comparing GC content, RNAfold minimum free energy, and Shannon entropy across bin classes.

## When to use

Use when benchmarking m6A detection tools on dRNA-seq bins and you need to explain systematic false-positive or false-negative calls through sequence-context features such as RRACH variants, GC content, structure, or entropy.

## When NOT to use

- When the goal is overall tool ranking or precision/recall comparison across datasets rather than explaining sequence-context-driven error patterns
- When evaluating performance on the synthetic oligos dataset where in vivo-like sequence contexts and confounding proximal modifications are absent

## Inputs

- Per-tool TP/FP/FN bin classifications from the m6A tool benchmark step
- Transcriptome FASTA file
- Gene/transcript BED annotation file
- Per-bin sequence context windows

## Outputs

- RRACH-variant accuracy table (precision, recall, F1 per tool across the 12 motif variants)
- List of de novo enriched motifs on false-positive bins with significance values
- Per-bin-class feature statistics table (GC content, RNAfold minimum free energy, Shannon entropy for TP/FP/FN), corresponding to Figure 5

## How to apply

Load the per-tool TP/FP/FN bin classifications from the NanOlympicsMod benchmark and annotate each bin with its RRACH motif variant (R in {A,G}, C, H in {A,C,U}; 12 variants) using the transcriptome FASTA and gene/transcript BED annotation. Compute per-tool precision, recall, and F1 within each of the 12 RRACH variants using the NanOlympicsMod statistical_analysis R scripts. Run de novo motif discovery and enrichment on false-positive bins with XSTREME from the MEME Suite to find recurring motifs driving FP calls. For each bin class (TP, FP, FN), compute GC content, RNAfold predicted minimum free energy, and Shannon entropy of the sequence context using the NanOlympicsMod postprocessing R scripts, then assemble the RRACH-variant accuracy table and per-bin-class feature statistics corresponding to Figure 5. The rationale is that sequence context systematically biases nanopore error-based and current-based m6A callers, so feature differences between bin classes reveal where tools over- or under-call.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline whose statistical_analysis and postprocessing R scripts compute RRACH-variant accuracy and per-bin-class GC content, RNAfold free energy, and Shannon entropy) — https://github.com/mfurla/NanOlympicsMod
- **XSTREME (MEME Suite)** (De novo motif discovery and enrichment on false-positive bins to detect recurring sequence motifs driving FP calls)

## Examples

```
Rscript NanOlympicsMod/Scripts/statistical_analysis.R --bins TP_FP_FN_classifications.tsv --motif RRACH  # then XSTREME on FP bin FASTA
```

## Evaluation signals

- Accuracy table covers all 12 RRACH motif variants for every benchmarked tool
- Enriched FP motifs are reported with significance measures
- Feature table reproduces the expected pattern: FN bins in lower-GC, higher-free-energy contexts than TP bins across all tools
- FP bins show lower and more heterogeneous Shannon entropy than TP/FN bins
- Per-tool precision, recall, and F1 are computable within each motif variant using the NanOlympicsMod statistical_analysis scripts

## Limitations

- Findings are drawn from biological datasets whose m6A references derive from orthogonal short-read methods, so reference bias may confound which bins are labeled TP/FP/FN and hence the feature patterns
- The pattern reflects the specific benchmarked tool set and datasets (oligos, yeast SK1, mouse, human) and may not generalize to other callers or species
- Lower m6A stoichiometry in biological datasets partially confounds the interpretation of FN-rich sequence contexts

## Evidence

- [methods] Run de novo motif discovery and enrichment on false-positive bins with XSTREME: [evidence span withheld — anchor status: not_found]
- [results] Across all tools, FN bins showed lower GC content and higher free energy: [evidence span withheld — anchor status: not_found]
- [methods] annotate each bin with its RRACH motif variant: [evidence span withheld — anchor status: not_found]
- [readme] a set of R scripts for data post-processing and statistical analysis: "Scripts: folder containing a bash script to run the nextflow pipeline, a set of R scripts for data post-processing and statistical analysis"
- [discussion] restricting to specific sets of bins, such as those associated with RRACH motifs: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"

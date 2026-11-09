---
name: benchmark-tool-error-characterization
description: Use when per-tool TP/FP/FN bin classifications from an m6A dRNA-seq benchmark
  exist and the goal is to explain tool errors via sequence features such as RRACH
  motif variants, GC content, RNAfold free energy, and Shannon entropy.
when_to_use_negative:
- No per-tool TP/FP/FN classifications exist yet (the benchmarking step must run first)
- The input is the synthetic oligos dataset alone, whose artificial design poorly
  recapitulates in vivo m6A location and frequency, so error-feature associations
  may not transfer
- The question is overall tool ranking or precision-recall trade-off tuning rather
  than explaining why tools err in specific sequence contexts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: NanOlympicsMod
  role: Nextflow pipeline providing the per-tool TP/FP/FN bin classifications and
    the R postprocessing/statistical_analysis scripts for RRACH-variant accuracy and
    feature computation
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine executing the containerized NanOlympicsMod pipeline that generates
    the bins being characterized
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:2167ad4cdd6f242b282f6cc43f4de57b359975cb97e0805db3f7c96679937619
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# benchmark-tool-error-characterization

## Summary

Characterizes the sequence features underlying true-positive, false-positive, and false-negative m6A bins produced by dRNA-seq detection tools benchmarked with NanOlympicsMod, revealing which sequence contexts (RRACH variants, GC content, secondary structure, entropy) make m6A easier or harder to detect.

## When to use

Use when per-tool TP/FP/FN bin classifications from an m6A dRNA-seq benchmark exist and the goal is to explain tool errors via sequence features such as RRACH motif variants, GC content, RNAfold free energy, and Shannon entropy.

## When NOT to use

- No per-tool TP/FP/FN classifications exist yet (the benchmarking step must run first)
- The input is the synthetic oligos dataset alone, whose artificial design poorly recapitulates in vivo m6A location and frequency, so error-feature associations may not transfer
- The question is overall tool ranking or precision-recall trade-off tuning rather than explaining why tools err in specific sequence contexts

## Inputs

- Per-tool TP/FP/FN bin classifications from the preceding NanOlympicsMod benchmark step
- Transcriptome fasta file
- Gene/transcript BED annotation
- Per-tool m6A hit tables filtered at 'default' or 'relaxed' thresholds

## Outputs

- RRACH-variant accuracy table (precision, recall, F1 per tool per motif variant, 12 variants x tools)
- Enriched false-positive motif list from XSTREME with significance values
- Per-bin-class sequence feature statistics table (GC content, RNAfold minimum free energy, Shannon entropy) reproducing Figure 5

## How to apply

Load the per-tool TP/FP/FN bin classifications from the NanOlympicsMod benchmark and annotate each bin with one of the 12 RRACH motif variants (R in {A,G}, H in {A,C,U}) using the transcriptome fasta and gene/transcript BED annotation. Compute per-tool precision, recall, and F1 within each RRACH variant using the NanOlympicsMod statistical_analysis R scripts, then run de novo motif discovery and enrichment on false-positive bins with XSTREME from the MEME Suite to find motifs driving FP calls. For each bin class (TP, FP, FN), compute GC content, RNAfold-predicted minimum free energy, and Shannon entropy of the sequence context using the NanOlympicsMod postprocessing R scripts, and assemble the per-bin-class feature table corresponding to Figure 5. The rationale is that systematic, feature-dependent error patterns (e.g. FNs in low-GC/high-free-energy contexts) indicate biases shared by all 14 tools rather than tool-specific noise, and can guide post-hoc filtering such as restricting to RRACH motifs or high-coverage bins.

## Related tools

- **NanOlympicsMod** (Nextflow pipeline providing the per-tool TP/FP/FN bin classifications and the R postprocessing/statistical_analysis scripts for RRACH-variant accuracy and feature computation) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine executing the containerized NanOlympicsMod pipeline that generates the bins being characterized)

## Examples

```
git clone https://github.com/mfurla/NanOlympicsMod.git && Rscript Scripts/statistical_analysis.R  # per-RRACH-variant precision/recall/F1 from TP/FP/FN bins
```

## Evaluation signals

- Accuracy table covers all 12 RRACH motif variants for every tool
- Enriched FP motifs from XSTREME are reported with significance values
- Feature table reproduces the expected pattern: FN bins show lower GC content and higher free energy than TP bins
- FP bins show lower and more heterogeneous Shannon entropy than TP bins
- Per-bin-class statistics match the published Figure 5 pattern when rerun on the same dataset

## Limitations

- Yeast, mouse and human datasets lack a ground truth; reference m6A sets come from orthogonal short-read methods, so differences in bias between orthogonal references and nanopore techniques confound TP/FP/FN labeling and hence the feature associations
- Low m6A stoichiometry in biological datasets only partially explains reduced F1 scores, so residual FN/FP causes remain unattributed to sequence features
- Restricting calls to RRACH motifs or exclusion-zone-free bins only improves performance if coverage is also high; only filtering for highly expressed bins significantly boosted precision and recall

## Evidence

- [discussion] We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones"
- [results] Across all tools, FN bins showed lower GC content and higher free energy than TP bins: [evidence span withheld — anchor status: not_found]
- [discussion] The source code for the NanOlympicsMod workflow, and for reproducing all the results included in this study, are available: "The source code for the NanOlympicsMod workflow, and for reproducing all the results included in this study, are available at the following GitHub repository: https://github.com/mfurla/NanOlympicsMod"
- [readme] a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome: "a set of R scripts for data post-processing and statistical analysis, and a python script to lift-over m6A peaks to SK1 reference genome"

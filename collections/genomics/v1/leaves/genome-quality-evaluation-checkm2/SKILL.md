---
name: genome-quality-evaluation-checkm2
description: Use when a set of reconstructed MAGs (from SemiBin2, SemiBin, VAMB, MetaBAT2,
  LRBinner, or any binner) needs completeness/contamination scoring to compare binners
  or select publication-quality bins.
when_to_use_negative:
- You need taxonomic classification of contigs or MAGs (use CAT/BAT from the CAT_pack
  instead of quality metrics)
- You need read mapping or abundance estimation (use samtools/strobealign rather than
  a genome quality checker)
- The input is raw reads or an unbinning assembly rather than a set of candidate bins
  to score
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: CheckM2
  role: Evaluates all reconstructed bins from every binner to obtain completeness
    and contamination estimates used for near-complete and high-quality bin classification
- name: SemiBin2
  role: Primary binner whose long-read bins (run with --sequencing-type=long_read
    and ensemble-based DBSCAN clustering) are scored against baselines
  repo: https://github.com/BigDataBiology/SemiBin/
- name: Flye
  role: Assembles long reads into contigs (with Racon/Medaka/Pilon polishing) that
    serve as the shared input for all binners prior to quality evaluation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:35d0fbe1df8db19f66641772aba515527e856cb105c99d49c0efcdb594345c4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CheckM2
  - Flye
  - SemiBin2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# Genome quality evaluation with CheckM2

## Summary

Evaluates metagenome-assembled genomes (MAGs) produced by binners such as SemiBin2 using CheckM2 to estimate completeness and contamination, then classifies bins into near-complete and high-quality tiers. This provides the quantitative basis for the SemiBin2 benchmark showing 13.1-26.3% more high-quality genomes than the second-best binner on long-read data.

## When to use

Use when a set of reconstructed MAGs (from SemiBin2, SemiBin, VAMB, MetaBAT2, LRBinner, or any binner) needs completeness/contamination scoring to compare binners or select publication-quality bins.

## When NOT to use

- You need taxonomic classification of contigs or MAGs (use CAT/BAT from the CAT_pack instead of quality metrics)
- You need read mapping or abundance estimation (use samtools/strobealign rather than a genome quality checker)
- The input is raw reads or an unbinning assembly rather than a set of candidate bins to score

## Inputs

- Reconstructed MAG bins (FASTA) from each binner over identical polished assemblies
- Polished long-read assemblies (Flye with Racon/Medaka/Pilon polishing) for PRJCA007414, PRJNA595610, PRJEB48021

## Outputs

- Per-bin completeness and contamination estimates from CheckM2
- Counts of near-complete bins (>=90% completeness, <=10% contamination) per binner per dataset
- Counts of high-quality bins (>=90% completeness, <=5% contamination) per binner per dataset
- Comparison table longread_benchmark_comparison.csv with SemiBin2 vs. second-best binner improvements

## How to apply

Run CheckM2 on the full set of bins produced by each binner over the same polished Flye assembly (with Racon/Medaka/Pilon polishing per the benchmark protocol for the PRJCA007414, PRJNA595610, and PRJEB48021 long-read projects). CheckM2 returns completeness and contamination estimates for every bin; classify bins as near-complete (>=90% completeness, <=10% contamination) and high-quality (>=90% completeness, <=5% contamination) and count bins per binner per dataset. Compare counts across binners on identical assemblies to compute each binner's percentage improvement over the second-best binner, verifying SemiBin2's reported 13.2%, 28.1%, and 14.8% more near-complete bins and 15.6%, 26.3%, and 13.1% more high-quality bins on human gut, sheep gut, and activated sludge, respectively. Write the full comparison table to longread_benchmark_comparison.csv.

## Related tools

- **CheckM2** (Evaluates all reconstructed bins from every binner to obtain completeness and contamination estimates used for near-complete and high-quality bin classification)
- **SemiBin2** (Primary binner whose long-read bins (run with --sequencing-type=long_read and ensemble-based DBSCAN clustering) are scored against baselines) — https://github.com/BigDataBiology/SemiBin/
- **Flye** (Assembles long reads into contigs (with Racon/Medaka/Pilon polishing) that serve as the shared input for all binners prior to quality evaluation)

## Evaluation signals

- Every bin output by every binner receives a CheckM2 completeness and contamination estimate (no missing rows)
- Bin classification thresholds are exactly >=90% completeness with <=10% contamination (near-complete) and <=5% contamination (high-quality)
- SemiBin2's improvement over the second-best binner reproduces 13.2%/28.1%/14.8% more near-complete bins and 15.6%/26.3%/13.1% more high-quality bins on human gut, sheep gut, and activated sludge
- longread_benchmark_comparison.csv contains the full binner-by-dataset comparison table with counts for all binner/dataset pairs
- All binners were run on identical polished assemblies so counts are directly comparable

## Limitations

- SemiBin2's operations are probabilistic — the tutorial README warns that 'Many of the operations of SemiBin are probabilistic and running it twice will return different results', so reproduced counts may vary across reruns
- Quality tiers depend on CheckM2's completeness/contamination estimates, which are predictions rather than ground truth
- Reported improvement percentages (13.1-26.3% more high-quality genomes) were measured on the three long-read projects with the specified Flye polishing protocol; other assemblies or polishing choices are not guaranteed to reproduce them

## Evidence

- [methods] Evaluate all reconstructed bins from every binner with CheckM2 to obtain completeness and contamination estimates.: [evidence span withheld — anchor status: not_found]
- [methods] Bin classification thresholds: [evidence span withheld — anchor status: not_found]
- [abstract] Long-read benchmark result: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [abstract] Short-read efficiency finding: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage"

---
name: binning-benchmark-percent-gain-computation
description: Use when comparing binning strategies on long-read metagenomic datasets
  where the only varying factor is the clustering step (ensemble vs single-epsilon
  DBSCAN), and per-environment high-quality bin counts must be converted into percentage
  gains.
when_to_use_negative:
- 'Short-read samples: the ensemble-based DBSCAN clustering is proposed specifically
  for long-read data; short-read benchmarks should not use this ensemble-vs-single-epsilon
  comparison'
- Comparisons against other binners (e.g. LRBinner, MetaBAT2, VAMB) rather than an
  internal ablation of SemiBin2's clustering strategy
- Runs where parameters other than the clustering strategy (training mode, model,
  contig set) differ, since the percentage-gain metric is only interpretable as a
  controlled single-factor comparison
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SemiBin2
  role: Performs the long-read binning under both clustering strategies (default ensemble-based
    DBSCAN and single-epsilon DBSCAN); installed as the SemiBin2 command as of version
    2.2
  repo: github.com/BigDataBiology/SemiBin
- name: samtools
  role: Used to map reads to the contig FASTA and sort the resulting BAM files that
    feed SemiBin2 binning
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:a45a73608d39e439e5e46827eb5b58528774b4df6b75f58faa98cf3467d05afc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SemiBin2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# binning-benchmark-percent-gain-computation

## Summary

Benchmarking protocol for quantifying the percentage gain in high-quality bins reconstructed by SemiBin2's ensemble-based DBSCAN clustering versus a best single-epsilon DBSCAN run on the five CAMI II long-read simulated environments (airways, gastrointestinal, oral, skin, urogenital).

## When to use

Use when comparing binning strategies on long-read metagenomic datasets where the only varying factor is the clustering step (ensemble vs single-epsilon DBSCAN), and per-environment high-quality bin counts must be converted into percentage gains.

## When NOT to use

- Short-read samples: the ensemble-based DBSCAN clustering is proposed specifically for long-read data; short-read benchmarks should not use this ensemble-vs-single-epsilon comparison
- Comparisons against other binners (e.g. LRBinner, MetaBAT2, VAMB) rather than an internal ablation of SemiBin2's clustering strategy
- Runs where parameters other than the clustering strategy (training mode, model, contig set) differ, since the percentage-gain metric is only interpretable as a controlled single-factor comparison

## Inputs

- CAMI II long-read simulated datasets for five environments (airways, gastrointestinal, oral, skin, urogenital)
- Contigs FASTA files from metagenomic assembly
- Sorted BAM files from mapping reads to contigs (samtools)
- bins_info.tsv tables (total basepairs, contig count, N50) for reconstructed bins
- Completeness/contamination estimates for reconstructed bins

## Outputs

- output_bins directories with final bins from ensemble-DBSCAN and single-epsilon DBSCAN runs per environment
- dbscan_ensemble_vs_single_epsilon_hq_bins.csv with per-environment high-quality bin counts and percentage gains

## How to apply

For each of the five CAMI II long-read environments, run SemiBin2 with --sequencing-type=long_read (bin_long workflow) using the default ensemble-based DBSCAN clustering to obtain final bins in output_bins. Re-run binning on the identical inputs (same contigs, same BAM mapping files produced by samtools, same training mode and model) with the best single-epsilon DBSCAN setting, so clustering strategy is the only varying factor. For each environment and strategy, count bins classified as high-quality by combining bins_info.tsv (total basepairs, contig count, N50) with completeness/contamination estimates of the reconstructed bins. Compute the per-environment gain as (ensemble - single-epsilon) / single-epsilon x 100; validate against the reported values of 78.6%, 37.8%, 51.7%, 66.4%, and 25.4% within ±2 percentage points and write per-environment counts and gains to dbscan_ensemble_vs_single_epsilon_hq_bins.csv.

## Related tools

- **SemiBin2** (Performs the long-read binning under both clustering strategies (default ensemble-based DBSCAN and single-epsilon DBSCAN); installed as the SemiBin2 command as of version 2.2) — github.com/BigDataBiology/SemiBin
- **samtools** (Used to map reads to the contig FASTA and sort the resulting BAM files that feed SemiBin2 binning) — https://github.com/samtools/samtools

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : -o SemiBin2
```

## Evaluation signals

- Per-environment gains computed as (ensemble - single-epsilon) / single-epsilon x 100 match the reported Fig. 4 values of 78.6%, 37.8%, 51.7%, 66.4%, and 25.4% within ±2 percentage points for all five CAMI II long-read environments
- The single-epsilon re-run holds training mode, model, and contig set identical to the ensemble run so that clustering strategy is the only varying factor
- High-quality bin counts are reproducible from bins_info.tsv (total basepairs, contig count, N50) combined with completeness/contamination estimates
- dbscan_ensemble_vs_single_epsilon_hq_bins.csv contains counts and gains for all five environments with no missing rows

## Limitations

- SemiBin operations are probabilistic; running it twice can return different results, so single-epsilon baselines should be selected as the best run when comparing
- The reported gains are specific to simulated CAMI II long-read environments and may not transfer to real samples
- The claimed 13.1-26.3% improvement for ensemble DBSCAN is relative to the second best binner, which is a different quantity from the ensemble-vs-single-epsilon gains computed here

## Evidence

- [abstract] we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data.: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [methods] When you install the SemiBin package, the `SemiBin2` command is available. As of version 2.2, only `SemiBin2` is installed.: [evidence span withheld — anchor status: not_found]
- [readme] SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : -o SemiBin: [evidence span withheld — anchor status: external_doc]

---
name: self-supervised-ablation-study
description: Use when you need to quantify how much a deep-learning training step
  (here, self-supervised learning) contributes to metagenomic binning quality by comparing
  the full method against an ablated variant on simulated ground-truth data.
when_to_use_negative:
- Your goal is simply to produce the best possible MAGs for a sample — run full SemiBin2
  with self-supervised learning directly rather than setting up a two-arm ablation
- You lack ground-truth genome labels, since AMBER-based high-quality bin counts at
  strain/species/genus levels cannot be computed on real datasets without reference
  genomes
- The comparison target is not a training-step ablation — e.g. comparing SemiBin2
  against other binners or against SemiBin1's semi-supervised learning requires a
  different benchmarking setup
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SemiBin2
  role: The binner under test; run in self-supervised mode via single_easy_bin for
    the full arm, and with train_self skipped (clustering directly on data.csv/data_split.csv)
    for the NoSemi ablation arm
  repo: https://github.com/BigDataBiology/SemiBin/
- name: samtools
  role: Used to sort and index the BAM files of reads mapped back to contigs, a prerequisite
    input for SemiBin2 binning
  repo: https://github.com/samtools/samtools
- name: AMBER
  role: Evaluates bins from both configurations, computing the number of high-quality
    bins at strain, species, and genus levels for short reads and total high-quality
    bins for long reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:7483b51c486c169d8eba37b22de9ccafd7284a4b539b6a7c096a402e0a49e842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - AMBER
  - SemiBin1
  - SemiBin2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# self-supervised-ablation-study

## Summary

A benchmarking skill that isolates the contribution of SemiBin2's self-supervised contrastive learning by running the full self-supervised mode against the NoSemi ablation (clustering directly on original data.csv/data_split.csv feature tables without train_self), evaluating both with AMBER on CAMI II short- and long-read datasets.

## When to use

Use when you need to quantify how much a deep-learning training step (here, self-supervised learning) contributes to metagenomic binning quality by comparing the full method against an ablated variant on simulated ground-truth data.

## When NOT to use

- Your goal is simply to produce the best possible MAGs for a sample — run full SemiBin2 with self-supervised learning directly rather than setting up a two-arm ablation
- You lack ground-truth genome labels, since AMBER-based high-quality bin counts at strain/species/genus levels cannot be computed on real datasets without reference genomes
- The comparison target is not a training-step ablation — e.g. comparing SemiBin2 against other binners or against SemiBin1's semi-supervised learning requires a different benchmarking setup

## Inputs

- Five CAMI II environment inputs: contigs FASTA assembled from simulated reads, sorted/indexed BAM files from mapping reads back to contigs (samtools)
- SemiBin2 feature tables (data.csv, data_split.csv) for the NoSemi ablation arm
- Sequencing-type flag per arm (short-read default; --sequencing-type=long_read)
- AMBER ground-truth labels for the simulated CAMI II genomes

## Outputs

- output_bins directories containing bins for each environment from both full SemiBin2 and NoSemi configurations
- AMBER evaluation of high-quality bins at strain, species, and genus levels (short reads) and total high-quality bins (long reads)
- self_supervised_ablation_results.tsv with per-environment and aggregate percentage increases of full SemiBin2 over the ablation variant

## How to apply

Prepare five CAMI II environments by assembling reads into contigs, mapping reads back to the contigs, and sorting/indexing BAM files with samtools. Run full SemiBin2 in self-supervised mode using single_easy_bin (short-read default; --sequencing-type=long_read for the long-read arm) on each environment, then run the ablation variant by skipping train_self and passing the original data.csv/data_split.csv feature tables directly to clustering, writing bins to output_bins. Evaluate bins from both configurations with AMBER, counting high-quality bins at strain, species, and genus levels for short reads and total high-quality bins for long reads. Compute the percentage increase of full SemiBin2 over NoSemi per environment and aggregate, and write results to self_supervised_ablation_results.tsv; the expected direction is that self-supervised learning recovers more high-quality bins, e.g. on average 21.6% more distinct high-quality strains, 18.4% more species, and 17.1% more genera on short reads, and 31.5% more high-quality bins on long reads.

## Related tools

- **SemiBin2** (The binner under test; run in self-supervised mode via single_easy_bin for the full arm, and with train_self skipped (clustering directly on data.csv/data_split.csv) for the NoSemi ablation arm) — https://github.com/BigDataBiology/SemiBin/
- **samtools** (Used to sort and index the BAM files of reads mapped back to contigs, a prerequisite input for SemiBin2 binning) — https://github.com/samtools/samtools
- **AMBER** (Evaluates bins from both configurations, computing the number of high-quality bins at strain, species, and genus levels for short reads and total high-quality bins for long reads)

## Examples

```
SemiBin2 single_easy_bin -i contigs.fasta -b *.bam -o output_bins --sequencing-type=long_read
```

## Evaluation signals

- self_supervised_ablation_results.tsv exists and contains both per-environment and aggregate comparisons for both sequencing types
- Full SemiBin2 shows a positive percentage increase over NoSemi in high-quality bins for every environment arm (expected direction: self-supervised learning recovers more high-quality bins)
- Short-read aggregate percentages are consistent with the reported averages: ~21.6% more high-quality strains, ~18.4% more species, ~17.1% more genera
- Long-read aggregate shows ~31.5% more high-quality bins for full SemiBin2 over NoSemi
- The NoSemi arm verifiably bypasses train_self: bins are produced by clustering the original data.csv/data_split.csv feature tables without learned embeddings

## Limitations

- Results are established on simulated CAMI II datasets with ground truth; the article's claim that self-supervised learning achieves better results spans simulated and real datasets, but the ablation percentages reported here are simulation-based
- SemiBin operations are probabilistic — rerunning can return different bins, so replicate runs may shift the exact percentage increases
- The NoSemi arm discards the learned feature embeddings, so it cannot be used to assess other SemiBin2 improvements (e.g. the ensemble-based DBSCAN clustering for long reads), only the self-supervised learning step

## Evidence

- [abstract] SemiBin2 self-supervised learning beats NoSemi: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs."
- [abstract] Self-supervised outperforms semi-supervised on simulated and real data: "In simulated and real datasets, we show that self-supervised learning achieves better results"
- [methods] Full pipeline run via single_easy_bin: "Run SemiBin2 for this sample using the `single_easy_bin` subcommand"
- [methods] train_self is the self-supervised training step replaced in the semi-supervised mode: "`train_semi` (replacing `train_self` from the self-supervised mode)"
- [readme] Benchmark invocation of SemiBin2 in self-supervised mode on BAM inputs: [evidence span withheld — anchor status: external_doc]

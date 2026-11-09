---
name: greedy-ensemble-bin-integration
description: Use when binning long-read sequencing contigs with SemiBin2 and you need
  to integrate candidate bins from multiple DBSCAN epsilon runs into a final non-overlapping
  bin set via greedy F1-score selection.
when_to_use_negative:
- Input is short-read data — SemiBin2 uses a different clustering path for short reads;
  the ensemble DBSCAN algorithm is specific to long-read binning
- You only need bins from a single clustering resolution — running DBSCAN at 12 epsilon
  values plus greedy integration is unnecessary overhead if a single epsilon suffices
- Contigs lack trained model embeddings (no data.csv/data_split.csv) — the clustering
  operates on learned deep-learning features, not raw sequence composition
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: Implements the ensemble DBSCAN clustering and greedy bin integration for long-read
    binning
  repo: https://github.com/BigDataBiology/SemiBin/
- name: DBSCAN
  role: Density-based clustering of contig embeddings at 12 epsilon values (0.01–0.55)
    via scikit-learn
- name: hmmer
  role: External dependency used to estimate completeness and contamination from single-copy
    marker genes
- name: scikit-learn
  role: Provides the DBSCAN implementation used per-epsilon
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:255b38fbeb81286208a8a574412b932b32fe520aa05744ac3eee3740f4e1efb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - DBSCAN
  - SemiBin2
  - hmmer
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# greedy-ensemble-bin-integration

## Summary

SemiBin2's ensemble-based DBSCAN clustering integration for long-read metagenomic binning: run DBSCAN at 12 epsilon values, score candidate bins with 107 single-copy marker genes, and merge per-epsilon results via a greedy best-F1 selection algorithm.

## When to use

Use when binning long-read sequencing contigs with SemiBin2 and you need to integrate candidate bins from multiple DBSCAN epsilon runs into a final non-overlapping bin set via greedy F1-score selection.

## When NOT to use

- Input is short-read data — SemiBin2 uses a different clustering path for short reads; the ensemble DBSCAN algorithm is specific to long-read binning
- You only need bins from a single clustering resolution — running DBSCAN at 12 epsilon values plus greedy integration is unnecessary overhead if a single epsilon suffices
- Contigs lack trained model embeddings (no data.csv/data_split.csv) — the clustering operates on learned deep-learning features, not raw sequence composition

## Inputs

- Trained SemiBin2 deep learning model feature files (data.csv, data_split.csv) with learned contig embeddings
- Long-read contigs FASTA used for binning (--sequencing-type=long_read)
- 107 prokaryotic single-copy marker gene models for hmmer-based scoring

## Outputs

- Final ensemble bins as gzipped FASTA files (one per bin) in output_bins
- bins_info.tsv with per-bin completeness/contamination/F1 statistics
- contig_bins.tsv mapping each contig to its accepted bin

## How to apply

Load the trained deep-learning model features (data.csv/data_split.csv) and the long-read contigs (with --sequencing-type=long_read). Run DBSCAN (scikit-learn) on the learned contig embeddings at 12 epsilon values spanning 0.01–0.55 to produce per-epsilon candidate bin sets. Score every candidate bin for completeness and contamination using 107 prokaryotic single-copy marker genes (e.g. via hmmer) and compute an F1 score per Eq. 4. Then apply the greedy Algorithm 1: iteratively accept the highest-F1 bin whose contamination does not exceed the threshold, sweeping thresholds 0.1–1.0, and remove contigs already assigned to an accepted bin until no qualifying bins remain. The rationale is that no single epsilon recovers all genomes, so the ensemble yields more high-quality bins than any single-epsilon run. Write final bins as gzipped FASTA per bin to output_bins along with bins_info.tsv and contig_bins.tsv.

## Related tools

- **SemiBin2** (Implements the ensemble DBSCAN clustering and greedy bin integration for long-read binning) — https://github.com/BigDataBiology/SemiBin/
- **DBSCAN** (Density-based clustering of contig embeddings at 12 epsilon values (0.01–0.55) via scikit-learn)
- **hmmer** (External dependency used to estimate completeness and contamination from single-copy marker genes)
- **scikit-learn** (Provides the DBSCAN implementation used per-epsilon)

## Examples

```
SemiBin2 single_easy_bin -i contigs.fasta -b mapped.bam --sequencing-type=long_read --self-supervised -o output_bins
```

## Evaluation signals

- The ensemble bin set yields more high-quality bins than any single-epsilon DBSCAN run (paper reports 13.1–26.3% more high-quality genomes than the second-best long-read binner)
- Each accepted bin's contamination does not exceed the swept threshold (thresholds swept 0.1–1.0)
- No contig is assigned to more than one accepted bin — contigs of accepted bins are removed before subsequent greedy selections
- Every candidate bin has completeness, contamination, and F1 scores computed from 107 single-copy marker genes (e.g. hmmer output)
- Output directory contains one gzipped FASTA per bin plus bins_info.tsv and contig_bins.tsv

## Limitations

- Applies only to long-read sequencing data; short-read binning in SemiBin2 uses a different clustering strategy
- SemiBin operations are probabilistic — running twice can return different results (per the tutorial README)
- Requires trained deep-learning model features; bins cannot be produced directly from raw contigs without the embedding step

## Evidence

- [abstract] Ensemble DBSCAN integration for long-read binning: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner"
- [abstract] Deep feature learning: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs."
- [methods] Greedy ensemble clustering workflow: [evidence span withheld — anchor status: not_found]
- [methods] Marker-gene based bin scoring: [evidence span withheld — anchor status: not_found]
- [methods] External dependencies including hmmer: "the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan)"
- [readme] Probabilistic reproducibility caveat: [evidence span withheld — anchor status: external_doc]

---
name: dbscan-epsilon-ensemble-clustering
description: Use when binning long-read sequencing data with SemiBin2 and you need
  the default ensemble DBSCAN strategy, or when comparing ensemble clustering against
  a single-epsilon DBSCAN baseline on CAMI II-style long-read environments.
when_to_use_negative:
- Input is short-read data — SemiBin2 uses its standard single-clustering path for
  short reads; ensemble DBSCAN is specific to long-read (--sequencing-type=long_read)
  binning.
- You need the fast SemiBin1-style semi-supervised route — the article states self-supervised
  SemiBin2 replaced it (as of v2.2 only the SemiBin2 command is installed).
- Contigs have not been assembled/mapped yet — the skill requires model feature embeddings
  derived from assembled contigs and read-mapping BAM files, not raw reads.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SemiBin2
  role: Implements the ensemble DBSCAN clustering, greedy bin selection, and long-read
    binning workflow (bin_long); invoked with --sequencing-type=long_read.
  repo: https://github.com/BigDataBiology/SemiBin/
- name: hmmer
  role: External dependency used to estimate completeness and contamination of candidate
    bins from the 107 single-copy marker genes.
- name: samtools
  role: Used to map/sort reads to contigs, producing BAM inputs, and available for
    bin-level validation counts.
  repo: https://github.com/samtools/samtools
- name: scikit-learn
  role: Provides the DBSCAN implementation run at 12 epsilon values (0.01–0.55) on
    the learned contig embeddings.
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:2981f787175e22207e1bc59d4acc1cc0bf742b14ff55d9d380863bfbcbfc4420
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - SemiBin2
  - hmmer
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# dbscan-epsilon-ensemble-clustering

## Summary

SemiBin2's ensemble-based DBSCAN clustering for long-read metagenomic binning: run scikit-learn DBSCAN at 12 epsilon values (0.01–0.55), score candidate bins with 107 single-copy marker genes, and merge them with a greedy best-F1 selection algorithm to outperform any single-epsilon run.

## When to use

Use when binning long-read sequencing data with SemiBin2 and you need the default ensemble DBSCAN strategy, or when comparing ensemble clustering against a single-epsilon DBSCAN baseline on CAMI II-style long-read environments.

## When NOT to use

- Input is short-read data — SemiBin2 uses its standard single-clustering path for short reads; ensemble DBSCAN is specific to long-read (--sequencing-type=long_read) binning.
- You need the fast SemiBin1-style semi-supervised route — the article states self-supervised SemiBin2 replaced it (as of v2.2 only the SemiBin2 command is installed).
- Contigs have not been assembled/mapped yet — the skill requires model feature embeddings derived from assembled contigs and read-mapping BAM files, not raw reads.

## Inputs

- Trained SemiBin2 deep-learning model features for long-read contigs (data.csv / data_split.csv embeddings)
- Long-read contigs FASTA file used for binning (run with --sequencing-type=long_read)
- Sorted BAM files of reads mapped to contigs (produced e.g. with samtools/ngless)
- 107 prokaryotic single-copy marker gene models for bin scoring

## Outputs

- output_bins directory with one gzipped FASTA file per ensemble bin
- bins_info.tsv with per-bin total basepairs, contig count, N50
- contig_bins.tsv mapping each contig to its assigned bin
- Comparison table (e.g. dbscan_ensemble_vs_single_epsilon_hq_bins.csv) when evaluating against single-epsilon runs

## How to apply

Load the trained deep learning model features (data.csv/data_split.csv) for long-read contigs supplied via --sequencing-type=long_read, then run DBSCAN on the learned contig embeddings at 12 epsilon values spanning 0.01–0.55 to generate per-epsilon candidate bin sets. Score each candidate bin by estimating completeness and contamination from 107 prokaryotic single-copy marker genes (via hmmer) and compute the F1 score per Eq. 4. Integrate per-epsilon results with the greedy Algorithm 1: iteratively accept the highest-F1 bin whose contamination does not exceed the threshold, sweeping thresholds 0.1–1.0 and removing contigs already assigned to an accepted bin. Rationale: no single epsilon is optimal across all environments, and the ensemble selection yielded 13.1–26.3% more high-quality genomes than the second-best long-read binner, and 25.4–78.6% more high-quality bins than the best single-epsilon run across the five CAMI II long-read environments (airways, gastrointestinal, oral, skin, urogenital).

## Related tools

- **SemiBin2** (Implements the ensemble DBSCAN clustering, greedy bin selection, and long-read binning workflow (bin_long); invoked with --sequencing-type=long_read.) — https://github.com/BigDataBiology/SemiBin/
- **hmmer** (External dependency used to estimate completeness and contamination of candidate bins from the 107 single-copy marker genes.)
- **samtools** (Used to map/sort reads to contigs, producing BAM inputs, and available for bin-level validation counts.) — https://github.com/samtools/samtools
- **scikit-learn** (Provides the DBSCAN implementation run at 12 epsilon values (0.01–0.55) on the learned contig embeddings.)

## Examples

```
SemiBin bin_long -i contigs.fasta -b *.bam --sequencing-type=long_read -o output_bins
```

## Evaluation signals

- The ensemble bin set must yield more high-quality bins than any single-epsilon DBSCAN run on the same inputs.
- On the five CAMI II long-read environments, per-environment high-quality-bin gains of (ensemble − single-epsilon)/single-epsilon × 100 should reproduce the reported 78.6%, 37.8%, 51.7%, 66.4%, and 25.4% within ±2 percentage points.
- Each accepted bin's F1 score is computed from completeness/contamination estimated with 107 single-copy marker genes; bins with contamination above the swept threshold (0.1–1.0) must not be accepted.
- Output directory must contain gzipped FASTA per bin plus bins_info.tsv and contig_bins.tsv; bins_info.tsv total basepairs/contig count/N50 should be consistent with contig_bins.tsv assignments.

## Limitations

- The ensemble approach is described and benchmarked for long-read data only; short-read samples use SemiBin2's regular clustering path.
- Bin quality judgments depend on single-copy marker gene completeness/contamination estimates (via hmmer), which can mis-score bins in heavily fragmented or divergent assemblies.
- SemiBin operations are probabilistic — running twice can return different bins, so exact reproduction of the comparison percentages requires fixing seeds/inputs and allowing a small tolerance (±2 percentage points).

## Evidence

- [abstract] we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data.: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [methods] When you install the SemiBin package, the `SemiBin2` command is available. As of version 2.2, only `SemiBin2` is installed.: [evidence span withheld — anchor status: not_found]
- [methods] the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan): "the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan)"
- [readme] SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2: [evidence span withheld — anchor status: external_doc]

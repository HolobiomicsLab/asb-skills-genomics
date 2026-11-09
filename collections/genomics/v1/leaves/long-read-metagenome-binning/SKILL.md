---
name: long-read-metagenome-binning
description: Use when binning long-read (e.g. PacBio/ONT) assembled contigs into MAGs
  and you want the ensemble clustering strategy that recovers more high-quality bins
  than any single-epsilon DBSCAN run or second- binner.
when_to_use_negative:
- 'Input is short-read data: SemiBin2 uses the standard single-clustering path, not
  the ensemble-based DBSCAN long-read strategy'
- You only have raw long reads without an assembled contigs FASTA file and sorted
  BAM alignments — assemble first (e.g. Flye) before binning
- You need bins from the NoSemi ablation setting (clustering directly on original
  features without self-supervised training) — that is a different variant, not the
  default long-read binning skill
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: 'Performs the long-read binning: self-supervised feature learning plus ensemble-based
    DBSCAN clustering with greedy final-bin selection (--sequencing-type=long_read)'
  repo: github.com/BigDataBiology/SemiBin
- name: samtools
  role: Sorts and indexes the BAM alignments of reads mapped to contigs, a required
    binner input
  repo: https://github.com/samtools/samtools
- name: hmmer
  role: Supports estimation of completeness and contamination from the 107 single-copy
    marker genes used to score candidate bins
- name: strobealign
  role: Fast abundance estimation (strobealign --aemb all-by-all) that can supply
    the co-abundance features for binning
  repo: https://github.com/ksahlin/strobealign
- name: Flye
  role: Long-read assembler used to produce the contigs FASTA that is the primary
    binning input in the benchmark protocol
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  - task_006
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:2cbcbdd8186751f62624bc4471f9c8514aea6f2d135d766887d23fdf28793fba
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Flye
  - SemiBin1
  - SemiBin2
  - hmmer
  - samtools
  - strobealign
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# long-read-metagenome-binning

## Summary

Reconstruct metagenome-assembled genomes (MAGs) from long-read assemblies using SemiBin2's ensemble-based DBSCAN clustering, which runs DBSCAN at 12 epsilon values (0.01–0.55), scores candidate bins with 107 single-copy marker genes, and greedily selects the best-F1 bins.

## When to use

Use when binning long-read (e.g. PacBio/ONT) assembled contigs into MAGs and you want the ensemble clustering strategy that recovers more high-quality bins than any single-epsilon DBSCAN run or second-best binner.

## When NOT to use

- Input is short-read data: SemiBin2 uses the standard single-clustering path, not the ensemble-based DBSCAN long-read strategy
- You only have raw long reads without an assembled contigs FASTA file and sorted BAM alignments — assemble first (e.g. Flye) before binning
- You need bins from the NoSemi ablation setting (clustering directly on original features without self-supervised training) — that is a different variant, not the default long-read binning skill

## Inputs

- Assembled long-read contigs FASTA file (e.g. Flye assembly with Racon/Medaka/Pilon polishing)
- Sorted and indexed BAM files of reads mapped back to the contigs (samtools)
- Optionally a data.csv/data_split.csv feature table from a trained deep learning model

## Outputs

- output_bins directory with gzipped FASTA file per reconstructed bin
- bins_info.tsv with per-bin total basepairs, contig count, and N50
- contig_bins.tsv mapping each contig to its assigned bin
- AMBER precision/recall/F1 and high-quality bin counts (simulated data) or CheckM2 completeness/contamination estimates (real data)

## How to apply

Prepare the contigs FASTA file (e.g. from Flye assembly) and sorted BAM alignments (samtools sort) as inputs. Run SemiBin2 in long-read mode with the `--sequencing-type=long_read` flag via the single_easy_bin / bin_long workflow, which internally trains the self-supervised deep learning model to produce contig embeddings. For long reads, SemiBin2 runs DBSCAN (scikit-learn) at 12 epsilon values spanning 0.01–0.55, estimates completeness and contamination for each candidate bin from 107 prokaryotic single-copy marker genes (via hmmer), computes an F1 score per bin, and applies greedy Algorithm 1: iteratively accept the highest-F1 bin (sweeping contamination thresholds 0.1–1.0) and remove its contigs until no qualifying bins remain. Evaluate the final bins with AMBER (simulated data with ground truth) or CheckM2 (real data, near-complete = >=90% completeness and <=10% contamination; high-quality = >=90% completeness and <=5% contamination). The rationale is that the ensemble step is what drives gains of 13.1–26.3% more high-quality genomes over the second-best long-read binner, and per-environment gains of 27.1%, 21.5%, 52.7%, 47.5%, and 14.4% on CAMI II airways, gastrointestinal, oral, skin, and urogenital environments.

## Related tools

- **SemiBin2** (Performs the long-read binning: self-supervised feature learning plus ensemble-based DBSCAN clustering with greedy final-bin selection (--sequencing-type=long_read)) — github.com/BigDataBiology/SemiBin
- **samtools** (Sorts and indexes the BAM alignments of reads mapped to contigs, a required binner input) — https://github.com/samtools/samtools
- **hmmer** (Supports estimation of completeness and contamination from the 107 single-copy marker genes used to score candidate bins)
- **strobealign** (Fast abundance estimation (strobealign --aemb all-by-all) that can supply the co-abundance features for binning) — https://github.com/ksahlin/strobealign
- **Flye** (Long-read assembler used to produce the contigs FASTA that is the primary binning input in the benchmark protocol)

## Examples

```
SemiBin2 single_easy_bin -i contigs.fasta -b *.sorted.bam --sequencing-type=long_read -o output_bins
```

## Evaluation signals

- Ensemble bin set yields more high-quality bins than any single-epsilon DBSCAN run; per-environment gains on CAMI II long reads should match 78.6%, 37.8%, 51.7%, 66.4%, and 25.4% within ~±2 percentage points
- AMBER evaluation on CAMI II long reads shows SemiBin2 reconstructing 27.1%, 21.5%, 52.7%, 47.5%, and 14.4% more high-quality bins than the second-best binner in airways, gastrointestinal, oral, skin, and urogenital environments
- On real long-read projects (PRJCA007414, PRJNA595610, PRJEB48021) CheckM2-based comparison reproduces 15.6%, 26.3%, and 13.1% more high-quality bins (>=90% completeness, <=5% contamination) than the second-best binner
- Output directory contains the expected artifacts: output_bins/*.gz per bin, bins_info.tsv (total basepairs, contig count, N50), and contig_bins.tsv
- Each accepted bin in the greedy selection has contamination not exceeding the swept threshold (0.1–1.0) and was scored against 107 single-copy marker genes

## Limitations

- The ensemble-based DBSCAN strategy is specific to long-read mode; short-read binning does not use the 12-epsilon ensemble
- SemiBin2's operations are probabilistic — running it twice can return different results (per the SemiBin tutorial)
- On simulated CAMI II data, validation requires ground-truth genomes and AMBER; on real data there is no ground truth, so near-complete/high-quality classification relies on CheckM2 estimates

## Evidence

- [abstract] we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [methods] Run SemiBin2 for this sample using the `single_easy_bin` subcommand: "Run SemiBin2 for this sample using the `single_easy_bin` subcommand"
- [methods] the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan): "the external command-line tools (samtools, bedtools, hmmer, mmseqs2, prodigal, fraggenescan)"
- [readme] SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2: [evidence span withheld — anchor status: external_doc]

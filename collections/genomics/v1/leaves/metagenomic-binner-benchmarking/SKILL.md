---
name: metagenomic-binner-benchmarking
description: Use when comparing metagenomic binning tools on real short-read (human
  gut, dog gut, ocean, soil) or long-read (human gut, sheep gut, activated sludge)
  projects to quantify gains in high-quality bin reconstruction.
when_to_use_negative:
- Input is simulated data only — the headline benchmarks and reported gains (8.3-21.5%
  vs SemiBin1, 13.1-26.3% vs second-best long-read binner) were established on real
  short- and long-read sequencing projects
- You need to bin a single small sample quickly rather than compare binners — the
  tutorial repository warns its toy dataset 'should not be used to draw any biological
  conclusions'
- Reads are not yet assembled and mapped — the workflow requires contig FASTA and
  sorted BAM inputs before any binner can be run
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: SemiBin2
  role: Primary binner run in self-supervised mode (multi_easy_bin/single_easy_bin)
    and long-read mode
  repo: https://github.com/BigDataBiology/SemiBin/
- name: SemiBin1
  role: Baseline binner run in semi-supervised mode (generate_cannot_links, train_semi,
    bin_short)
  repo: https://github.com/BigDataBiology/SemiBin/
- name: VAMB
  role: Short-read baseline binner on identical BAM inputs
- name: CheckM2
  role: Completeness/contamination estimation for long-read benchmark bins
- name: CheckM
  role: Completeness/contamination estimation for short-read benchmark bins
- name: GUNC
  role: Chimerism assessment used to classify high-quality bins
- name: Flye
  role: Long-read assembly into contigs with Racon/Medaka/Pilon polishing
- name: samtools
  role: Sorting BAM files of reads mapped to contigs
  repo: https://github.com/samtools/samtools
- name: mmseqs2
  role: Contig annotation against GTDB to generate cannot-link files for semi-supervised
    training
- name: strobealign
  role: Fast all-by-all abundance estimation for metagenomic binning
  repo: https://github.com/ksahlin/strobealign
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:76ad46dba2a638877148d4f5d7cfbc1d5b33ea9bddbaf04d2863e15b0489e584
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CheckM
  - CheckM2
  - Flye
  - GUNC
  - SemiBin1
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  - strobealign
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# metagenomic-binner-benchmarking

## Summary

A protocol for benchmarking metagenomic binners (SemiBin2 vs SemiBin1, VAMB, and long-read baselines) on real short- and long-read datasets, counting near-complete and high-quality MAGs with CheckM/CheckM2 and GUNC, and computing percentage improvements over the second-best binner.

## When to use

Use when comparing metagenomic binning tools on real short-read (human gut, dog gut, ocean, soil) or long-read (human gut, sheep gut, activated sludge) projects to quantify gains in high-quality bin reconstruction.

## When NOT to use

- Input is simulated data only — the headline benchmarks and reported gains (8.3-21.5% vs SemiBin1, 13.1-26.3% vs second-best long-read binner) were established on real short- and long-read sequencing projects
- You need to bin a single small sample quickly rather than compare binners — the tutorial repository warns its toy dataset 'should not be used to draw any biological conclusions'
- Reads are not yet assembled and mapped — the workflow requires contig FASTA and sorted BAM inputs before any binner can be run

## Inputs

- Contig FASTA files assembled from short-read or long-read metagenomic projects
- Sorted BAM files from mapping reads back to contigs
- Long-read sequencing projects (e.g., PRJCA007414, PRJNA595610, PRJEB48021) and short-read projects (82, 129, 109, 101 samples)
- Reference genomes (GTDB) for semi-supervised cannot-link generation

## Outputs

- Per-binner per-dataset counts of high-quality bins (>=90% completeness, <=5% contamination) and near-complete bins (>=90% completeness, <=10% contamination)
- Percentage improvement table of SemiBin2 over second-best binner (longread_benchmark_comparison.csv / benchmark_results.csv)
- Wilcoxon signed-rank test P-values comparing paired per-sample quality scores between binners

## How to apply

Assemble reads into contigs (e.g., Flye with Racon/Medaka/Pilon polishing for long reads) and map reads back with samtools to produce sorted BAM files as binner inputs. Run SemiBin2 using multi_easy_bin with self-supervised mode (or single_easy_bin for single samples) and compare against baselines such as SemiBin1 in semi-supervised mode (generate_cannot_links with mmseqs2/GTDB, train_semi, bin_short) and VAMB under recommended settings; for long reads run in --sequencing-type=long_read mode against six long-read baselines. Evaluate all bins with CheckM or CheckM2 for completeness/contamination and GUNC for chimerism, classifying high-quality bins (>=90% completeness, <=5% contamination) and near-complete bins (>=90% completeness, <=10% contamination). Count bins per binner per dataset, compute SemiBin2's percentage improvement over the second-best binner, and apply the Wilcoxon signed-rank test to paired per-sample quality scores for statistical significance. Expected targets: SemiBin2 reconstructs 8.3-21.5% more high-quality bins than SemiBin1 and 27.3-229.9% more than VAMB on short reads, and 13.1-26.3% more high-quality bins than the second-best long-read binner.

## Related tools

- **SemiBin2** (Primary binner run in self-supervised mode (multi_easy_bin/single_easy_bin) and long-read mode) — https://github.com/BigDataBiology/SemiBin/
- **SemiBin1** (Baseline binner run in semi-supervised mode (generate_cannot_links, train_semi, bin_short)) — https://github.com/BigDataBiology/SemiBin/
- **VAMB** (Short-read baseline binner on identical BAM inputs)
- **CheckM2** (Completeness/contamination estimation for long-read benchmark bins)
- **CheckM** (Completeness/contamination estimation for short-read benchmark bins)
- **GUNC** (Chimerism assessment used to classify high-quality bins)
- **Flye** (Long-read assembly into contigs with Racon/Medaka/Pilon polishing)
- **samtools** (Sorting BAM files of reads mapped to contigs) — https://github.com/samtools/samtools
- **mmseqs2** (Contig annotation against GTDB to generate cannot-link files for semi-supervised training)
- **strobealign** (Fast all-by-all abundance estimation for metagenomic binning) — https://github.com/ksahlin/strobealign

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- Short-read high-quality bin counts reproduce Fig. 5 targets (1678, 3776, 631, 254 bins for human gut, dog gut, ocean, soil) with SemiBin2 gains within 8.3-21.5% (vs SemiBin1) and 27.3-229.9% (vs VAMB)
- Long-read percentages reproduce 13.2%, 28.1%, 14.8% more near-complete bins and 15.6%, 26.3%, 13.1% more high-quality bins than the second-best binner on human gut, sheep gut, and activated sludge
- Wilcoxon signed-rank P-values on paired per-sample quality scores indicate statistically significant SemiBin2 improvement
- All binners are run on identical inputs (same contig FASTA and sorted BAM files) for a fair comparison
- Bin classification thresholds consistently applied: high-quality = >=90% completeness, <=5% contamination; near-complete = >=90% completeness, <=10% contamination; GUNC used to flag chimerism

## Limitations

- SemiBin operations are probabilistic: 'running it twice will return different results', so exact bin counts may vary between runs
- Reported percentage gains cover specific ranges (e.g., 27.3-229.9% vs VAMB) and may not transfer to environments outside human gut, dog gut, ocean, soil, sheep gut, or activated sludge
- The SemiBin_tutorial toy dataset is artificial and 'should not be used to draw any biological conclusions'

## Evidence

- [abstract] Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory usage in real short-read sequencing samples.: "Compared to SemiBin1, SemiBin2 can reconstruct 8.3-21.5% more high-quality bins and requires only 25% of the running time and 11% of peak memory"
- [abstract] we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data.: "resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data"
- [methods] Run SemiBin2 on each project using its default self-supervised training mode (multi_easy_bin for the multi-sample projects).: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [methods] Assess every reconstructed bin from the three binners with CheckM (completeness/contamination) and GUNC (chimerism) to classify high-quality bins per project.: "Run the contig annotations using mmseqs with GTDB and generate `cannot-link` file used in the semi-supervised deep learning model training"
- [readme] SemiBin2_benchmark README documents VAMB and SemiBin2 multi_easy_bin commands with --minfasta 200000 -m 2000 and fraggenescan ORF finder.: [evidence span withheld — anchor status: external_doc]
- [readme] Many of the operations of SemiBin are probabilistic and running it twice will return different results.: [evidence span withheld — anchor status: external_doc]

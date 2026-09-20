---
name: binner-performance-comparison
description: Use when reproducing or extending binner benchmark comparisons on simulated
  metagenomic datasets (e.g., CAMI II short- or long-read) where per-environment high-quality
  bin counts and AMBER precision/recall/F1 must be tabulated per tool.
when_to_use_negative:
- Input is a single sample with no ground-truth genomes available — AMBER-based high-quality
  bin comparison cannot be computed without reference binning
- Goal is to bin a new real-world metagenome rather than to compare binner performance
  on benchmark data with known ground truth
- Data are not assembled contigs plus sorted BAM alignments (e.g., raw FASTQ only)
  — binners require assembled contigs as input
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: Primary binner under evaluation; run in long-read mode via single_easy_bin/bin_long
    with --sequencing-type=long_read and self-supervised learning
  repo: https://github.com/BigDataBiology/SemiBin
- name: SemiBin
  role: Command-line tool wrapper used to invoke SemiBin2 subcommands (multi_easy_bin)
    in the benchmark
  repo: https://github.com/BigDataBiology/SemiBin
- name: samtools
  role: Used to sort BAM alignments provided as binner inputs
  repo: https://github.com/samtools/samtools
- name: VAMB
  role: Comparison binner run with --minfasta 200000 -m 2000 --cuda on identical contig/BAM
    inputs
- name: MetaBAT2
  role: Comparison binner run on identical per-environment contig and alignment inputs
- name: CONCOCT
  role: Comparison binner evaluated against CAMI II ground truth with AMBER
- name: strobealign
  role: Fast alternative for abundance estimation (strobealign --aemb all-by-all)
    usable in the binning pipeline
  repo: https://github.com/ksahlin/strobealign
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:e9d7670dd3dc11dae38336bd4febd66d54b2b6d2978206ffec124c96d7479123
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - CONCOCT
  - MetaBAT2
  - SemiBin
  - SemiBin1
  - SemiBin2
  - VAMB
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
n_claims: 14
---

# binner-performance-comparison

## Summary

Benchmark metagenomic binners (SemiBin2 vs MetaBAT2, MetaDecoder, VAMB, SemiBin1, CONCOCT, MetaBinner, LRBinner) on identical contig and alignment inputs, quantifying per-environment gains in high-quality bin counts via AMBER evaluation against ground-truth genomes.

## When to use

Use when reproducing or extending binner benchmark comparisons on simulated metagenomic datasets (e.g., CAMI II short- or long-read) where per-environment high-quality bin counts and AMBER precision/recall/F1 must be tabulated per tool.

## When NOT to use

- Input is a single sample with no ground-truth genomes available — AMBER-based high-quality bin comparison cannot be computed without reference binning
- Goal is to bin a new real-world metagenome rather than to compare binner performance on benchmark data with known ground truth
- Data are not assembled contigs plus sorted BAM alignments (e.g., raw FASTQ only) — binners require assembled contigs as input

## Inputs

- Assembled contig FASTA files per environment (e.g., CAMI II PacBio long-read assemblies)
- Sorted BAM files of reads mapped to contigs (samtools-sorted)
- CAMI II ground-truth genome assignments for AMBER evaluation
- Raw reads for mapping (short- or long-read sequencing)

## Outputs

- Output bin directories (e.g., output_bins) per binner per environment
- AMBER evaluation results: precision, recall, F1, and high-quality bin counts per binner
- Per-environment comparison summary table, e.g., amber_longread_comparison.csv

## How to apply

For each environment (e.g., the five CAMI II human-associated environments: airways, gastrointestinal, oral, skin, urogenital), prepare the assembled contig FASTA and sorted BAM alignments (samtools) as identical inputs for all binners. Run SemiBin2 in long-read mode via `single_easy_bin`/`bin_long` with `--sequencing-type=long_read`, and run the comparison binners (MetaBAT2, MetaDecoder, VAMB, SemiBin1, CONCOCT, MetaBinner; LRBinner for long reads) on the same inputs. Evaluate every binner's bins with AMBER against CAMI II ground-truth genomes to obtain precision, recall, F1, and high-quality bin counts. Tabulate high-quality bins per binner per environment and compute SemiBin2's percentage gain over the second-best binner (expected: 27.1%, 21.5%, 52.7%, 47.5%, 14.4% on CAMI II long-read). The SemiBin2_benchmark repository provides the reference commands per binner, e.g., VAMB with `--minfasta 200000 -m 2000 --cuda` and SemiBin2 via `multi_easy_bin ... --self-supervised`.

## Related tools

- **SemiBin2** (Primary binner under evaluation; run in long-read mode via single_easy_bin/bin_long with --sequencing-type=long_read and self-supervised learning) — https://github.com/BigDataBiology/SemiBin
- **SemiBin** (Command-line tool wrapper used to invoke SemiBin2 subcommands (multi_easy_bin) in the benchmark) — https://github.com/BigDataBiology/SemiBin
- **samtools** (Used to sort BAM alignments provided as binner inputs) — https://github.com/samtools/samtools
- **VAMB** (Comparison binner run with --minfasta 200000 -m 2000 --cuda on identical contig/BAM inputs)
- **MetaBAT2** (Comparison binner run on identical per-environment contig and alignment inputs)
- **CONCOCT** (Comparison binner evaluated against CAMI II ground truth with AMBER)
- **strobealign** (Fast alternative for abundance estimation (strobealign --aemb all-by-all) usable in the binning pipeline) — https://github.com/ksahlin/strobealign

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- SemiBin2 should reconstruct 27.1%, 21.5%, 52.7%, 47.5%, and 14.4% more high-quality bins than the second-best binner in the airways, gastrointestinal, oral, skin, and urogenital CAMI II long-read environments, respectively
- For long-read data, SemiBin2's ensemble-based DBSCAN clustering yields 13.1–26.3% more high-quality genomes than the second-best binner
- AMBER evaluation must produce precision, recall, and F1 for every binner's bins against CAMI II ground-truth genomes
- All binners must consume byte-identical contig FASTA and sorted BAM inputs per environment to ensure a fair comparison
- The summary file amber_longread_comparison.csv should contain high-quality bin counts per binner per environment matching the published percentages

## Limitations

- Benchmark conclusions are drawn from simulated datasets (CAMI II); real-sample behavior may differ
- SemiBin operations are probabilistic — running twice returns different results, so exact bin counts can vary between runs (noted in SemiBin_tutorial README)
- Long-read comparison results depend on running SemiBin2 with the --sequencing-type=long_read flag and its ensemble-based DBSCAN clustering; omitting these changes the expected gains

## Evidence

- [results] SemiBin2 outperformed all other binners on CAMI II long-read: [evidence span withheld — anchor status: not_found]
- [methods] Long-read pipeline: [evidence span withheld — anchor status: not_found]
- [results] Expected percentage gains: [evidence span withheld — anchor status: not_found]
- [abstract] Ensemble clustering gain: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [readme] Benchmark commands: [evidence span withheld — anchor status: external_doc]
- [readme] SemiBin2 benchmark command: [evidence span withheld — anchor status: external_doc]

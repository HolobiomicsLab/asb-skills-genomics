---
name: cami-benchmark-data-processing
description: 'Use when reproducing the CAMI II long-read binner comparison: per-environment
  assembled contig FASTA and sorted BAM inputs must be binned with SemiBin2 (--sequencing-type=long_read)
  and six comparison binners, then scored with AMBER.'
when_to_use_negative:
- The dataset is real (not CAMI II simulated) short-read data — use the self-supervised
  short-read workflow instead, as SemiBin2's long-read ensemble-based DBSCAN clustering
  is specific to long-read benchmarking
- Only a single binner is needed without a comparative benchmark against AMBER ground
  truth
- Input reads are not yet assembled — contig FASTA files are required inputs, so assembly
  must precede this skill
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: SemiBin2
  role: Primary binner run in long-read mode with --sequencing-type=long_read to obtain
    output_bins
  repo: https://github.com/BigDataBiology/SemiBin/
- name: SemiBin
  role: Command-line package providing the single_easy_bin/multi_easy_bin subcommands
    used for binning
  repo: https://github.com/BigDataBiology/SemiBin/
- name: samtools
  role: Used for sorting BAM alignments prior to binning
  repo: https://github.com/samtools/samtools
- name: BigDataBiology/SemiBin2_benchmark
  role: Benchmark code repository containing the binner invocation commands (SemiBin2,
    VAMB, LRBinner, MetaBAT2, etc.)
  repo: https://github.com/BigDataBiology/SemiBin2_benchmark
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:4d87491c5225ca0d724222dad393a0211a205a310dcb62e0a0a47a0755712cd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - BigDataBiology/SemiBin2_benchmark
  - SemiBin
  - SemiBin1
  - SemiBin2
  - samtools
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cami-benchmark-data-processing

## Summary

Reproduces the SemiBin2 binner benchmark on CAMI II simulated long-read datasets across five human-associated environments (airways, gastrointestinal, oral, skin, urogenital), comparing SemiBin2 against six binners with AMBER.

## When to use

Use when reproducing the CAMI II long-read binner comparison: per-environment assembled contig FASTA and sorted BAM inputs must be binned with SemiBin2 (--sequencing-type=long_read) and six comparison binners, then scored with AMBER.

## When NOT to use

- The dataset is real (not CAMI II simulated) short-read data — use the self-supervised short-read workflow instead, as SemiBin2's long-read ensemble-based DBSCAN clustering is specific to long-read benchmarking
- Only a single binner is needed without a comparative benchmark against AMBER ground truth
- Input reads are not yet assembled — contig FASTA files are required inputs, so assembly must precede this skill

## Inputs

- Assembled contig FASTA file per CAMI II long-read environment
- Sorted BAM alignment files per environment (reads mapped to contigs, sorted with samtools)
- CAMI II ground-truth genome profiles for AMBER evaluation

## Outputs

- Binned output_bins directories per binner per environment
- AMBER precision, recall, F1, and high-quality bin counts per binner
- amber_longread_comparison.csv summarizing per-environment high-quality bin counts and SemiBin2's percentage gain over the second-best binner

## How to apply

For each of the five CAMI II PacBio long-read environments, prepare the assembled contig FASTA and sorted BAM alignments (samtools) as binner inputs. Run SemiBin2 in long-read mode via the single_easy_bin pipeline with `--sequencing-type=long_read`, then run MetaBAT2, MetaDecoder, VAMB, SemiBin1, CONCOCT, and MetaBinner on the identical per-environment inputs. Evaluate every binner's bins with AMBER against the CAMI II ground-truth genomes to obtain precision, recall, F1, and high-quality bin counts per binner. Tabulate high-quality bins per binner per environment and compute SemiBin2's percentage gain over the second-best binner (expected: 27.1%, 21.5%, 52.7%, 47.5%, 14.4%), saving the summary as amber_longread_comparison.csv. The rationale is that SemiBin2's ensemble-based DBSCAN clustering yields 13.1-26.3% more high-quality genomes than the second-best long-read binner.

## Related tools

- **SemiBin2** (Primary binner run in long-read mode with --sequencing-type=long_read to obtain output_bins) — https://github.com/BigDataBiology/SemiBin/
- **SemiBin** (Command-line package providing the single_easy_bin/multi_easy_bin subcommands used for binning) — https://github.com/BigDataBiology/SemiBin/
- **samtools** (Used for sorting BAM alignments prior to binning) — https://github.com/samtools/samtools
- **BigDataBiology/SemiBin2_benchmark** (Benchmark code repository containing the binner invocation commands (SemiBin2, VAMB, LRBinner, MetaBAT2, etc.)) — https://github.com/BigDataBiology/SemiBin2_benchmark

## Examples

```
SemiBin single_easy_bin -i input.fasta -b *.bam --sequencing-type=long_read -o output_bins
```

## Evaluation signals

- AMBER outputs precision, recall, and F1 per binner when evaluated against CAMI II ground-truth genomes
- SemiBin2's high-quality bin counts exceed the second-best binner in all five environments by approximately 27.1% (airways), 21.5% (gastrointestinal), 52.7% (oral), 47.5% (skin), and 14.4% (urogenital)
- amber_longread_comparison.csv contains a per-environment, per-binner summary of high-quality bins
- Each binner ran on identical per-environment contig and sorted BAM inputs (a controlled comparison invariant)

## Limitations

- SemiBin operations are probabilistic; running it twice will return different results (per the tutorial README)
- Percentage-gain results apply specifically to CAMI II simulated PacBio long-read datasets and may not transfer to real data or other sequencing platforms
- The tutorial README warns that its toy dataset should not be used to draw biological conclusions

## Evidence

- [abstract] Ensemble-based DBSCAN clustering yields 13.1-26.3% more high-quality genomes than the second best binner for long-read data.: "we also propose ensemble-based DBSCAN clustering algorithm, resulting in 13.1-26.3% more high-quality genomes than the second best binner for long-read data."
- [methods] Map reads to the FASTA file generated in Step 1.: "**Step 2**: Map reads to the FASTA file generated in _Step 1_."
- [methods] Run SemiBin2 for this sample using the `single_easy_bin` subcommand.: "Run SemiBin2 for this sample using the `single_easy_bin` subcommand"
- [intro] SemiBin is a command tool for metagenomic binning with deep learning, handles both short and long reads.: "SemiBin is a command tool for metagenomic binning with deep learning, handles both short and long reads."
- [readme] The benchmark README provides the VAMB invocation used in the long/short-read comparisons.: [evidence span withheld — anchor status: external_doc]

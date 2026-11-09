---
name: cami2-simulated-metagenome-handling
description: Use when benchmarking metagenomic binners on CAMI II simulated short-
  or long-read datasets, where per-sample FASTQs and ground-truth genomes are available
  for AMBER evaluation of high-quality bin recovery.
when_to_use_negative:
- Real (non-simulated) metagenomes without CAMI II ground truth — AMBER-based high-quality
  genus/species/strain recovery cannot be computed
- Single-sample binning with a prebuilt model — prebuilt models cannot be used in
  multi-sample mode, and this skill targets the multi-sample CAMI II benchmark
- Input is already a bin set or feature table — this skill covers the full assembly-through-evaluation
  benchmark pipeline, not downstream re-analysis
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SemiBin2
  role: Primary binner; run in self-supervised multi-sample mode via multi_easy_bin
    on combined contigs and BAMs (or single_easy_bin with --sequencing-type=long_read
    for long reads)
  repo: https://github.com/BigDataBiology/SemiBin/
- name: VAMB
  role: Competing multi-sample binner run on the same combined contigs and abundance
    information for head-to-head comparison
- name: AMBER
  role: Evaluates both bin sets against the CAMI II ground truth, computing high-quality
    genus, species, and strain recovery
- name: samtools
  role: Sorts and indexes the BAM files of reads mapped to the combined contig FASTA
  repo: https://github.com/samtools/samtools
- name: strobealign
  role: Alternative fast read mapper for generating abundance information (strobealign
    --aemb all-by-all) for multi-sample binning
  repo: https://github.com/ksahlin/strobealign
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:e6b5aeb61a3bd049c10d4e97409a21a56fd2ec8e639e0dd3a5ebedfaefecf72d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - AMBER
  - SemiBin1
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  - strobealign
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# cami2-simulated-metagenome-handling

## Summary

A benchmarking skill for evaluating metagenomic binners (SemiBin2, VAMB, and ablation variants) on the five CAMI II short- and long-read simulated environments, where the CAMI II ground truth enables AMBER-based evaluation of high-quality genus, species, and strain recovery.

## When to use

Use when benchmarking metagenomic binners on CAMI II simulated short- or long-read datasets, where per-sample FASTQs and ground-truth genomes are available for AMBER evaluation of high-quality bin recovery.

## When NOT to use

- Real (non-simulated) metagenomes without CAMI II ground truth — AMBER-based high-quality genus/species/strain recovery cannot be computed
- Single-sample binning with a prebuilt model — prebuilt models cannot be used in multi-sample mode, and this skill targets the multi-sample CAMI II benchmark
- Input is already a bin set or feature table — this skill covers the full assembly-through-evaluation benchmark pipeline, not downstream re-analysis

## Inputs

- CAMI II simulated short-read FASTQ files for five environments (and long-read equivalents)
- Per-sample assembled contig FASTA files (e.g. MEGAHIT output)
- Sorted, indexed BAM files of reads mapped to the combined contig FASTA
- CAMI II ground-truth genome assignments for AMBER evaluation

## Outputs

- SemiBin2 bins from multi_easy_bin (combined FASTA + BAMs, self-supervised mode)
- Comparator bins (e.g. VAMB) on identical contigs and abundance information
- AMBER evaluation tables of high-quality genus/species/strain recovery per binner
- Comparison table (e.g. self_supervised_ablation_results.tsv) with per-environment and aggregate percentage advantages

## How to apply

Assemble the reads of each of the five CAMI II environments into per-sample contig FASTA files (MEGAHIT for short reads), then concatenate them with SemiBin2's `concatenate_fasta` subcommand so contigs carry the sample name followed by a separator character; prebuilt models cannot be used in multi-sample mode, so self-supervised mode is required. Map each sample's reads back to the combined contigs (bowtie2 or strobealign), sort and index the BAMs with samtools, and run SemiBin2's `multi_easy_bin` (short reads) or `--sequencing-type=long_read` (long reads) alongside the competing binner (e.g. VAMB with the same combined FASTA and BAM inputs). For the NoSemi ablation, run `single_easy_bin` per environment and compare against a variant that skips `train_self`, feeding the original data.csv/data_split.csv feature tables directly to clustering. Evaluate all bin sets with AMBER against the CAMI II ground truth, counting distinct high-quality genera, species, and strains (total high-quality bins for long reads), and compute SemiBin2's average percentage advantage over the comparator per taxonomic level.

## Related tools

- **SemiBin2** (Primary binner; run in self-supervised multi-sample mode via multi_easy_bin on combined contigs and BAMs (or single_easy_bin with --sequencing-type=long_read for long reads)) — https://github.com/BigDataBiology/SemiBin/
- **VAMB** (Competing multi-sample binner run on the same combined contigs and abundance information for head-to-head comparison)
- **AMBER** (Evaluates both bin sets against the CAMI II ground truth, computing high-quality genus, species, and strain recovery)
- **samtools** (Sorts and indexes the BAM files of reads mapped to the combined contig FASTA) — https://github.com/samtools/samtools
- **strobealign** (Alternative fast read mapper for generating abundance information (strobealign --aemb all-by-all) for multi-sample binning) — https://github.com/ksahlin/strobealign

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- On CAMI II short-read datasets, the comparison table reports SemiBin2 averages of ~44.8% more distinct high-quality genera (range 14.3–73.7%), ~42.5% more species (range 26.1–59.1%), and ~47.1% more strains than VAMB, matching Fig. 2
- NoSemi ablation table shows full SemiBin2 recovering on average 21.6% more high-quality strains, 18.4% more species, and 17.1% more genera (short reads) and 31.5% more high-quality bins (long reads) than the ablation variant
- After concatenate_fasta, every contig name contains the sample name followed by a separator character
- All bin sets being compared were produced from identical combined contig FASTA and BAM inputs, so AMBER scores differ only by binner
- self_supervised_ablation_results.tsv exists and shows full SemiBin2 outperforming the ablation variant in every environment

## Limitations

- Prebuilt SemiBin models cannot be used in multi-sample mode; only self-supervised mode is applicable to the combined-FASTA workflow
- Many SemiBin operations are probabilistic: running it twice can return different results (per the SemiBin_tutorial README)
- The SemiBin_tutorial toy dataset 'should not be used to draw any biological conclusions' — only the full CAMI II environments support the reported benchmarks

## Evidence

- [methods] The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [methods] Concatenating the inputs can be done with the `concatenate_fasta` subcommand.: "Concatenating the inputs can be done with the `concatenate_fasta` subcommand."
- [methods] The contigs are renamed to include the sample name followed by a separator character.: "The contigs are renamed to include the sample name followed by a separator character."
- [readme] SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2: [evidence span withheld — anchor status: external_doc]
- [abstract] In simulated and real datasets, we show that self-supervised learning achieves better results: "In simulated and real datasets, we show that self-supervised learning achieves better results"

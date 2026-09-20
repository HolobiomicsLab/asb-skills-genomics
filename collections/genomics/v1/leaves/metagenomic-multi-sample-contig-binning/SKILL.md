---
name: metagenomic-multi-sample-contig-binning
description: Use when you have metagenomic short-read samples from multiple environments
  and want to bin contigs across samples jointly (multi-sample mode), comparing or
  replacing single-sample binners like VAMB.
when_to_use_negative:
- 'Single-sample data: use SemiBin2''s single_easy_bin instead; multi_easy_bin requires
  the combined contig file from several samples plus BAM files'
- You plan to use prebuilt taxonomic models — these cannot be used in multi-sample
  mode; self-supervised mode is required
- Input is already a set of finished bins or MAGs rather than raw reads/contigs needing
  binning
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: SemiBin2
  role: Multi-sample binner run via multi_easy_bin in self-supervised (default) mode
    on combined contigs and BAM files
  repo: https://github.com/BigDataBiology/SemiBin
- name: VAMB
  role: Competing multi-sample binner used for benchmarking on the same combined contigs
    and abundance information
  repo: BigDataBiology/SemiBin2_benchmark
- name: MEGAHIT
  role: Assembles each sample's reads into per-sample contig FASTA files
- name: samtools
  role: Sorts and indexes BAM files from read mapping to combined contigs
  repo: https://github.com/samtools/samtools
- name: AMBER
  role: Evaluates bin sets against CAMI II ground truth, computing high-quality genus/species/strain
    recovery
- name: mmseqs2
  role: Contig annotation used to generate cannot-link files for semi-supervised model
    training (single-sample variant of the workflow)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad209
    title: 'SemiBin2: self-supervised contrastive learning leads to better MAGs for
      short- and long-read sequencing'
  source_content_hash: sha256:84375c6306adba8009a5f9f06b40bdf83ae1eed25b5bcec2f2182ce9f5023275
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BigDataBiology/SemiBin
  tools:
  - AMBER
  - MEGAHIT
  - SemiBin2
  - VAMB
  - mmseqs2
  - samtools
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# metagenomic-multi-sample-contig-binning

## Summary

Multi-sample metagenomic binning workflow that concatenates per-sample contigs from several environments, maps all reads against the combined FASTA, and bins with SemiBin2's self-supervised contrastive learning (multi_easy_bin) to recover more high-quality genera/species/strains than VAMB.

## When to use

Use when you have metagenomic short-read samples from multiple environments and want to bin contigs across samples jointly (multi-sample mode), comparing or replacing single-sample binners like VAMB.

## When NOT to use

- Single-sample data: use SemiBin2's single_easy_bin instead; multi_easy_bin requires the combined contig file from several samples plus BAM files
- You plan to use prebuilt taxonomic models — these cannot be used in multi-sample mode; self-supervised mode is required
- Input is already a set of finished bins or MAGs rather than raw reads/contigs needing binning

## Inputs

- Per-sample metagenomic short-read sequencing files (FASTQ)
- Per-sample contig FASTA files assembled with MEGAHIT
- Combined (concatenated) contig FASTA with sample-name-separated contig IDs
- Sorted, indexed BAM files mapping each sample's reads to the combined contigs

## Outputs

- SemiBin2 metagenomic bins (multi-sample MAGs) in FASTA format
- Comparison metrics against a competing binner (e.g., VAMB): counts/percentages of distinct high-quality genera, species, and strains (AMBER)

## How to apply

Assemble reads of each sample (e.g., the five CAMI II short-read environments) into per-sample contig FASTA files with MEGAHIT, then merge them with SemiBin2's concatenate_fasta subcommand, which renames contigs with the sample-name separator. Map each sample's reads to the combined contig file with bowtie2/samtools to produce sorted, indexed BAM files. Run SemiBin2 multi_easy_bin in self-supervised mode (the default) on the combined FASTA and BAM files; note prebuilt models cannot be used in multi-sample mode. Evaluate the resulting bins against ground truth (e.g., with AMBER) and expect roughly 44.8% more distinct high-quality genera and 42.5% more high-quality species than VAMB on CAMI II short-read data.

## Related tools

- **SemiBin2** (Multi-sample binner run via multi_easy_bin in self-supervised (default) mode on combined contigs and BAM files) — https://github.com/BigDataBiology/SemiBin
- **VAMB** (Competing multi-sample binner used for benchmarking on the same combined contigs and abundance information) — BigDataBiology/SemiBin2_benchmark
- **MEGAHIT** (Assembles each sample's reads into per-sample contig FASTA files)
- **samtools** (Sorts and indexes BAM files from read mapping to combined contigs) — https://github.com/samtools/samtools
- **AMBER** (Evaluates bin sets against CAMI II ground truth, computing high-quality genus/species/strain recovery)
- **mmseqs2** (Contig annotation used to generate cannot-link files for semi-supervised model training (single-sample variant of the workflow))

## Examples

```
SemiBin multi_easy_bin -i input.fasta -b *.bam --orf-finder fraggenescan -s : --self-supervised -o SemiBin2
```

## Evaluation signals

- multi_easy_bin runs without error on the combined FASTA and all per-sample BAM files, producing bins with sample-name-separated contig IDs
- AMBER evaluation shows SemiBin2 recovering on average ~44.8% more distinct high-quality genera and ~42.5% more high-quality species than VAMB (per-environment ranges 14.3–73.7% and 26.1–59.1%)
- All per-sample reads are represented by sorted, indexed BAM files against the combined contig reference before binning
- Contigs in the combined FASTA carry sample-name separator tags produced by concatenate_fasta, confirming correct concatenation
- Bin quality thresholds: high-quality bins recovered per taxonomic level match values reported in Fig. 2 of the paper

## Limitations

- Prebuilt models cannot be used in multi-sample mode; only self-supervised training applies
- SemiBin operations are probabilistic — running twice returns different results (noted in the tutorial repository)
- All-by-all abundance estimation (or per-sample mapping) across samples adds compute cost; strobealign-aemb (v0.13+) is suggested as a fast alternative

## Evidence

- [results] multi-sample binning outperforms single-sample binning: [evidence span withheld — anchor status: not_found]
- [methods] multi_easy_bin input requirements: "The command `multi_easy_bin` requires the combined contig file from several samples, BAM files (reads mapping to the combined contig) as inputs"
- [methods] concatenate_fasta renaming: [evidence span withheld — anchor status: not_found]
- [readme] benchmark commands: [evidence span withheld — anchor status: external_doc]
- [abstract] self-supervised advantage: "We propose SemiBin2, which uses self-supervised learning to learn feature embeddings from the contigs. In simulated and real datasets, we show that self-supervised learning achieves better results"

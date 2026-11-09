---
name: alignment-free-scaffolding-graph-construction
description: Use when building a scaffolding graph from raw or error-corrected long
  reads (PacBio, Nanopore) or Illumina reads without a read aligner, or when extracting
  synthetic mate-pair libraries from long reads to feed short-read scaffolders.
when_to_use_negative:
- The task requires per-base read alignments or variant calls — Fast-SG only builds
  a scaffolding graph and extracts synthetic libraries, it does not align reads to
  contigs
- Only a single very short long-read library is available — synthetic library insert
  size is restricted to the actual long-read size, so long-range links cannot be extracted
- Gap-size estimation for long-range mate-pair technologies like 10X Genomics or DOVETAIL
  is the goal — linking information there comes from a range of insert sizes with
  possibly unknown pair orientation, unlike Fast-SG synthetic libraries
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Fast-SG
  role: Alignment-free algorithm that constructs the scaffolding graph and extracts
    synthetic mate-pair libraries from short or long reads; run via the FAST-SG.pl
    PERL wrapper
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: K-mer counter (kmc, kmc_dump) whose binaries must be placed in $Fast-SG/KMC/bin
    before compiling Fast-SG
  repo: https://github.com/refresh-bio/KMC
- name: PERL
  role: Language of the FAST-SG.pl wrapper that drives k-mer size, library file, contigs,
    and output prefix
- name: C++
  role: Implementation language of the Fast-SG core algorithm
provenance:
  source_task_ids:
  - task_002
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:d1cf4367ba3ffd2801d33b86c0120699616ad71310bf3e6bb7b981240166e009
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - C++
  - Fast-SG
  - KMC
  - Links
  - PERL
  - ScaffMatch
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# alignment-free-scaffolding-graph-construction

## Summary

Construct a scaffolding graph from short or long reads without aligning reads to contigs, using Fast-SG's ultrafast alignment-free algorithm with light-weight data structures (KMC k-mer counting, ntHash rolling hashes, BBHash minimal perfect hashing). This enables reuse of efficient short-read scaffolders on long-read data via synthetic mate-pair libraries.

## When to use

Use when building a scaffolding graph from raw or error-corrected long reads (PacBio, Nanopore) or Illumina reads without a read aligner, or when extracting synthetic mate-pair libraries from long reads to feed short-read scaffolders.

## When NOT to use

- The task requires per-base read alignments or variant calls — Fast-SG only builds a scaffolding graph and extracts synthetic libraries, it does not align reads to contigs
- Only a single very short long-read library is available — synthetic library insert size is restricted to the actual long-read size, so long-range links cannot be extracted
- Gap-size estimation for long-range mate-pair technologies like 10X Genomics or DOVETAIL is the goal — linking information there comes from a range of insert sizes with possibly unknown pair orientation, unlike Fast-SG synthetic libraries

## Inputs

- Uncorrected or corrected long-read FASTA (possibly gzipped), e.g. PacBio or Oxford Nanopore libraries
- Short-read Illumina FASTA/FASTQ (e.g. example/ecoli-illumina.fa.gz)
- Contig assembly FASTA/FASTA.gz (e.g. DiscovarDeNovo contigs for NA12878)
- Read configuration file (text) describing each library: type, ID, FASTA path, insert sizes, single-end flag

## Outputs

- Synthetic mate-pair libraries extracted alignment-free from the long reads (up to BAC-like insert sizes of 150-180 kb)
- Scaffolding graph built from short or long reads using light-weight data structures
- Scaffolds when the synthetic libraries are subsequently fed to a scaffolder (Opera-LG, ScaffMatch, Boss)

## How to apply

Clone Fast-SG (https://github.com/adigenova/fastsg) and place KMC3 binaries (kmc, kmc_dump) in $Fast-SG/KMC/bin, then compile with 'make all' and validate with 'make test' (tested with g++ 5.3 on Linux and clang 4.2 on Mac). Write a read configuration file listing each library as a space-separated line with type ('long' or Illumina), library ID, FASTA path, comma-separated insert sizes (e.g. 1000,2000,3000,5000), and a single-end flag, following the examples/ecoli-reads.txt format. Run the FAST-SG.pl wrapper with a k-mer size in the supported range [12-256] (e.g. -k 22 for human hybrid scaffolding, or a range such as -k 15-25:5), the read file (-l), the contig assembly in FASTA/FASTA.gz (-r), and an output prefix (-p). Fast-SG counts k-mers alignment-free and emits synthetic mate-pair libraries whose insert size is bounded by the actual long-read length — repeat extraction across insert-size ranges up to BAC-like sizes (150-180 kb) to generate up to ~20 libraries for long-range scaffolding. Feed the extracted synthetic libraries to short-read scaffolders (Opera-LG, ScaffMatch, Boss, Besst2) as a drop-in replacement for aligner-built scaffolding graphs.

## Related tools

- **Fast-SG** (Alignment-free algorithm that constructs the scaffolding graph and extracts synthetic mate-pair libraries from short or long reads; run via the FAST-SG.pl PERL wrapper) — https://github.com/adigenova/fast-sg
- **KMC** (K-mer counter (kmc, kmc_dump) whose binaries must be placed in $Fast-SG/KMC/bin before compiling Fast-SG) — https://github.com/refresh-bio/KMC
- **PERL** (Language of the FAST-SG.pl wrapper that drives k-mer size, library file, contigs, and output prefix)
- **C++** (Implementation language of the Fast-SG core algorithm)

## Examples

```
./FAST-SG.pl -k 22 -l reads.txt -r contigs.fa.gz -p prefix
```

## Evaluation signals

- 'make test' completes successfully after compilation, validating the Fast-SG build with a small test
- Extracted synthetic libraries respect the constraint that insert size is bounded by the actual long-read length
- library_stats.tsv records pair counts, linking percentage, and outlier percentage for every extracted library
- Scaffolders fed with Fast-SG synthetic libraries (Opera-LG, ScaffMatch, Boss) reach F-scores superior to or matching Links on E. coli K12 and S. cerevisiae W303 benchmarks
- Benchmark tables contain one row per genome-scaffolder combination with numeric F-score and error counts (misjoins, rearrangements)

## Limitations

- The synthetic library insert size is restricted to the actual long-read size, limiting maximum achievable link distance
- Fast-SG has no stated version number (only RRID SCR_015934); compilation was tested only with g++ 5.3 (Linux) and clang 4.2 (Mac OSX)
- Gap-size estimation behaves differently from long-range mate-pair technologies (10X Genomics, DOVETAIL) where insert sizes vary and pair orientation may be unknown

## Evidence

- [intro] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [abstract] Alignment-free graph construction claim: "we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
- [discussion] Insert size restricted to read length: "The insert size is restricted to the actual long-read size."
- [discussion] BAC-like insert sizes from ultralong Nanopore reads: "using ultralong Nanopore reads,Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes"
- [methods] Usage via FAST-SG.pl: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [readme] Compile and test instructions: "## Compile Fast-SG
	make all"
- [abstract] Superiority over aligners: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"

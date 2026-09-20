---
name: synthetic-mate-pair-library-extraction-from-long-reads
description: Use when you have raw or Lordec-corrected long reads (Nanopore/PacBio)
  plus an Illumina contig assembly in FASTA, and need synthetic mate-pair FASTQ libraries
  with user-specified insert sizes to drive short-read scaffolders alignment-free.
when_to_use_negative:
- The reads are already short Illumina mate-pair libraries — Fast-SG's synthetic extraction
  targets long-read linking information and is unnecessary
- You need a long-read-aware scaffolder to consume the original long reads directly
  (e.g. Links is the baseline for that; Fast-SG synthetic pairs are for short-read
  scaffolders)
- Only a raw read set with no contig assembly exists — extraction builds the scaffolding
  graph against a reference contig set (-r), so no contigs means the skill cannot
  be applied as described
edam_operation: http://edamontology.org/operation_0523
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Fast-SG
  role: Alignment-free extraction of synthetic mate-pair libraries and construction
    of the scaffolding graph; driven by the FAST-SG.pl PERL wrapper
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: K-mer counter supplying kmc/kmc_dump binaries that Fast-SG uses to index contig
    and read k-mers (k up to 256)
  repo: https://github.com/refresh-bio/KMC
- name: ScaffMatch
  role: Short-read scaffolder applied to the extracted synthetic libraries to produce
    hybrid scaffolds (DFS pipeline)
- name: Links
  role: Long-read scaffolding baseline used to benchmark scaffolding quality of Fast-SG
    synthetic libraries
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_005
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:83b0e1b9a851725f9b3a4ae1f251c328a91ea99b7457dfbed69ac832132e3f1d
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
  edam_operation: http://edamontology.org/operation_0523
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# synthetic-mate-pair-library-extraction-from-long-reads

## Summary

Extract alignment-free synthetic mate-pair libraries from raw or error-corrected long reads (ONT, PacBio) using Fast-SG, converting long-read linking information into short-read-style mate pairs that reusable short-read scaffolders (Opera-LG, ScaffMatch, Boss) can consume.

## When to use

Use when you have raw or Lordec-corrected long reads (Nanopore/PacBio) plus an Illumina contig assembly in FASTA, and need synthetic mate-pair FASTQ libraries with user-specified insert sizes to drive short-read scaffolders alignment-free.

## When NOT to use

- The reads are already short Illumina mate-pair libraries — Fast-SG's synthetic extraction targets long-read linking information and is unnecessary
- You need a long-read-aware scaffolder to consume the original long reads directly (e.g. Links is the baseline for that; Fast-SG synthetic pairs are for short-read scaffolders)
- Only a raw read set with no contig assembly exists — extraction builds the scaffolding graph against a reference contig set (-r), so no contigs means the skill cannot be applied as described

## Inputs

- Raw or hybrid error-corrected (e.g. Lordec) long-read libraries in FASTA/FASTA(.gz) (Oxford Nanopore R9.2, PacBio Sequel)
- Illumina contig assembly in FASTA or gzipped FASTA (e.g. DiscovarDeNovo contigs for E. coli K12, S. cerevisiae W303, NA12878)
- Space-separated long-read configuration file (type, library ID, path, comma-separated insert sizes, single-end flag)
- KMC3 binaries (kmc, kmc_dump) placed in $Fast-SG/KMC/bin

## Outputs

- Synthetic mate-pair libraries as FASTQ files (200 bp forward/reverse pairs at each user-specified insert size)
- Scaffolding graph built by Fast-SG keyed on the output prefix (-p)
- Per-library statistics (pair counts, linking percentage, outlier percentage, estimated insert size, standard deviation, orientation) in library_stats.tsv
- Recall metrics table (k-mer recall, mate-pair read recall, outlier percentage) per technology/correction state, e.g. ecoli_k12_recall_metrics.tsv

## How to apply

Author a space-separated read configuration file (#type libID Path(long read) Insert-sizes SAM) listing each long-read library, e.g. 'long pac <reads.fasta.gz> 1000,2000,3000,5000 1', modeled on examples/ecoli-reads.txt. Compile Fast-SG with 'make all' (g++ 5.3 on Linux / clang 4.2 on Mac) after placing KMC3 binaries (kmc, kmc_dump) in $Fast-SG/KMC/bin, and verify with 'make test'. Run FAST-SG.pl with a k-mer size in the supported range (e.g. -k 15 for E. coli K12, -k 22 for human NA12878; up to 256 supported via KMC), the read configuration (-l), the contig FASTA/FASTA.gz (-r), and an output prefix (-p); Fast-SG excises 200 bp forward/reverse pairs at each specified insert size using a 100 bp moving window until the read is fully scanned, with insert size restricted to the actual long-read length. For hybrid assembly, repeat extraction across an insert-size range up to BAC-like sizes (150-180 kb, e.g. 20 synthetic libraries for NA12878 ultralong Nanopore reads) and feed the synthetic pairs to ScaffMatch. Because extraction is alignment-free (k-mer hashing via KMC, ntHash, BBHash, quasi_dictionary), no read-to-contig alignment step is needed.

## Related tools

- **Fast-SG** (Alignment-free extraction of synthetic mate-pair libraries and construction of the scaffolding graph; driven by the FAST-SG.pl PERL wrapper) — https://github.com/adigenova/fast-sg
- **KMC** (K-mer counter supplying kmc/kmc_dump binaries that Fast-SG uses to index contig and read k-mers (k up to 256)) — https://github.com/refresh-bio/KMC
- **ScaffMatch** (Short-read scaffolder applied to the extracted synthetic libraries to produce hybrid scaffolds (DFS pipeline))
- **Links** (Long-read scaffolding baseline used to benchmark scaffolding quality of Fast-SG synthetic libraries)

## Examples

```
./FAST-SG.pl -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- 'make test' passes after compiling Fast-SG, confirming the KMC3 binaries and library build (kseqcpp, ntHash, BBHash, quasi_dictionary) are correctly installed
- Each synthetic pair consists of 200 bp forward/reverse reads separated by the user-specified insert size D, and no pair exceeds the actual long-read length (insert size restricted to actual read size)
- library_stats.tsv contains one row per configured library with pair counts, linking percentage, outlier percentage, estimated insert size, standard deviation, and orientation, with estimated insert sizes consistent with the user-specified values
- Recall/outlier metrics parse as percentages in [0, 100] with one row per technology/correction-state combination (e.g. raw vs Lordec-corrected ONT and PacBio E. coli K12)
- Synthetic pair counts scale with the requested insert-size range; ultralong Nanopore reads support BAC-like insert sizes of 150-180 kb

## Limitations

- Insert size is restricted to the actual long-read size, so requested insert sizes exceeding read lengths cannot be honored
- Synthetic mate-pair recall is substantially lower on raw reads than hybrid error-corrected reads (e.g. 49.42% raw ONT vs 75.12% corrected; 31.65% raw PacBio vs 65.02% corrected), so error correction strongly affects yield
- Gap-size estimation is more challenging with 10X Genomics and DOVETAIL technologies whose linking information spans a range of insert sizes with possibly unknown orientation; synthetic libraries sidestep this but require the Fast-SG workflow
- Compilation was only tested with g++ 5.3 (Linux) and clang 4.2 (Mac OSX)

## Evidence

- [abstract] we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures: "we propose a new method, called Fast-SG, that uses a new ultrafast alignment-free algorithm specifically designed for constructing a scaffolding graph using light-weight data structures"
- [discussion] synthetic libraries of BAC sizes: "Fast-SG is able to extract synthetic libraries of even bacterial artificial chromosome clone sizes with insert sizes of 150–180 kb"
- [discussion] insert size restricted to read length: "The insert size is restricted to the actual long-read size."
- [methods] read configuration file example: "Example of a read configuration file can be found in examples/ecoli-reads.txt"
- [readme] README invocation: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [readme] make test in README: "## Test Fast-SG (small test)
	make test"

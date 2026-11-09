---
name: aligner-runtime-measurement
description: Use when you must quantify precision, recall, and wall-clock runtime
  of short-read aligners against ground-truth simulated reads (e.g. Illumina reads
  simulated from GRCh38.p10) to justify an aligner or scaffolding-graph construction
  choice.
when_to_use_negative:
- Input reads are real (not simulated), so no ground truth exists against which alignment
  precision and recall can be computed
- The goal is a full assembly evaluation rather than measuring alignment/linking-recovery
  quality and aligner runtime
- Data are long reads (PacBio/Nanopore); this benchmark design compares state-of-the-art
  short-read aligners and Fast-SG on simulated Illumina data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Fast-SG
  role: Alignment-free method under benchmark; constructs the scaffolding graph from
    short or long reads and is timed against the aligners
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: k-mer counter used by Fast-SG; its kmc and kmc_dump binaries must be placed
    in Fast-SG/KMC/bin before compiling and running the benchmark arm
  repo: https://github.com/refresh-bio/KMC
- name: Bowtie
  role: Comparison short-read aligner; run on the identical simulated reads to build
    equivalent read-to-contig alignments
- name: Bowtie2
  role: Comparison short-read aligner run in both global and local mode on the same
    reads and contigs
- name: Bwa
  role: Comparison short-read aligner run on the identical simulated read sets
- name: Bwa-Mem
  role: Comparison aligner that achieved the highest precision (99.97%) in the benchmark
- name: g++ 5.3
  role: Compiler used to build Fast-SG on Linux before benchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:3d0f14b7bdf0f3d805773a6527d0aaf05ccbf33c4db47e44d17c344b6d9c6206
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Bowtie
  - Bowtie2
  - Bwa
  - Bwa-Mem
  - Fast-SG
  - KMC
  - PERL
  - g++
  - g++ 5.3
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# aligner-runtime-measurement

## Summary

Benchmark read-to-contig alignment quality and wall-clock runtime across short-read aligners (Bowtie, Bowtie2 global/local, Bwa, Bwa-Mem) and the alignment-free Fast-SG scaffolding-graph method, so that precision, recall, and speed can be compared on identical simulated reads and contigs.

## When to use

Use when you must quantify precision, recall, and wall-clock runtime of short-read aligners against ground-truth simulated reads (e.g. Illumina reads simulated from GRCh38.p10) to justify an aligner or scaffolding-graph construction choice.

## When NOT to use

- Input reads are real (not simulated), so no ground truth exists against which alignment precision and recall can be computed
- The goal is a full assembly evaluation rather than measuring alignment/linking-recovery quality and aligner runtime
- Data are long reads (PacBio/Nanopore); this benchmark design compares state-of-the-art short-read aligners and Fast-SG on simulated Illumina data

## Inputs

- Simulated paired-end Illumina FASTQ read libraries from GRCh38.p10 (or the target reference)
- Fast-SG read configuration file ('#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format)
- Contig/reference sequences in FASTA format (e.g. gzipped ecoli-illumina.fa.gz)
- Compiled Fast-SG with KMC3 binaries (kmc, kmc_dump) in Fast-SG/KMC/bin
- Installed Bowtie, Bowtie2, Bwa, Bwa-Mem for the comparison arm

## Outputs

- aligner_benchmark_table.csv with precision, recall, and runtime per tool x k-mer size
- Read-to-contig alignments (SAM) from each aligner for scaffolding-graph construction
- Recorded wall-clock runtime per tool and k-mer size

## How to apply

Simulate paired-end Illumina read libraries from the reference (GRCh38.p10) and describe them in a Fast-SG read configuration file using the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format. Run Fast-SG once per k-mer size via FAST-SG.pl with the four mandatory arguments (-k restricted to [12-256], range syntax such as 15-25:5; -r contigs in FASTA; -l read config; -p output prefix), after compiling Fast-SG with g++ and placing KMC3 binaries (kmc, kmc_dump) inside Fast-SG/KMC/bin. Run Bowtie, Bowtie2 in global and local mode, Bwa, and Bwa-Mem on the identical simulated read sets and contigs to build the equivalent read-to-contig alignments for the scaffolding graph. For each tool and k-mer size, evaluate alignment precision and recall of the recovered linking information against the truth from the simulated reads, and record wall-clock runtime, then aggregate into a per-tool, per-k table. Expected qualitative outcome: Fast-SG precision is high for any k-mer size (minimum 99.21%), superior to Bowtie2-local (98.17%) and comparable to Bowtie2-global (99.74%), Bwa-Mem leads (99.97%), and Fast-SG is 7X-14X faster than the next fastest aligner with recall (71.67% at k=25-30) comparable to Bowtie (71.52%).

## Related tools

- **Fast-SG** (Alignment-free method under benchmark; constructs the scaffolding graph from short or long reads and is timed against the aligners) — https://github.com/adigenova/fast-sg
- **KMC** (k-mer counter used by Fast-SG; its kmc and kmc_dump binaries must be placed in Fast-SG/KMC/bin before compiling and running the benchmark arm) — https://github.com/refresh-bio/KMC
- **Bowtie** (Comparison short-read aligner; run on the identical simulated reads to build equivalent read-to-contig alignments)
- **Bowtie2** (Comparison short-read aligner run in both global and local mode on the same reads and contigs)
- **Bwa** (Comparison short-read aligner run on the identical simulated read sets)
- **Bwa-Mem** (Comparison aligner that achieved the highest precision (99.97%) in the benchmark)
- **g++ 5.3** (Compiler used to build Fast-SG on Linux before benchmarking)

## Examples

```
./FAST-SG.pl -k 15-25:5 -l reads-config.txt -r contigs.fa.gz -p bench
```

## Evaluation signals

- aligner_benchmark_table.csv is complete: every tool x k-mer size combination has precision, recall, and runtime values
- Fast-SG precision stays above ~99% for all k-mer sizes (minimum 99.21%), beating Bowtie2-local (98.17%) and close to Bowtie2-global (99.74%)
- Fast-SG runtime is 7X-14X faster than the next fastest aligner on the identical read sets
- Fast-SG recall at k=25-30 (71.67%) is comparable to Bowtie (71.52%)
- All aligners and Fast-SG are run on the same simulated read sets and contigs, so comparisons are directly paired

## Limitations

- k-mer size is restricted to [12-256] and Fast-SG must be run once per k-mer size (or with range syntax such as 15-25:5)
- Benchmark conclusions are based on simulated Illumina reads from GRCh38.p10; Bwa-Mem still leads in precision (99.97%)
- Compilation was only tested with g++ 5.3 on Linux and clang 4.2 on Mac OSX, so runtime measurements depend on the build environment

## Evidence

- [abstract] Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [discussion] Fast-SG is faster than current short-read aligners: "Fast-SG is faster than the current state-of-the-art short-read aligners"
- [methods] Example Fast-SG invocation with mandatory arguments: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [methods] Using a range of k-mers for repeated Fast-SG benchmark runs: "./FAST-SG.pl  -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [discussion] Compilation environment tested for the benchmark builds: "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"

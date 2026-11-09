---
name: short-read-alignment-benchmarking
description: Use when the goal is to quantify how an alignment-free scaffolding-graph
  builder compares with Bowtie/Bowtie2/Bwa/Bwa-Mem on short-read data, via precision/recall
  of link recovery and wall-clock runtime per k-mer size.
when_to_use_negative:
- The goal is scaffolder output quality (F-score/error counts against a reference)
  rather than aligner precision/recall/runtime of link recovery
- The input is long reads (PacBio, Oxford Nanopore) — Fast-SG can extract synthetic
  libraries from raw or error-corrected long reads, but the short-read aligners benchmarked
  here are not the right comparators
- Only one graph-construction method is of interest with no aligner baseline — there
  is nothing to benchmark against
edam_operation: http://edamontology.org/operation_0448
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Fast-SG
  role: Alignment-free tool under test; builds the scaffolding graph from short reads
    via FAST-SG.pl wrapper
  repo: https://github.com/adigenova/fast-sg
- name: KMC
  role: K-mer counter whose binaries (kmc, kmc_dump) must be placed in Fast-SG/KMC/bin
    before compilation
  repo: https://github.com/refresh-bio/KMC
- name: Bowtie
  role: Short-read aligner benchmarked as a comparator on the identical simulated
    read sets and contigs
- name: Bowtie2
  role: Short-read aligner benchmarked in both global and local modes on the identical
    read sets and contigs
- name: Bwa / Bwa-Mem
  role: Short-read aligners benchmarked as comparators for scaffolding-graph construction
- name: g++
  role: Compiler used to build Fast-SG from source (tested with g++ 5.3 on Linux)
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:1a6306a0ea423c22d9c68f7c373cca8527d4fda4abdf1ff422bf5683881f9641
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Bowtie
  - Bowtie2
  - Bwa
  - Bwa / Bwa-Mem
  - Bwa-Mem
  - C++
  - Fast-SG
  - KMC
  - PERL
  - g++
  edam_operation: http://edamontology.org/operation_0448
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# short-read-alignment-benchmarking

## Summary

Benchmarks Fast-SG's alignment-free scaffolding-graph construction against state-of-the-art short-read aligners (Bowtie, Bowtie2 global/local, Bwa, Bwa-Mem) on identical read sets, comparing precision, recall, and runtime of the recovered linking information.

## When to use

Use when the goal is to quantify how an alignment-free scaffolding-graph builder compares with Bowtie/Bowtie2/Bwa/Bwa-Mem on short-read data, via precision/recall of link recovery and wall-clock runtime per k-mer size.

## When NOT to use

- The goal is scaffolder output quality (F-score/error counts against a reference) rather than aligner precision/recall/runtime of link recovery
- The input is long reads (PacBio, Oxford Nanopore) — Fast-SG can extract synthetic libraries from raw or error-corrected long reads, but the short-read aligners benchmarked here are not the right comparators
- Only one graph-construction method is of interest with no aligner baseline — there is nothing to benchmark against

## Inputs

- Paired-end short-read libraries (e.g. simulated Illumina reads from the GRCh38.p10 human reference) as FASTQ/FASTA
- Fast-SG read configuration file in the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format
- Contig sequences in FASTA format (passed via -r)
- Compiled Fast-SG with KMC3 binaries (kmc, kmc_dump) placed inside Fast-SG/KMC/bin

## Outputs

- aligner_benchmark_table.csv with one row per tool x k-mer combination, each holding precision, recall, and runtime values
- Per-tool read-to-contig alignments for scaffolding-graph construction

## How to apply

Simulate (or obtain) paired-end short-read libraries — e.g. simulated Illumina reads from GRCh38.p10 — and describe them in a Fast-SG read configuration file using the '#type libID Path(fwd) Path(rev) SAM(1:single 2:paired)' format. Compile Fast-SG from source with g++ (tested with g++ 5.3 on Linux) after placing the KMC3 binaries (kmc, kmc_dump) inside the Fast-SG/KMC/bin directory. Run FAST-SG.pl once per k-mer size with the four mandatory arguments: -k (restricted to [12-256], supporting range syntax such as 15-25:5), -r for contig sequences in FASTA, -l for the read configuration file, and -p for the output prefix. Run Bowtie, Bowtie2 in global and local mode, Bwa, and Bwa-Mem on the identical simulated read sets and contigs to build the equivalent read-to-contig alignments for the scaffolding graph. For each tool and k-mer size, evaluate alignment precision and recall of the recovered linking information against the truth from the simulated reads, record wall-clock runtime, and aggregate all three metrics per tool x k-mer into aligner_benchmark_table.csv.

## Related tools

- **Fast-SG** (Alignment-free tool under test; builds the scaffolding graph from short reads via FAST-SG.pl wrapper) — https://github.com/adigenova/fast-sg
- **KMC** (K-mer counter whose binaries (kmc, kmc_dump) must be placed in Fast-SG/KMC/bin before compilation) — https://github.com/refresh-bio/KMC
- **Bowtie** (Short-read aligner benchmarked as a comparator on the identical simulated read sets and contigs)
- **Bowtie2** (Short-read aligner benchmarked in both global and local modes on the identical read sets and contigs)
- **Bwa / Bwa-Mem** (Short-read aligners benchmarked as comparators for scaffolding-graph construction)
- **g++** (Compiler used to build Fast-SG from source (tested with g++ 5.3 on Linux))

## Examples

```
./FAST-SG.pl -k 15-25:5 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test
```

## Evaluation signals

- Completeness: aligner_benchmark_table.csv contains every tool x k-mer combination with precision, recall, and runtime values present
- Expected precision ranges on simulated human Illumina data: Fast-SG >= 99.21% for any k-mer size, Bowtie2-global ~99.74%, Bowtie2-local ~98.17%, Bwa-Mem ~99.97%
- Expected recall: Fast-SG recall at k=25-30 (~71.67%) is comparable to Bowtie (~71.52%)
- Expected runtime: Fast-SG is 7X-14X faster than the next fastest aligner in the comparison
- Consistency check: results reproduce the paper's qualitative claim that Fast-SG outperforms state-of-the-art short-read aligners when building the scaffolding graph

## Limitations

- The k-mer size passed to Fast-SG via -k is restricted to the range [12-256]
- Compilation was only tested with g++ 5.3 on Linux and clang 4.2 on Mac OSX
- Precision/recall evaluation depends on ground truth from simulated reads, so the benchmark is anchored to simulation rather than real-data truth
- No version number is stated for the Fast-SG software itself (only RRID SCR_015934)

## Evidence

- [abstract] Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph: "Fast-SG outperforms the state-of-the-art short-read aligners when building the scaffoldinggraph"
- [discussion] Fast-SG is faster than current state-of-the-art short-read aligners: "Fast-SG is faster than the current state-of-the-art short-read aligners"
- [methods] Run Fast-SG via FAST-SG.pl with k-mer size, library file, contigs, and output prefix: "./FAST-SG.pl  -k 15 -l example/ecoli-reads.txt -r example/ecoli-illumina.fa.gz -p test"
- [methods] Compilation was tested with g++ 5.3 (Linux) and clang 4.2 (Mac OSX): "Compilation was tested with g++ version 5.3 (Linux) and clang version 4.2 (Mac OSX)"
- [readme] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."

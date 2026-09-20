---
name: genome-polishing-runtime-benchmarking
description: Use when comparing the computational cost of polishing tools on the same
  HiFi-assembled genomes, or when reproducing the reported ~1-5x (real data) and ~10x
  (simulated data) speedup of NextPolish2 over alternative pipelines such as Racon
  + Merfin (RM).
when_to_use_negative:
- When the goal is accuracy evaluation (QV, overcorrection, switch errors) rather
  than computational cost — use Merqury/DeepVariant/Paftools.js-based evaluation workflows
  instead
- When the input assemblies are not HiFi-derived T2T/high-quality drafts and no >=60X
  short reads exist to build yak k-mer datasets, since NextPolish2 performance relies
  heavily on short-read k-mers
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: NextPolish2
  role: Tool under benchmark; runs with 5 threads on hifi.map.sort.bam, asm.fa.gz,
    k21.yak, k31.yak while runtime and peak memory are recorded
  repo: https://github.com/Nextomics/NextPolish2
- name: NextPolish
  role: Comparator polishing tool run under identical conditions to compute speedup
    and memory ratios
- name: winnowmap
  role: Maps HiFi reads to each assembly (with meryl k=15 repetitive k-mer filter,
    -W repetitive_k15.txt) to generate the shared BAM input
  repo: https://github.com/marbl/Winnowmap
- name: minimap2
  role: Alternative HiFi read mapper for preparing the shared alignment input when
    winnowmap is not used
  repo: https://github.com/lh3/minimap2/
- name: samtools
  role: Sorts winnowmap/minimap2 output into hifi.map.sort.bam for polishing
- name: yak
  role: Counts k-mer datasets (k21.yak, k31.yak) with -b37 from high-coverage short
    reads consumed by NextPolish2
  repo: https://github.com/lh3/yak
- name: fastp
  role: Quality control and filtering (adapter removal, trimming, -q 20) of short
    reads before yak k-mer counting
- name: meryl
  role: Counts k=15 assembly k-mers and prints k-mers greater than distinct=0.9998
    as the winnowmap repetitive k-mer filter
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gpbjnl/qzad009
    title: 'NextPolish2: A Repeat-aware Polishing Tool for Genomes Assembled Using
      HiFi Long Reads'
  source_content_hash: sha256:9809bfb9ed7479070588b6e948a7b0e0ff892bfe6274c156f23e0ba1059ec89b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Nextomics/NextPolish2
  tools:
  - NextPolish
  - NextPolish2
  - fastp
  - meryl
  - minimap2
  - samtools
  - winnowmap
  - yak
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 15
---

# genome-polishing-runtime-benchmarking

## Summary

Benchmarks wall-clock runtime and peak memory of genome polishing tools (NextPolish2 vs NextPolish, or NextPolish2 vs Racon+Merfin) on identical HiFi assemblies, using speedup and memory ratios as comparison metrics.

## When to use

Use when comparing the computational cost of polishing tools on the same HiFi-assembled genomes, or when reproducing the reported ~1-5x (real data) and ~10x (simulated data) speedup of NextPolish2 over alternative pipelines such as Racon + Merfin (RM).

## When NOT to use

- When the goal is accuracy evaluation (QV, overcorrection, switch errors) rather than computational cost — use Merqury/DeepVariant/Paftools.js-based evaluation workflows instead
- When the input assemblies are not HiFi-derived T2T/high-quality drafts and no >=60X short reads exist to build yak k-mer datasets, since NextPolish2 performance relies heavily on short-read k-mers

## Inputs

- HiFi long reads (FASTQ/FASTA, gzipped)
- Genome assemblies to polish (asm.fa.gz), including real (e.g. HG005, HG002, A. thaliana) and simulated datasets
- Sorted HiFi alignment BAM (hifi.map.sort.bam) from winnowmap or minimap2 + samtools sort
- Repetitive k-mer list from meryl (k=15, distinct=0.9998) for winnowmap -W
- yak k-mer count files (k21.yak, k31.yak) built with -b37 from >=60X fastp-cleaned short reads

## Outputs

- runtime_memory_benchmark.tsv with wall-clock runtime and peak memory per tool per dataset
- Per-dataset speedup ratios (NextPolish / NextPolish2 or RM / NextPolish2)
- Per-dataset memory usage ratios
- Polished assemblies (asm.np2.fa) produced under identical thread counts for fair comparison

## How to apply

Generate inputs identically for every polishing tool being compared: map HiFi reads to each assembly with winnowmap using a meryl k=15 repetitive k-mer filter (greater-than distinct=0.9998) or with minimap2, sort with samtools, and index the resulting hifi.map.sort.bam. Build yak k-mer datasets (k=21 and k=31, count with -b 37) from >=60X quality-filtered short reads cleaned with fastp (-5 -3 -n 0 -f 5 -F 5 -t 5 -T 5 -q 20). Run NextPolish2 with 5 threads on each BAM/assembly/k-mer tuple while recording wall-clock time and peak memory, then run the comparator tool (NextPolish, or the Racon + Merfin pipeline) on the same assemblies and inputs under identical conditions. Compute per-dataset speedup and memory ratios; the article reports NextPolish2 is ~1-5x faster than RM on real datasets and ~10x faster on a simulated dataset, using less memory on real datasets. Validate by confirming the benchmark table contains runtime and memory columns for both tools on every dataset and that measured speedups fall within the reported ranges.

## Related tools

- **NextPolish2** (Tool under benchmark; runs with 5 threads on hifi.map.sort.bam, asm.fa.gz, k21.yak, k31.yak while runtime and peak memory are recorded) — https://github.com/Nextomics/NextPolish2
- **NextPolish** (Comparator polishing tool run under identical conditions to compute speedup and memory ratios)
- **winnowmap** (Maps HiFi reads to each assembly (with meryl k=15 repetitive k-mer filter, -W repetitive_k15.txt) to generate the shared BAM input) — https://github.com/marbl/Winnowmap
- **minimap2** (Alternative HiFi read mapper for preparing the shared alignment input when winnowmap is not used) — https://github.com/lh3/minimap2/
- **samtools** (Sorts winnowmap/minimap2 output into hifi.map.sort.bam for polishing)
- **yak** (Counts k-mer datasets (k21.yak, k31.yak) with -b37 from high-coverage short reads consumed by NextPolish2) — https://github.com/lh3/yak
- **fastp** (Quality control and filtering (adapter removal, trimming, -q 20) of short reads before yak k-mer counting)
- **meryl** (Counts k=15 assembly k-mers and prints k-mers greater than distinct=0.9998 as the winnowmap repetitive k-mer filter)

## Examples

```
/usr/bin/time -v ./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa
```

## Evaluation signals

- runtime_memory_benchmark.tsv exists with runtime and peak-memory columns for both polishing tools on every dataset
- Measured NextPolish2 speedup over RM falls within ~1-5x on real datasets
- Measured NextPolish2 speedup on the simulated dataset falls near ~10x
- NextPolish2 uses less peak memory than the comparator on real datasets
- Both tools ran on byte-identical inputs (same hifi.map.sort.bam, asm.fa.gz, yak k-mer files) and the same thread count (5) so ratios are valid

## Limitations

- NextPolish2 performance relies heavily on short reads to check whether a K-mer contains errors, so benchmarks without >=60X short reads are not representative
- No software version numbers are stated for NextPolish2 or its dependencies, making exact reproduction of benchmark numbers version-sensitive
- Comparisons on the simulated dataset (~10x speedup) may not transfer to real datasets (~1-5x speedup)

## Evidence

- [discussion] NextPolish2 is a fast open-source polishing tool specifically developed for correcting errors in genomes assembled from HiFi long reads.: "NextPolish2 is a fast open-source polishing tool specifically developed for correcting errors in genomes assembled from HiFi long reads."
- [methods] winnowmap mapping and samtools sort command: "winnowmap -t 5 -W repetitive_k15.txt -ax map-pb asm.fa.gz hifi.fasta.gz|samtools sort -o hifi.map.sort.bam -"
- [methods] yak k-mer dataset preparation with >=60X short reads: "2. Prepare k-mer dataset files ([yak](https://github.com/lh3/yak)) (We recommend using >=60X short reads)."
- [methods] NextPolish2 invocation with 5 threads on BAM, assembly and yak k-mer files: "./target/release/nextPolish2 -t 5 hifi.map.sort.bam asm.fa.gz k21.yak k31.yak > asm.np2.fa"
- [methods] fastp quality filtering of short reads: "quality filtering steps (fastp) such as adapter removal, global or quality trimming, and read filtering are essential for short reads."
- [supplementary] benchmark test results availability: "The source code as well as results of the benchmark tests is freely available from https://github.com/Nextomics/NextPolish2"
- [readme] yak k-mer counting for high-coverage reads: "# build k-mer hash tables for high-coverage reads; discard singletons
./yak count -b37 -t32 -o ccs.yak ccs-reads.fq.gz"

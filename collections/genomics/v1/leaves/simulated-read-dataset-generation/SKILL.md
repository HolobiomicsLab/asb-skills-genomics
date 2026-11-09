---
name: simulated-read-dataset-generation
description: Use when benchmarking long-read mapper runtime or accuracy and controlled
  simulated PacBio CLR datasets at multiple sequencing coverages (10x–40x) are needed
  as identical inputs for all tools.
when_to_use_negative:
- The input data are real sequencing reads (e.g. the NA12878 real human genome sequencing
  dataset used in the paper for variant-calling evaluation) rather than controlled
  simulated reads
- The goal is short-read mapping benchmarking — the skill is specific to long noisy
  reads from PacBio or Oxford Nanopore sequencers
- Reads are already aligned and only downstream SV-calling comparison is required
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: invMap
  role: Benchmarked long-read mapper that consumes the simulated read datasets and
    the reference genome to produce SAM alignments during the runtime benchmark
  repo: https://github.com/zhang134/invMap.git
- name: minimap2
  role: Benchmarked mapper run with 64 threads on each simulated coverage level to
    establish the fastest runtime baseline
- name: lra
  role: Benchmarked mapper run with 64 threads on each simulated coverage level
- name: Winnowmap2
  role: Benchmarked mapper run with 64 threads on each simulated coverage level
- name: NGMLR
  role: Benchmarked mapper run with 64 threads on each simulated coverage level; expected
    to be slower than invMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:b9d368edc665b4660b64b3601942d6a1dbb18e205b8b17f6f5d6700dd51193aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# simulated-read-dataset-generation

## Summary

Generates (or obtains) simulated long-read sequencing datasets — e.g. PacBio CLR reads aligned to a reference genome at 10x, 20x, 30x, and 40x coverage — so that competing mappers (invMap, minimap2, Winnowmap2, lra, NGMLR) can be benchmarked for execution time under identical, controlled inputs.

## When to use

Use when benchmarking long-read mapper runtime or accuracy and controlled simulated PacBio CLR datasets at multiple sequencing coverages (10x–40x) are needed as identical inputs for all tools.

## When NOT to use

- The input data are real sequencing reads (e.g. the NA12878 real human genome sequencing dataset used in the paper for variant-calling evaluation) rather than controlled simulated reads
- The goal is short-read mapping benchmarking — the skill is specific to long noisy reads from PacBio or Oxford Nanopore sequencers
- Reads are already aligned and only downstream SV-calling comparison is required

## Inputs

- Reference genome FASTA file (e.g. genome.fa)
- Simulated PacBio CLR long reads in FASTA or FASTQ format (invMap accepts fasta or fastq input)
- Target sequencing coverage levels: 10x, 20x, 30x, 40x

## Outputs

- Simulated read datasets per coverage level (FASTA/FASTQ)
- Per-mapper SAM alignment outputs (e.g. invMap_aligned.sam produced as './invMap genome.fa reads.fa > invMap_aligned.sam')
- mapper_runtime_benchmark.csv tabulating wall-clock runtimes per coverage level and mapper

## How to apply

Obtain or generate simulated PacBio CLR read datasets aligned to the reference genome at four sequencing coverage levels: 10x, 20x, 30x, and 40x, mirroring the SV benchmarking setup of experiment 3.3 of the invMap paper. Keep all parameters except the mapper fixed: run each of minimap2, lra, Winnowmap2, invMap, and NGMLR on the same simulated datasets with 64 threads and record wall-clock runtime per coverage level. This matches the invMap benchmarking design, where simulated reads across genomes and coverages are used because longer PacBio or Oxford Nanopore reads more frequently span SV breakpoints. invMap needs substantial resources for large references — the README specifies a minimum of 16G RAM and ~800 M disk space — so provision the benchmarking machine accordingly and compile invMap with make before running. Tabulate runtimes per coverage and mapper into mapper_runtime_benchmark.csv, and validate against the Supplementary Fig. S7 ordering (minimap2 fastest, then lra and Winnowmap2, invMap faster than NGMLR).

## Related tools

- **invMap** (Benchmarked long-read mapper that consumes the simulated read datasets and the reference genome to produce SAM alignments during the runtime benchmark) — https://github.com/zhang134/invMap.git
- **minimap2** (Benchmarked mapper run with 64 threads on each simulated coverage level to establish the fastest runtime baseline)
- **lra** (Benchmarked mapper run with 64 threads on each simulated coverage level)
- **Winnowmap2** (Benchmarked mapper run with 64 threads on each simulated coverage level)
- **NGMLR** (Benchmarked mapper run with 64 threads on each simulated coverage level; expected to be slower than invMap)

## Evaluation signals

- The recorded runtimes reproduce the Supplementary Fig. S7 ordering: minimap2 fastest at all coverages, followed by lra and Winnowmap2, with invMap significantly faster than NGMLR
- All five mappers are run on identical simulated datasets at each of the four coverages (10x, 20x, 30x, 40x) with the same thread count (64 threads)
- Each mapper produces a valid alignment output per coverage level (invMap yields invMap_aligned.sam in SAM format)
- mapper_runtime_benchmark.csv contains one runtime entry per mapper per coverage level with no missing cells
- The benchmark machine meets invMap's stated requirements (minimum 16G RAM for mapping to large reference genomes, ~800 M disk space)

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which may constrain benchmarking hardware
- invMap supports Linux (recommended), Windows 7, and Windows 10 only; it must be compiled from source with make before use
- Findings and ordering are specific to the simulated multi-coverage setting and PacBio CLR read simulation; real-data behavior (e.g. NA12878) is assessed separately in the paper

## Evidence

- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs)"
- [abstract] experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods: [evidence span withheld — anchor status: not_found]
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] The final mapping result is invMap_aligned.sam in SAM format.: "Step 1: compile codes using make command Step 2: ./invmap then come out help information and options explanation"

---
name: multi-tool-comparison-analysis
description: Use when a new mapping or analysis tool must be validated against established
  alternatives on the same datasets, e.g. comparing runtimes or SV-calling accuracy
  across coverage levels.
when_to_use_negative:
- The goal is a single tool's alignment quality on one dataset rather than a controlled
  head-to-head comparison across tools
- Read data are short Illumina reads — the compared mappers are designed for long
  PacBio/Oxford Nanopore reads
- Compute resources (threads, RAM) cannot be held constant across tools, making runtime
  comparisons invalid
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: invMap
  role: Tool under evaluation; maps long noisy reads and detects inversion SVs; benchmarked
    for runtime and SV-calling accuracy
  repo: https://github.com/zhang134/invMap.git
- name: minimap2
  role: Baseline comparator mapper; consistently the fastest tool at all coverages
    in the runtime benchmark
- name: Winnowmap2
  role: Comparator long-read mapper; ranked after lra and minimap2 in runtime
- name: lra
  role: Comparator long-read mapper; second-fastest after minimap2 in the runtime
    benchmark
- name: NGMLR
  role: Comparator long-read mapper; slowest of the five tools, significantly slower
    than invMap
- name: PacBio
  role: Sequencing platform whose CLR simulated reads form the benchmark datasets
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:f1ee05a0d2673804122ac113c1840e713464a84198b9697cae736a4745a1c3fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - NGMLR
  - PacBio
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# multi-tool-comparison-analysis

## Summary

Benchmarks competing long-read mapping tools (invMap, minimap2, Winnowmap2, lra, NGMLR) side-by-side under identical conditions to quantify accuracy and runtime differences, as done in the invMap SV benchmarking study on simulated PacBio CLR datasets.

## When to use

Use when a new mapping or analysis tool must be validated against established alternatives on the same datasets, e.g. comparing runtimes or SV-calling accuracy across coverage levels.

## When NOT to use

- The goal is a single tool's alignment quality on one dataset rather than a controlled head-to-head comparison across tools
- Read data are short Illumina reads — the compared mappers are designed for long PacBio/Oxford Nanopore reads
- Compute resources (threads, RAM) cannot be held constant across tools, making runtime comparisons invalid

## Inputs

- Simulated PacBio CLR read datasets in FASTA/FASTQ aligned to a reference genome at 10x, 20x, 30x, and 40x coverage
- Installed builds of invMap, minimap2, Winnowmap2, lra, and NGMLR
- A reference genome FASTA file (e.g. genome.fa)

## Outputs

- mapper_runtime_benchmark.csv with wall-clock runtime per mapper per coverage level
- Ordering/ranking of tools per coverage level reproducible against Supplementary Fig. S7
- SAM alignment outputs per tool (e.g. invMap_aligned.sam)

## How to apply

Fix a common evaluation setup: simulated PacBio CLR read datasets aligned to a reference genome at multiple sequencing coverages (10x, 20x, 30x, 40x in the invMap study). Run every tool under identical resource settings — the source workflow specifies 64 threads for each mapper — and record wall-clock runtime per tool per coverage. Complement runtime with accuracy readouts such as SV/inversion calling performance on simulated genomes, since the paper's claim is that invMap is more accurate for inversions while minimap2 remains fastest. Tabulate all measurements into a structured CSV (mapper_runtime_benchmark.csv) with one row per coverage level and mapper. Validate the benchmark against the reference ordering (Supplementary Fig. S7): minimap2 fastest, then lra and Winnowmap2, invMap faster than NGMLR.

## Related tools

- **invMap** (Tool under evaluation; maps long noisy reads and detects inversion SVs; benchmarked for runtime and SV-calling accuracy) — https://github.com/zhang134/invMap.git
- **minimap2** (Baseline comparator mapper; consistently the fastest tool at all coverages in the runtime benchmark)
- **Winnowmap2** (Comparator long-read mapper; ranked after lra and minimap2 in runtime)
- **lra** (Comparator long-read mapper; second-fastest after minimap2 in the runtime benchmark)
- **NGMLR** (Comparator long-read mapper; slowest of the five tools, significantly slower than invMap)
- **PacBio** (Sequencing platform whose CLR simulated reads form the benchmark datasets)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Recorded runtimes reproduce the Supplementary Fig. S7 ordering: minimap2 fastest, followed by lra and Winnowmap2, with invMap faster than NGMLR
- Each mapper ran with the same thread count (64) and on identical read files per coverage level
- All five mappers produce valid SAM output for every coverage level (e.g. invMap_aligned.sam parses as SAM)
- mapper_runtime_benchmark.csv contains one runtime entry for each of 5 tools x 4 coverage levels (20 rows)
- Accuracy side-check: invMap locates aligned regions and calls inversion SVs more accurately than competitors on the simulated datasets

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Comparison findings are grounded in simulated PacBio CLR data across specific genomes and coverages, not exhaustive real-data conditions
- invMap version used (1.0.0, 2023/09/08) has no changelog, so reproducibility depends on pinning this version

## Evidence

- [results] Minimap2 is consistently the fastest tool at all coverages, followed by lra and Winnowmap2, while invMap is significantly faster than NGMLR.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap is more accurate to locate aligned regions and call SVs for inversions than competing methods: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "A running example
-----------

./invMap genome.fa reads.fa > invMap_aligned.sam"
- [abstract] Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"

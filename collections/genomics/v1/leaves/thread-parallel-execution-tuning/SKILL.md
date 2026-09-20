---
name: thread-parallel-execution-tuning
description: Use when comparing runtime performance of long-read aligners on multi-coverage
  simulated CLR datasets, or when fixing thread counts (64) so wall-clock times are
  comparable across mappers.
when_to_use_negative:
- When the goal is mapping accuracy or SV-calling sensitivity rather than execution-time
  comparison — use the accuracy benchmarking skill instead
- When input reads are short reads or real (non-simulated) datasets; this ranking
  was established on simulated PacBio CLR data at controlled coverages
- When fewer than 64 threads or less than 16G RAM are available for invMap, since
  timing results would not be comparable
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: invMap
  role: Mapper under test; inversion-sensitive long-read mapper run with 64 threads
    whose wall-clock time is recorded
  repo: github.com/zhang134/invMap
- name: minimap2
  role: Mapper under test; consistently the fastest tool at all coverages in the reference
    ranking
- name: lra
  role: Mapper under test; second-fastest tier in the Supplementary Fig. S7 ordering
- name: Winnowmap2
  role: Mapper under test; groups with lra in the middle of the runtime ranking
- name: NGMLR
  role: Mapper under test; slowest tool, significantly slower than invMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:3c1c14cbf7f240e5808d9df703bfff31646e7d03b1a860d0c5e0d68393e9dfd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NGMLR
  - Winnowmap2
  - invMap
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# thread-parallel-execution-tuning

## Summary

Benchmark the wall-clock execution time of long-read mappers (invMap, minimap2, Winnowmap2, lra, NGMLR) on simulated PacBio CLR datasets at fixed thread counts (64) across 10x–40x sequencing coverage, to quantify runtime trade-offs of SV-sensitive mapping.

## When to use

Use when comparing runtime performance of long-read aligners on multi-coverage simulated CLR datasets, or when fixing thread counts (64) so wall-clock times are comparable across mappers.

## When NOT to use

- When the goal is mapping accuracy or SV-calling sensitivity rather than execution-time comparison — use the accuracy benchmarking skill instead
- When input reads are short reads or real (non-simulated) datasets; this ranking was established on simulated PacBio CLR data at controlled coverages
- When fewer than 64 threads or less than 16G RAM are available for invMap, since timing results would not be comparable

## Inputs

- Simulated PacBio CLR read datasets in FASTA or FASTQ format aligned to a reference genome at 10x, 20x, 30x, and 40x coverage
- Reference genome FASTA file (e.g. genome.fa)

## Outputs

- mapper_runtime_benchmark.csv with wall-clock runtime per mapper per coverage level
- Per-mapper SAM alignment files (e.g. invMap_aligned.sam)

## How to apply

Generate or obtain simulated PacBio CLR read datasets aligned to a reference genome at 10x, 20x, 30x, and 40x coverage. Run each of the five mappers — minimap2, lra, Winnowmap2, invMap, NGMLR — on every dataset under identical conditions, holding the thread count fixed at 64 threads so wall-clock runtime is attributable to the tool rather than parallelism differences. Record the wall-clock runtime for each mapper/coverage combination and tabulate them into mapper_runtime_benchmark.csv. Validate by checking that the recorded runtimes reproduce the Supplementary Fig. S7 ordering: minimap2 fastest at all coverages, followed by lra and Winnowmap2, with invMap significantly faster than NGMLR. Note that invMap needs substantial resources — at least 16G RAM for mapping to large reference genomes — so provision hardware before timing runs.

## Related tools

- **invMap** (Mapper under test; inversion-sensitive long-read mapper run with 64 threads whose wall-clock time is recorded) — github.com/zhang134/invMap
- **minimap2** (Mapper under test; consistently the fastest tool at all coverages in the reference ranking)
- **lra** (Mapper under test; second-fastest tier in the Supplementary Fig. S7 ordering)
- **Winnowmap2** (Mapper under test; groups with lra in the middle of the runtime ranking)
- **NGMLR** (Mapper under test; slowest tool, significantly slower than invMap)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Recorded runtimes reproduce the Supplementary Fig. S7 ordering: minimap2 fastest, then lra and Winnowmap2, with invMap faster than NGMLR at every coverage level
- Every mapper/coverage combination was run with exactly 64 threads, so wall-clock differences reflect the tool rather than parallelism
- mapper_runtime_benchmark.csv contains one runtime row per mapper (5) per coverage level (10x, 20x, 30x, 40x)
- All mappers consumed the same simulated PacBio CLR FASTA/FASTQ input datasets at identical coverage values
- invMap outputs valid SAM alignments (invMap_aligned.sam) for each timed run, confirming the timing measured a completed mapping

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, which constrains the machines on which fair timing is possible
- Runtime ranking was established on simulated datasets; real-data runtimes may differ
- Ranking depends on fixed hardware and thread settings (64 threads); results are not transferable to different thread counts or machines without re-benchmarking

## Evidence

- [methods] Run minimap2 on each simulated CLR dataset with 64 threads and record the wall-clock runtime.: [evidence span withheld — anchor status: not_found]
- [supplementary] the recorded runtimes must reproduce the Supplementary Fig. S7 ordering: [evidence span withheld — anchor status: not_found]
- [results] Minimap2 is consistently the fastest tool at all coverages: [evidence span withheld — anchor status: not_found]
- [readme] invMap requires a minimum of 16G of RAM: "invMap requires a minimum of
16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Input file requires fasta or fastq format: "Input file requires fasta or fastq format"

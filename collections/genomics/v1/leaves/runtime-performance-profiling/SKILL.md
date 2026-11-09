---
name: runtime-performance-profiling
description: Use when comparing wall-clock runtime of long-read aligners on simulated
  PacBio CLR datasets at multiple sequencing coverages (10x–40x), or when reproducing
  the Supplementary Fig. S7 mapper-speed ordering for SV benchmarking.
when_to_use_negative:
- The goal is SV-calling or alignment accuracy comparison (e.g. inversion detection)
  rather than execution-time ranking
- Input reads are short Illumina reads — all five mappers in this benchmark target
  long PacBio/Oxford Nanopore noisy reads
- Only a single coverage level or a single mapper is being timed, since the skill's
  validation depends on the cross-coverage, cross-mapper ordering
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: invMap
  role: Mapped long noisy reads and timed as one of the five compared mappers; produces
    SAM output via ./invMap genome.fa reads.fa
  repo: https://github.com/zhang134/invMap.git
- name: minimap2
  role: Fastest mapper in the runtime benchmark; run on each simulated CLR dataset
    with 64 threads
- name: lra
  role: Mapper timed in the benchmark; second-fastest tier alongside minimap2 and
    Winnowmap2
- name: Winnowmap2
  role: Mapper timed in the benchmark; in the middle speed tier of the Supplementary
    Fig. S7 ordering
- name: NGMLR
  role: Slowest mapper in the benchmark; run on each dataset with 64 threads and recorded
    for runtime comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:65ed631bd619fce2b6c9300d7165a9429740b803ea13410fd52ee2977bb2bf20
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# runtime-performance-profiling

## Summary

Benchmarking protocol for comparing wall-clock execution times of five long-read mappers (invMap, minimap2, Winnowmap2, lra, NGMLR) on simulated PacBio CLR datasets across 10x, 20x, 30x, and 40x sequencing coverage, as used in the invMap SV benchmarking experiment (experiment 3.3).

## When to use

Use when comparing wall-clock runtime of long-read aligners on simulated PacBio CLR datasets at multiple sequencing coverages (10x–40x), or when reproducing the Supplementary Fig. S7 mapper-speed ordering for SV benchmarking.

## When NOT to use

- The goal is SV-calling or alignment accuracy comparison (e.g. inversion detection) rather than execution-time ranking
- Input reads are short Illumina reads — all five mappers in this benchmark target long PacBio/Oxford Nanopore noisy reads
- Only a single coverage level or a single mapper is being timed, since the skill's validation depends on the cross-coverage, cross-mapper ordering

## Inputs

- Simulated PacBio CLR read datasets (FASTA/FASTQ) aligned to a reference genome at 10x, 20x, 30x, and 40x coverage
- Reference genome file (e.g. genome.fa in FASTA format)

## Outputs

- mapper_runtime_benchmark.csv with wall-clock runtimes per coverage level per mapper
- Per-mapper alignment outputs (e.g. invMap_aligned.sam in SAM format) from each timed run

## How to apply

Obtain or generate simulated PacBio CLR read datasets aligned to the reference genome at 10x, 20x, 30x, and 40x sequencing coverage. Run each of the five mappers — minimap2, lra, Winnowmap2, invMap, NGMLR — on every dataset using 64 threads, recording the wall-clock runtime for each run. invMap must be compiled first with `make` and requires a minimum of 16G of RAM for mapping to large reference genomes. Tabulate all runtimes per coverage level and mapper into mapper_runtime_benchmark.csv. Validate that the recorded ordering reproduces the Supplementary Fig. S7 result: minimap2 fastest at all coverages, followed by lra and Winnowmap2, with invMap significantly faster than NGMLR.

## Related tools

- **invMap** (Mapped long noisy reads and timed as one of the five compared mappers; produces SAM output via ./invMap genome.fa reads.fa) — https://github.com/zhang134/invMap.git
- **minimap2** (Fastest mapper in the runtime benchmark; run on each simulated CLR dataset with 64 threads)
- **lra** (Mapper timed in the benchmark; second-fastest tier alongside minimap2 and Winnowmap2)
- **Winnowmap2** (Mapper timed in the benchmark; in the middle speed tier of the Supplementary Fig. S7 ordering)
- **NGMLR** (Slowest mapper in the benchmark; run on each dataset with 64 threads and recorded for runtime comparison)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- Recorded runtime ordering reproduces Supplementary Fig. S7: minimap2 fastest, followed by lra and Winnowmap2, with invMap faster than NGMLR
- Each of the five mappers was run with exactly 64 threads on every coverage level (10x, 20x, 30x, 40x)
- mapper_runtime_benchmark.csv contains one runtime row per mapper per coverage level with no missing entries
- invMap run completed without memory errors, satisfying its 16G minimum RAM requirement for large reference genomes
- invMap run produced a valid SAM-format output (invMap_aligned.sam) that was generated during the timed execution

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes, so runtimes are only comparable on sufficiently provisioned hardware
- Findings are grounded in simulated PacBio CLR datasets; runtime ordering may differ on real datasets (e.g. the NA12878 human sequencing dataset) or on other read types
- Runtime comparisons are only valid under identical thread counts (64 threads here) and identical coverage levels across mappers

## Evidence

- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [readme] Step 1: compile codes using make command: "Step 1: compile codes using make command"
- [readme] ./invMap genome.fa reads.fa > invMap_aligned.sam: "./invMap genome.fa reads.fa > invMap_aligned.sam"
- [supplementary] Minimap2 is consistently the fastest tool at all coverages, followed by lra and Winnowmap2, while invMap is significantly faster than NGMLR.: [evidence span withheld — anchor status: not_found]
- [abstract] experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods: "experimental results demonstrate that invMap is more accurate to locate aligned regions and call SVs for inversions than the competing methods"

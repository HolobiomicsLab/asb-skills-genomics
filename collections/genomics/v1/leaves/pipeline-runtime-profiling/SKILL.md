---
name: pipeline-runtime-profiling
description: Use when benchmarking a long-read TE/SV pipeline's compute cost per processing
  stage, or when reporting a fair head-to-head runtime comparison between tools executed
  on the same samples with controlled thread counts.
when_to_use_negative:
- When the benchmark tools run under incommensurate thread allocations (e.g. GraffiTE
  consuming 96 threads when assigned 32) — timing is not comparable and should be
  excluded
- When the goal is accuracy (precision/recall/F1 against the 5,630-insertion HGSVC
  truth set) rather than compute cost — use the SURVIVOR-based benchmarking skill
  instead
- For short-read TE tools such as MELT or xTea, which the paper did not evaluate for
  runtime
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RetroInspector
  role: Snakemake pipeline whose per-stage runtimes (alignment, sorting, variant calling,
    genotyping) are profiled; --stats flag records timing
  repo: https://github.com/javiercguard/retroinspector
- name: Snakemake
  role: Workflow engine; '-c <threads>' sets parallelism and '--stats' captures stage
    timing
- name: minimap2
  role: Long-read alignment stage against GRCh38 — the most time-consuming stage (231.88
    ± 32 min/sample)
  repo: https://github.com/lh3/minimap2
- name: cuteSV
  role: Variant calling stage timed at 28.01 ± 0.90 min per sample
  repo: https://github.com/tjiangHIT/cuteSV
- name: Sniffles2
  role: Variant calling stage timed at 7.29 ± 0.33 min per sample
  repo: https://github.com/fritzsedlazeck/Sniffles
- name: mosdepth
  role: Genome-wide coverage calculation, accounting for 98.3% of genotyping time
    (37.83 ± 3.59 min/sample)
  repo: https://github.com/brentp/mosdepth
- name: PALMER
  role: Benchmark tool run one execution per TE type per sample (4 per sample) under
    GNU parallel on 20 threads for runtime ordering comparison
- name: GNU parallel
  role: Orchestrates PALMER's per-TE-type executions across 20 threads
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-025-98847-7
    title: Advanced analysis of retrotransposon variation in the human genome with
      nanopore sequencing using RetroInspector
  source_content_hash: sha256:eee154ec2e15637e7a36852bc0ce3327974bf0502ab627b0f88fb86ecc4bfe68
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/javiercguard/retroinspector
  repo_url: https://github.com/javiercguard/retroinspector
  tools:
  - FamDB toolkit
  - GNU parallel
  - GraffiTE
  - PALMER
  - PanGenIE
  - RetroInspector
  - Snakemake
  - Sniffles2
  - cuteSV
  - minimap2
  - mosdepth
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 24
---

# pipeline-runtime-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiling per-stage wall-clock runtime of the RetroInspector Snakemake pipeline (alignment, sorting/indexing, cuteSV/Sniffles2 variant calling, reassembly, genotyping) on nanopore samples, and comparing runtimes fairly against benchmark tools like PALMER and GraffiTE.

## When to use

Use when benchmarking a long-read TE/SV pipeline's compute cost per processing stage, or when reporting a fair head-to-head runtime comparison between tools executed on the same samples with controlled thread counts.

## When NOT to use

- When the benchmark tools run under incommensurate thread allocations (e.g. GraffiTE consuming 96 threads when assigned 32) — timing is not comparable and should be excluded
- When the goal is accuracy (precision/recall/F1 against the 5,630-insertion HGSVC truth set) rather than compute cost — use the SURVIVOR-based benchmarking skill instead
- For short-read TE tools such as MELT or xTea, which the paper did not evaluate for runtime

## Inputs

- Nanopore FASTQ files (one per sample, e.g. HG00733, HG00514, NA19240 from HGSV)
- Reference genome FASTA (GRCh38/hg38)
- Snakemake pipeline configuration (config.template.yaml / CLI --config)

## Outputs

- Per-stage runtime table (mean ± minutes per sample) covering alignment, sorting/indexing, Sniffles2, cuteSV, reassembly, and genotyping
- Snakemake --stats timing file
- Cross-tool runtime comparison table with per-sample runtime ordering (PALMER vs RetroInspector; GraffiTE excluded from timing)

## How to apply

Run the Snakemake pipeline with an explicit thread count (the paper used 31 threads, recommended via '-c <number of threads>') and record time with the '--stats' flag, then decompose runtime into its named stages. In the paper's HGSV three-sample run, each stage was timed per sample: alignment with minimap2 against GRCh38 dominated (231.88 ± 32 min/sample), sorting/indexing the BAM took 12.78 ± 1.31 min, Sniffles2 took 7.29 ± 0.33 min and cuteSV 28.01 ± 0.90 min, and genotyping consumed 37.83 ± 3.59 min — of which 98.3% was spent calculating genome-wide coverage (e.g. with mosdepth). When comparing against alternatives (PALMER, run one execution per TE type per sample under GNU parallel on 20 threads; GraffiTE), normalize or exclude runs whose actual thread usage differs from the assignment — the paper excluded GraffiTE from timing because it used 96 threads when assigned 32. Report mean ± spread per stage and assemble a per-sample runtime ordering table.

## Related tools

- **RetroInspector** (Snakemake pipeline whose per-stage runtimes (alignment, sorting, variant calling, genotyping) are profiled; --stats flag records timing) — https://github.com/javiercguard/retroinspector
- **Snakemake** (Workflow engine; '-c <threads>' sets parallelism and '--stats' captures stage timing)
- **minimap2** (Long-read alignment stage against GRCh38 — the most time-consuming stage (231.88 ± 32 min/sample)) — https://github.com/lh3/minimap2
- **cuteSV** (Variant calling stage timed at 28.01 ± 0.90 min per sample) — https://github.com/tjiangHIT/cuteSV
- **Sniffles2** (Variant calling stage timed at 7.29 ± 0.33 min per sample) — https://github.com/fritzsedlazeck/Sniffles
- **mosdepth** (Genome-wide coverage calculation, accounting for 98.3% of genotyping time (37.83 ± 3.59 min/sample)) — https://github.com/brentp/mosdepth
- **PALMER** (Benchmark tool run one execution per TE type per sample (4 per sample) under GNU parallel on 20 threads for runtime ordering comparison)
- **GNU parallel** (Orchestrates PALMER's per-TE-type executions across 20 threads)

## Examples

```
snakemake --use-conda -c 31 --stats stats.txt --config referenceGenome=/home/yourname/hg38.fa outputPath=/home/yourname/experiment_results fastq_directory=/home/yourname/data/samples
```

## Evaluation signals

- Per-stage durations fall in the reported ranges: alignment ~232 min/sample, sorting/indexing ~12.8 min, Sniffles2 ~7.3 min, cuteSV ~28 min, genotyping ~37.8 min (31 threads)
- Genotyping stage time is dominated by coverage calculation (expected ~98.3%) — a sanity check that genome-wide coverage, not genotyping logic, is the bottleneck
- All timed tools run on the same samples with recorded thread counts; any tool whose actual thread usage deviates from assignment (as GraffiTE's 96 vs 32) is excluded from the timing table
- Per-sample runtime ordering relative to PALMER (4 executions per sample) is reproduced in the assembled comparison table
- Snakemake --stats output exists for each sample run and aggregates per-rule times consistent with the stage breakdown

## Limitations

- GraffiTE had to be excluded from runtime comparison because it used 96 threads when assigned 32, making cross-tool timing non-comparable
- PALMER requires one execution per TE type (4 per sample), which inflates its total runtime relative to single-run pipelines
- HGSVC truth-set data was basecalled with an older basecaller, so timing on these samples may not reflect current basecaller throughput
- Pipeline repository settings are in a template configuration file; version numbers for the timed tools are not stated in the results section

## Evidence

- [methods] Timing captured with --stats flag: [evidence span withheld — anchor status: not_found]

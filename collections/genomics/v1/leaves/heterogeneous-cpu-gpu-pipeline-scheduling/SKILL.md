---
name: heterogeneous-cpu-gpu-pipeline-scheduling
description: Use when implementing or tuning a heterogeneous CPU+GPU methylation-calling
  pipeline (e.g. Galaxy-methyl call-methylation) and you need batch scheduling that
  overlaps the four call-methylation steps across concurrent batches.
when_to_use_negative:
- Input reads are already aligned and you only need a single-threaded methylation
  call without pipelining overhead
- The workflow is f5c eventalign or Nanopolish consensus/variant calling, which are
  outside the pipelined call-methylation scheduling this skill covers
- No NVIDIA GPU is available and only the CPU-parallel path of Galaxy-methyl/f5c is
  required, since the pipeline is designed to overlap GPU methylation score calculation
  with CPU steps
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Galaxy-methyl
  role: Pipelined, GPU-optimized call-methylation module whose four steps are overlapped
    by the three-batch scheduler
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: f5c
  role: Optimized re-implementation of Nanopolish call-methylation/eventalign that
    Galaxy-methyl modified to add GPU parallelization and pipelining
  repo: https://github.com/hasindu2008/f5c
- name: Nanopolish
  role: Base signal-level package containing the original call-methylation module
    that Galaxy-methyl optimizes
  repo: https://github.com/jts/nanopolish
- name: minimap2
  role: Aligns basecalled ONT reads to the reference genome ('-a -x map-ont') as the
    upstream alignment feeding the pipeline
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the BAM produced by the minimap2 alignment step
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:c9919da734b3e4b60a935a60f41ff7bfcf123fc2f7c78089a60d75a7300b3a5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fengyilin118/Galaxy-methyl
  tools:
  - Galaxy-methyl
  - Nanopolish
  - f5c
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# heterogeneous-cpu-gpu-pipeline-scheduling

## Summary

Pipelines the four steps of Galaxy-methyl's call-methylation module (event detection, alignment, final calibration, methylation score calculation on GPU) while keeping three batches in flight, avoiding CPU/GPU over- or under-utilization as described in Feng et al. 2022.

## When to use

Use when implementing or tuning a heterogeneous CPU+GPU methylation-calling pipeline (e.g. Galaxy-methyl call-methylation) and you need batch scheduling that overlaps the four call-methylation steps across concurrent batches.

## When NOT to use

- Input reads are already aligned and you only need a single-threaded methylation call without pipelining overhead
- The workflow is f5c eventalign or Nanopolish consensus/variant calling, which are outside the pipelined call-methylation scheduling this skill covers
- No NVIDIA GPU is available and only the CPU-parallel path of Galaxy-methyl/f5c is required, since the pipeline is designed to overlap GPU methylation score calculation with CPU steps

## Inputs

- Raw FAST5 signal data directory
- Basecalled FASTQ file (e.g. FAB41174.fastq.gz)
- Reference genome FASTA (e.g. hg38.fasta) for read alignment
- Batch size and thread count parameters (e.g. -B 4.0M, -t 4)

## Outputs

- Sorted, indexed BAM file of aligned basecalled reads
- Methylation calls file (e.g. methylation_calls.tsv) with detected methylated bases
- Execution timeline log of per-batch start/end timestamps for each of the four steps

## How to apply

After indexing FAST5 signals against the basecalled FASTQ (Galaxy-methyl index) and producing a sorted, indexed BAM via 'minimap2 -a -x map-ont' piped into 'samtools sort', implement the pipelined scheduler inside 'Galaxy-methyl call-methylation'. Launch CPU threads (e.g. -t 4) with a GPU-side batch size (e.g. -B 4.0M) and keep three batches in flight across the four steps: batch n in final calibration while batch n+1 runs alignment and batch n+2 runs event detection. Enforce per-step ordering by having the thread executing a step for batch k wait on batch k-1 completing that same step, so GPU-heavy methylation score calculation overlaps CPU-bound event detection and alignment without resource contention. Log per-batch start/end timestamps per step to an execution timeline to verify the overlapped three-batch schedule. The final step writes detected methylated bases to an output file (e.g. methylation_calls.tsv).

## Related tools

- **Galaxy-methyl** (Pipelined, GPU-optimized call-methylation module whose four steps are overlapped by the three-batch scheduler) — https://github.com/fengyilin118/Galaxy-methyl
- **f5c** (Optimized re-implementation of Nanopolish call-methylation/eventalign that Galaxy-methyl modified to add GPU parallelization and pipelining) — https://github.com/hasindu2008/f5c
- **Nanopolish** (Base signal-level package containing the original call-methylation module that Galaxy-methyl optimizes) — https://github.com/jts/nanopolish
- **minimap2** (Aligns basecalled ONT reads to the reference genome ('-a -x map-ont') as the upstream alignment feeding the pipeline) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the BAM produced by the minimap2 alignment step) — https://www.htslib.org/

## Examples

```
./Galaxy-methyl call-methylation -r reads.fastq -b sorted_indexed.bam -g ref.fasta -t 4 -B 4.0M > methylation_calls.tsv
```

## Evaluation signals

- The execution timeline log shows three batches concurrently in flight with per-step ordering preserved (batch k's step never starts before batch k-1 completes the same step)
- methylation_calls.tsv (or equivalent output) is produced by the pipelined run
- Total run time is reduced versus a non-pipelined f5c baseline (article reports on average 35% total execution time reduction compared with f5c)
- Per-step GPU utilization shows the methylation score calculation on GPU overlapping CPU-bound event detection/alignment without overutilization
- Methylation calls agree with non-pipelined call-methylation results on the same inputs

## Limitations

- Requires an NVIDIA GPU with CUDA library for the accelerated methylation score calculation step
- The pipeline is specific to the call-methylation module's four steps; other Nanopolish/f5c modules (e.g. eventalign, consensus) are not pipelined by this scheduler
- Per-step ordering constraint (batch k waits on batch k-1 for the same step) serializes each step, so throughput depends on balanced step durations; heavily imbalanced step times limit overlap benefit
- Nanopolish does not support R10.4 flowcells for methylation calling, constraining applicable chemistries

## Evidence

- [abstract] Pipeline keeps three batches in flight across four steps: "We propose Galaxy-methyl, which parallelizes and optimizes the methylation score calculation step on GPU"
- [intro] Pipelining of the four call-methylation steps: "Galaxy-methyl modified *f5c* to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module."
- [methods] Index links read ids with signal-level data: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] Alignment pipeline with minimap2 and samtools: "align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome"
- [abstract] Speedup figures versus prior tools: "reduce the total execution time by 35% compared with f5c, on average"

---
name: gpu-kernel-profiling-nvprof
description: Use when you need to profile a GPU-accelerated kernel (e.g. the methylation
  score calculation of Galaxy-methyl call-methylation) and extract per-run sm_efficiency,
  achieved_occupancy and memory metrics under NVProf.
when_to_use_negative:
- The workload runs on CPU only (e.g. stock Nanopolish or f5c without GPU acceleration)
  — no sm_efficiency or achieved_occupancy counters will be emitted
- You need wall-clock speedup or end-to-end execution time comparisons rather than
  per-kernel GPU utilization metrics — use timing benchmarks instead
- The input FAST5/BLOW5 signal data has not been indexed against the basecalled reads
  and the BAM is not sorted/indexed, so call-methylation cannot run under the profiler
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Galaxy-methyl
  role: GPU-parallelized call-methylation module whose methylation score calculation
    kernel (HMM-based, -t 4 threads, -B 4.0M batch size, three batches) is profiled
    under NVProf
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: NVProf
  role: NVIDIA profiler used to capture memory consumption, sm_efficiency and achieved_occupancy
    counters from the GPU kernel
- name: minimap2
  role: Aligns basecalled ONT reads to the hg38 reference (-a -x map-ont) to produce
    the SAM/BAM consumed by the profiled call-methylation run
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 alignment output to prepare the sorted BAM
    input for call-methylation
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:cb0d6325ae785efb5ec97523d67245b559a47f5d5a0b7ec0b4c21f79873b9c15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fengyilin118/Galaxy-methyl
  tools:
  - Galaxy-methyl
  - NVProf
  - minimap2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# gpu-kernel-profiling-nvprof

## Summary

Reproduce and validate GPU kernel-level profiling metrics (memory consumption, sm_efficiency, achieved_occupancy) of the HMM-based methylation score calculation step in Galaxy-methyl using NVProf across two GPU server configurations.

## When to use

Use when you need to profile a GPU-accelerated kernel (e.g. the methylation score calculation of Galaxy-methyl call-methylation) and extract per-run sm_efficiency, achieved_occupancy and memory metrics under NVProf.

## When NOT to use

- The workload runs on CPU only (e.g. stock Nanopolish or f5c without GPU acceleration) — no sm_efficiency or achieved_occupancy counters will be emitted
- You need wall-clock speedup or end-to-end execution time comparisons rather than per-kernel GPU utilization metrics — use timing benchmarks instead
- The input FAST5/BLOW5 signal data has not been indexed against the basecalled reads and the BAM is not sorted/indexed, so call-methylation cannot run under the profiler

## Inputs

- Indexed D2 (FAB41174) FAST5 signal data linked to basecalled FASTQ (via Galaxy-methyl index)
- BAM file of ONT reads aligned to hg38 with minimap2 -a -x map-ont, sorted and indexed with samtools
- GPU servers: Serversmall and Serverlarge configurations

## Outputs

- NVProf profiling reports with per-run GPU memory consumption, sm_efficiency and achieved_occupancy counters
- gpu_profile_metrics.csv of extracted GPU metrics per run (three-batch runs on both servers)

## How to apply

Prepare the standard Galaxy-methyl pipeline inputs: index the D2 (FAB41174) FAST5 signal data against the basecalled FASTQ with Galaxy-methyl index, then align the basecalled ONT reads to the hg38 reference with minimap2 (-a -x map-ont) and sort/index the output with samtools sort and samtools index. Launch Galaxy-methyl call-methylation under NVProf on Serversmall with -t 4 CPU threads and -B 4.0M batch size so the run spans three batches, capturing memory consumption, sm_efficiency and achieved_occupancy counters. Repeat the identically parameterized profiled run on Serverlarge to compare GPU utilization across hardware scales. Parse the NVProf reports into gpu_profile_metrics.csv and validate the extracted values against the reported reference metrics (78% sm_efficiency / 0.6 achieved_occupancy on Serversmall; 90% / 0.8 on Serverlarge) to confirm the profiling reproduction is consistent.

## Related tools

- **Galaxy-methyl** (GPU-parallelized call-methylation module whose methylation score calculation kernel (HMM-based, -t 4 threads, -B 4.0M batch size, three batches) is profiled under NVProf) — https://github.com/fengyilin118/Galaxy-methyl
- **NVProf** (NVIDIA profiler used to capture memory consumption, sm_efficiency and achieved_occupancy counters from the GPU kernel)
- **minimap2** (Aligns basecalled ONT reads to the hg38 reference (-a -x map-ont) to produce the SAM/BAM consumed by the profiled call-methylation run) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 alignment output to prepare the sorted BAM input for call-methylation) — https://www.htslib.org/

## Examples

```
nvprof --metrics sm_efficiency,achieved_occupancy -o run.nvprof galaxy-methyl call-methylation -t 4 -B 4.0M reads.bam
```

## Evaluation signals

- Extracted sm_efficiency matches Table 3 reference values: 78% on Serversmall and 90% on Serverlarge
- Extracted achieved_occupancy matches reference values: 0.6 on Serversmall and 0.8 on Serverlarge
- The profiled run spans exactly three batches with -t 4 threads and -B 4.0M batch size, ensuring the workload is comparable across servers
- NVProf reports on Serversmall and Serverlarge were generated with identical parameters so metric deltas reflect hardware, not configuration differences
- gpu_profile_metrics.csv contains per-run entries for memory consumption, sm_efficiency and achieved_occupancy from each NVProf report

## Limitations

- Galaxy-methyl's reported speedup (3x-5x vs Nanopolish; ~35% total execution time reduction vs f5c) is a separate timing result — the profiling metrics here capture GPU utilization, not speedup
- Metrics are measured over a three-batch run with specific -t 4 / -B 4.0M parameters; different batch sizes or thread counts will change occupancy and efficiency
- Nanopolish-based signal-level methylation calling does not support R10.4 flowcells, limiting the applicability of the profiled kernel to earlier chemistries

## Evidence

- [abstract] parallelization and optimization of methylation score calculation step on GPU: "We propose Galaxy-methyl, which parallelizes and optimizes the methylation score calculation step on GPU"
- [methods] call-methylation thread parameter: "Finally, detect methylated bases. -t INT denotes how many threads are launched on CPU cores"
- [methods] read indexing step: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] alignment tooling: "align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome"
- [readme] module identity: "Galaxy-methyl* is an parallelized and optimized call-methylation module in [Nanopolish]"
- [abstract] performance context: "Galaxy-methyl can achieve 3×–5× speedup compared with Nanopolish"

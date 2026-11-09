---
name: gpu-occupancy-efficiency-metric-extraction
description: 'Use when validating GPU acceleration of the Nanopolish/f5c call-methylation
  module: you need sm_efficiency and achieved_occupancy counters from a three-batch
  NVProf run of Galaxy-methyl on Serversmall or Serverlarge.'
when_to_use_negative:
- The goal is methylation accuracy or runtime benchmarking rather than GPU hardware
  utilization metrics (sm_efficiency, achieved_occupancy)
- The methylation caller is the CPU-only Nanopolish or CPU-only f5c path, which does
  not exercise the GPU methylation score calculation step
- Input uses R10.4 flowcells, which Nanopolish explicitly does not support for methylation
  calling
tools:
- name: Galaxy-methyl
  role: Profiled call-methylation module whose GPU methylation score calculation produces
    the sm_efficiency/achieved_occupancy counters; also provides 'index' for linking
    read ids to FAST5 signal data
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: NVProf
  role: NVIDIA profiler used to capture memory consumption, sm_efficiency and achieved_occupancy
    during the three-batch call-methylation runs
- name: minimap2
  role: Aligns basecalled ONT reads to the hg38 reference (-a -x map-ont) to produce
    the BAM required by call-methylation
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 alignment output so the profiled run receives
    a coordinate-sorted BAM
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:fd038a2ecd401e595e8cdade09bc2e5f0a4b9315c80eacc429c86567ce508c9e
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# gpu-occupancy-efficiency-metric-extraction

## Summary

Reproduce and extract GPU profiling metrics (memory consumption, sm_efficiency, achieved_occupancy) from Galaxy-methyl's GPU-accelerated methylation score calculation step under NVProf, to validate that observed parallelism matches reported values (78%/0.6 on Serversmall, 90%/0.8 on Serverlarge).

## When to use

Use when validating GPU acceleration of the Nanopolish/f5c call-methylation module: you need sm_efficiency and achieved_occupancy counters from a three-batch NVProf run of Galaxy-methyl on Serversmall or Serverlarge.

## When NOT to use

- The goal is methylation accuracy or runtime benchmarking rather than GPU hardware utilization metrics (sm_efficiency, achieved_occupancy)
- The methylation caller is the CPU-only Nanopolish or CPU-only f5c path, which does not exercise the GPU methylation score calculation step
- Input uses R10.4 flowcells, which Nanopolish explicitly does not support for methylation calling

## Inputs

- FAST5 raw signal data for dataset D2 (FAB41174) plus its basecalled FASTQ
- hg38 FASTA Human Reference Genome
- minimap2 alignment in sorted, indexed BAM form (samtools sort/index)
- GPU-enabled Galaxy-methyl build compiled with cuda=1

## Outputs

- gpu_profile_metrics.csv with per-run GPU memory consumption, sm_efficiency (%) and achieved_occupancy
- NVProf profiling reports from the three-batch Serversmall and Serverlarge runs

## How to apply

Index the D2 (FAB41174) FAST5 signal data against the basecalled FASTQ with Galaxy-methyl index, then align the basecalled ONT reads to the hg38 reference with minimap2 (-a -x map-ont), piping the output into samtools sort and samtools index to obtain a sorted, indexed BAM. Launch Galaxy-methyl call-methylation under NVProf with -t 4 CPU threads and -B 4.0M batch size so the run spans exactly three batches; capture the memory consumption, sm_efficiency and achieved_occupancy counters per batch. Repeat the identical profiled run on Serverlarge, then parse the NVProf reports into gpu_profile_metrics.csv. The three-batch setup matters because per-batch counters must be averaged, and the batch size (4.0M) is what keeps both servers comparable.

## Related tools

- **Galaxy-methyl** (Profiled call-methylation module whose GPU methylation score calculation produces the sm_efficiency/achieved_occupancy counters; also provides 'index' for linking read ids to FAST5 signal data) — https://github.com/fengyilin118/Galaxy-methyl
- **NVProf** (NVIDIA profiler used to capture memory consumption, sm_efficiency and achieved_occupancy during the three-batch call-methylation runs)
- **minimap2** (Aligns basecalled ONT reads to the hg38 reference (-a -x map-ont) to produce the BAM required by call-methylation) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 alignment output so the profiled run receives a coordinate-sorted BAM) — https://www.htslib.org/

## Examples

```
nvprof ./galaxy-methyl call-methylation -t 4 -B 4.0M -r fast5_dir -g hg38.fa -a sorted.bam > methylation.tsv
```

## Evaluation signals

- Extracted sm_efficiency matches the Table 3 reference of 78% on Serversmall and 90% on Serverlarge
- Extracted achieved_occupancy matches 0.6 on Serversmall and 0.8 on Serverlarge
- The profiled call-methylation run with -B 4.0M spans exactly three batches in the NVProf report
- gpu_profile_metrics.csv contains memory consumption, sm_efficiency and achieved_occupancy for both Serversmall and Serverlarge runs
- Profiled pipeline consumed the same sorted/indexed minimap2 BAM on both servers (identical -t 4 and -B 4.0M parameters)

## Limitations

- Metrics are averages over a three-batch run of a single dataset (D2/FAB41174); other batch sizes or datasets may shift sm_efficiency and achieved_occupancy
- Requires a CUDA-enabled Galaxy-methyl build (make cuda=1 CUDA_LIB=/path/to/cuda/library/) and an NVIDIA profiler; CPU-only builds yield no GPU counters
- Nanopolish does not support R10.4 flowcells, limiting reuse on newer chemistries needing signal-level methylation analysis

## Evidence

- [results] Reported GPU metric targets: [evidence span withheld — anchor status: not_found]
- [methods] Profiling workflow with batch parameters: [evidence span withheld — anchor status: not_found]
- [readme] Galaxy-methyl scope (README): "Galaxy-methyl modified f5c to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module"
- [methods] Alignment preparation step: "minimap2 is used to map reads to the human genome"

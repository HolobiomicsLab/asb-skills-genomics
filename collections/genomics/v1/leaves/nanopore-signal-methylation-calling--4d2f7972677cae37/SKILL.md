---
name: nanopore-signal-methylation-calling
description: Use when you need to detect methylated CpG sites from basecalled Nanopore
  reads (FASTQ/BAM) plus raw FAST5 signal data on a CUDA GPU, and prioritize end-to-end
  execution time versus CPU-only tools.
when_to_use_negative:
- 'R10.4 flowcells: Nanopolish does not support R10.4 variant/methylation calling,
  so signal-level comparisons against it on R10 data are not meaningful'
- No NVIDIA GPU is available and only CPU execution is possible; f5c or Nanopolish
  CPU paths are the appropriate baselines
- The goal is event alignment (signal-to-k-mer alignment) rather than methylation
  calling; use the eventalign module of f5c/Nanopolish instead
edam_operation: http://edamontology.org/operation_3186
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Galaxy-methyl
  role: GPU-accelerated, pipelined call-methylation tool that indexes FAST5 signals
    and detects methylated cytosines via HMM
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: Nanopolish
  role: Reference tool for correctness validation of per-CpG log-likelihood ratios
    and CPU performance baseline
  repo: https://github.com/jts/nanopolish
- name: f5c
  role: Optimized CPU (OpenMP) baseline for the methylation score calculation comparison
  repo: https://github.com/hasindu2008/f5c
- name: minimap2
  role: Aligns basecalled ONT reads to the reference genome with the -a -x map-ont
    preset
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 alignment BAM prior to methylation calling
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_007
  - task_002
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:6f1c53212187c8a73ad34ae5a143bd30bb71ea40e5893f86596fbc99b7fe5164
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
  edam_operation: http://edamontology.org/operation_3186
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47619047619047616
n_claims: 21
---

# nanopore-signal-methylation-calling

## Summary

GPU-accelerated, pipelined detection of methylated cytosines from basecalled Oxford Nanopore reads and raw FAST5 signals using Galaxy-methyl, a parallelized re-implementation of Nanopolish's call-methylation module that achieves 3x-5x speedup over Nanopolish and ~35% reduction versus f5c.

## When to use

Use when you need to detect methylated CpG sites from basecalled Nanopore reads (FASTQ/BAM) plus raw FAST5 signal data on a CUDA GPU, and prioritize end-to-end execution time versus CPU-only tools.

## When NOT to use

- R10.4 flowcells: Nanopolish does not support R10.4 variant/methylation calling, so signal-level comparisons against it on R10 data are not meaningful
- No NVIDIA GPU is available and only CPU execution is possible; f5c or Nanopolish CPU paths are the appropriate baselines
- The goal is event alignment (signal-to-k-mer alignment) rather than methylation calling; use the eventalign module of f5c/Nanopolish instead

## Inputs

- Basecalled FASTQ reads (e.g. FAB41174.fastq.gz) from Oxford Nanopore sequencing
- Raw FAST5 signal files (e.g. FAB41174-3976885577_Multi/fast5)
- Reference genome FASTA (hg38, GRCh38.primary_assembly.genome.fa, or E. coli K-12 MG1655 GCA_000005845.2)
- Sorted, indexed BAM file from minimap2 -a -x map-ont + samtools sort/index

## Outputs

- methylation_calls.tsv with per-CpG-site methylated/unmethylated log-likelihood ratios
- Signal index file linking read ids to FAST5 signal-level data
- Optional cpg_llr_comparison.tsv from validation against Nanopolish

## How to apply

First create an index linking read ids to signal-level data in the FAST5 files with './Galaxy-methyl index -d <fast5_dir> <fastq.gz>'. Then align basecalled reads to the matching reference (hg38 for human datasets, E. coli K-12 MG1655 GCA_000005845.2 for bacterial) using 'minimap2 -a -x map-ont' piped into 'samtools sort', and run 'samtools index' on the sorted BAM. Finally run './Galaxy-methyl call-methylation -t 4 -B 4.0M -r <fastq.gz> -b <sorted.bam> -g <reference.fasta>' to write methylation_calls.tsv; the -t flag launches 4 CPU threads and -B sets a 4.0M batch size. Galaxy-methyl parallelizes the methylation score calculation (Hidden Markov Model) step on GPU and pipelines the four call-methylation steps (event detection, alignment, final calibration, methylation score calculation), keeping three batches in flight so a thread executing a step for batch k waits on batch k-1 completing that step. Correctness is verified by comparing per-CpG log-likelihood ratios against Nanopolish on the same indexed, sorted inputs; performance is judged by wall-clock timing on Serversmall (Xeon E5-2620 + Tesla K20m) or Serverlarge (Xeon E5-2686 + Tesla K80).

## Related tools

- **Galaxy-methyl** (GPU-accelerated, pipelined call-methylation tool that indexes FAST5 signals and detects methylated cytosines via HMM) — https://github.com/fengyilin118/Galaxy-methyl
- **Nanopolish** (Reference tool for correctness validation of per-CpG log-likelihood ratios and CPU performance baseline) — https://github.com/jts/nanopolish
- **f5c** (Optimized CPU (OpenMP) baseline for the methylation score calculation comparison) — https://github.com/hasindu2008/f5c
- **minimap2** (Aligns basecalled ONT reads to the reference genome with the -a -x map-ont preset) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 alignment BAM prior to methylation calling) — https://www.htslib.org/

## Examples

```
./Galaxy-methyl call-methylation -t 4 -B 4.0M -r FAB41174.fastq.gz -b FAB41174.sorted.bam -g hg38.fasta > methylation_calls.tsv
```

## Evaluation signals

- methylation_calls.tsv is produced and per-CpG log-likelihood ratios match Nanopolish exactly on the shared CpG sites of the example dataset
- GPU methylation score (HMM) step is roughly 2x faster than CPU OpenMP f5c, with larger gains on compute-intensive human-genome datasets D1-D3
- Total call-methylation time shows ~36% reduction and ~4.8x speedup vs Nanopolish on Serversmall (34% / 3.2x on Serverlarge), ~35% average vs f5c
- Execution timeline log shows three batches concurrently in flight across the four pipeline steps with per-step ordering preserved
- Batch-count sweep on D6/Serverlarge shows minimum end-to-end time at batch count 3 among swept values 3-15

## Limitations

- Larger pipeline batch counts increase data-loading latency; best performance is at batch count 3, so very large batch counts degrade end-to-end time
- Results are almost the same as Nanopolish except a few differences due to floating point approximations (as documented for f5c), so bit-identity is not guaranteed
- Reported speedups were measured on specific hardware (Xeon E5-2620 + Tesla K20m; Xeon E5-2686 + Tesla K80) and may vary on other GPU/CPU configurations
- Speedup advantage is smaller on less compute-intensive datasets (D4-D6) than on the compute-intensive human-genome datasets D1-D3

## Evidence

- [abstract] abstract: "Galaxy-methyl can achieve 3×–5× speedup compared with Nanopolish"
- [intro] intro: "Galaxy-methyl modified f5c to parallelize and optimize the methylation score calculation step on GPU and then pipeline the four steps of the call-methylation module"
- [methods] methods: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [readme] readme: "git clone https://github.com/fengyilin118/Galaxy-methyl.git"

---
name: gpu-cpu-execution-time-benchmarking
description: Use when comparing wall-clock execution times of a GPU-accelerated tool
  against a CPU baseline (e.g. Galaxy-methyl vs f5c vs Nanopolish) on ONT methylation
  calling, or when sweeping pipeline parameters like batch count.
when_to_use_negative:
- The tool input is not ONT basecalled reads with raw signal FAST5 data (e.g. short-read
  or PacBio data), since call-methylation benchmarking requires Nanopore signal-level
  inputs
- You need correctness validation of methylation calls rather than execution-time
  comparison; this skill times pipelines and does not evaluate methylation call accuracy
- Nanopore R10.4 flowcell data, since Nanopolish does not support R10.4 for methylation
  calling per its README
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Galaxy-methyl
  role: GPU-accelerated, pipelined call-methylation implementation being timed; also
    swept over batch counts 3-15
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: f5c
  role: CPU (OpenMP) baseline re-implementation of call-methylation used as the timing
    comparison baseline
  repo: https://github.com/hasindu2008/f5c
- name: Nanopolish
  role: Original CPU call-methylation implementation timed to quantify the 3x-5x Galaxy-methyl
    speedup
  repo: https://github.com/jts/nanopolish
- name: minimap2
  role: Aligns basecalled ONT reads (-a -x map-ont) to the reference genome as the
    pre-benchmarking alignment step
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the BAM so timed call-methylation runs consume identical
    indexed inputs
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:2843f524c496b62ef54af74c597a0704c80301a5a9264221e186d39424957d94
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gpu-cpu-execution-time-benchmarking

## Summary

Benchmark methodology for quantifying the GPU-accelerated and pipelined call-methylation latency of Galaxy-methyl against CPU-based f5c and Nanopolish, replicating the ~2x GPU HMM speedup, 35% average reduction vs f5c, and 3x-5x speedup vs Nanopolish reported in Feng et al. 2022 (Bioinformatics Advances, vbac088).

## When to use

Use when comparing wall-clock execution times of a GPU-accelerated tool against a CPU baseline (e.g. Galaxy-methyl vs f5c vs Nanopolish) on ONT methylation calling, or when sweeping pipeline parameters like batch count.

## When NOT to use

- The tool input is not ONT basecalled reads with raw signal FAST5 data (e.g. short-read or PacBio data), since call-methylation benchmarking requires Nanopore signal-level inputs
- You need correctness validation of methylation calls rather than execution-time comparison; this skill times pipelines and does not evaluate methylation call accuracy
- Nanopore R10.4 flowcell data, since Nanopolish does not support R10.4 for methylation calling per its README

## Inputs

- FAST5 signal files and FASTQ basecalled reads for ENA datasets ERR2184700, ERR2184710, ERR2184719 (PRJEB23027) and ERR1676719, ERR1676724, ERR167672 (PRJEB13021)
- hg38 FASTA human reference genome (D1-D3, D5-D6)
- Escherichia coli str. k_12 substr. MG1655 reference genome GCA_000005845.2 (D4)
- Sorted, indexed BAM files produced via minimap2 -a -x map-ont and samtools sort/index

## Outputs

- Per-dataset wall-clock execution-time table for Galaxy-methyl, f5c, and Nanopolish call-methylation on Serversmall and Serverlarge
- Tabulated GPU methylation score calculation (HMM step) times vs CPU f5c times across D1-D6
- Execution-time-versus-batch-count curve (batch counts 3-15) identifying the minimum end-to-end time
- Methylation_calls.tsv output per timed run

## How to apply

Download the six ENA datasets (ERR2184700, ERR2184710, ERR2184719 from PRJEB23027 labelled D1-D3; ERR1676719, ERR1676724, ERR167672 from PRJEB13021 labelled D4-D6) and match references (hg38 for D1-D3 and D5-D6; E. coli str. k_12 MG1655 GCA_000005845.2 for D4). For each tool and dataset, run the index step to link read ids to FAST5 signal data, align basecalled reads with minimap2 -a -x map-ont piped into samtools sort, and index the BAM. Then time the call-methylation runs on Serversmall (Xeon E5-2620 + Tesla K20m) and Serverlarge (Xeon E5-2686 + Tesla K80) with identical settings: 4 CPU threads (-t 4) and batch size 4.0M (-B 4.0M). Separately isolate the GPU methylation score calculation (HMM) step and sweep pipeline batch counts 3-15 on D6 on Serverlarge to confirm batch count 3 minimizes end-to-end time. Aggregate per-dataset latencies into a comparison table mirroring Figure 6 and the execution-time table of the paper.

## Related tools

- **Galaxy-methyl** (GPU-accelerated, pipelined call-methylation implementation being timed; also swept over batch counts 3-15) — https://github.com/fengyilin118/Galaxy-methyl
- **f5c** (CPU (OpenMP) baseline re-implementation of call-methylation used as the timing comparison baseline) — https://github.com/hasindu2008/f5c
- **Nanopolish** (Original CPU call-methylation implementation timed to quantify the 3x-5x Galaxy-methyl speedup) — https://github.com/jts/nanopolish
- **minimap2** (Aligns basecalled ONT reads (-a -x map-ont) to the reference genome as the pre-benchmarking alignment step) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the BAM so timed call-methylation runs consume identical indexed inputs) — https://www.htslib.org/

## Examples

```
./Galaxy-methyl call-methylation -t 4 -B 4.0M -r <fastq.gz> -b <sorted.bam> -g <reference.fasta>
```

## Evaluation signals

- GPU methylation score calculation (HMM) shows approximately 2x (or greater) speedup versus CPU f5c, with larger gains on compute-intensive datasets D1-D3
- Total call-methylation execution time of Galaxy-methyl is reduced by ~36% vs f5c on Serversmall and ~34% on Serverlarge, reproducing Figure 6
- Galaxy-methyl achieves ~4.8x average speedup vs Nanopolish on Serversmall and ~3.2x on Serverlarge (3x-5x overall)
- The batch-count sweep on D6/Serverlarge yields its minimum end-to-end time at batch count 3, with larger counts increasing data-loading latency
- All three tools run on identical inputs (same index, same sorted/indexed BAM) with matched settings: -t 4 and -B 4.0M

## Limitations

- Galaxy-methyl requires an NVIDIA GPU (compiled with make cuda=1 CUDA_LIB=/path/to/cuda/library/), so timings are only meaningful on the evaluated hardware classes (Tesla K20m, Tesla K80)
- The reported 35% average reduction vs f5c applies to total call-methylation time across these six datasets; gains depend on how compute-intensive the methylation score calculation is for a given dataset (best on human-genome D1-D3)
- Larger pipeline batch counts increase data-loading latency, so timing results from a non-optimal batch count do not reflect the best achievable Galaxy-methyl performance

## Evidence

- [abstract] 35% average reduction vs f5c: "reduce the total execution time by 35% compared with f5c, on average"
- [abstract] 3x-5x speedup vs Nanopolish: "Galaxy-methyl can achieve 3×–5× speedup compared with Nanopolish"
- [intro] GPU parallelization of methylation score calculation: "Galaxy-methyl modified f5c to parallelize and optimize the methylation score calculation step on GPU"
- [methods] Index step linking reads to FAST5 signals: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] Alignment and BAM preprocessing with minimap2/samtools: "The output is piped directly into samtools sort to get a sorted bam file"
- [readme] Galaxy-methyl builds with CUDA enabled: "make cuda=1 CUDA_LIB=/path/to/cuda/library/"
- [readme] R10.4 not supported by Nanopolish methylation calling: [evidence span withheld — anchor status: external_doc]

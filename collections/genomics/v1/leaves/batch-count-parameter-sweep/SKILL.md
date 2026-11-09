---
name: batch-count-parameter-sweep
description: 'Use when tuning or validating the batch count of Galaxy-methyl''s pipelined
  call-methylation module: given FAST5 signals plus basecalled Nanopore reads, you
  need the batch count with lowest end-to-end wall-clock time on your hardware.'
when_to_use_negative:
- You are tuning batch counts on a different dataset or server configuration — the
  latency-optimal batch count (3 for D6 on Serverlarge) need not transfer to other
  hardware or input sizes.
- You are comparing call-methylation tools on accuracy/methylation output rather than
  optimizing execution time — this skill only measures wall-clock performance.
- Your FAST5/signal data is for R10.4 flowcells, which nanopolish does not support
  for methylation calling per the README.
tools:
- name: Galaxy-methyl
  role: Runs the pipelined call-methylation module whose batch count is swept; executed
    once per batch count value with -t 4 CPU threads
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: minimap2
  role: Aligns the basecalled Nanopore reads to the GRCh38/hg38 reference with the
    map-ont preset before call-methylation timing runs
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: Sorts and indexes the minimap2 BAM output to produce the sorted BAM consumed
    by call-methylation
  repo: https://www.htslib.org/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:889ec6af1235b8ad22dacdf6f7fd8b73a91bae28a4bfe5f0c98711517429e75c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fengyilin118/Galaxy-methyl
  tools:
  - Galaxy-methyl
  - minimap2
  - samtools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# batch-count-parameter-sweep

## Summary

Systematic sweep of the Galaxy-methyl pipeline batch count (3-15) to find the value minimizing end-to-end call-methylation execution time, validating the pipelined design on a fixed dataset and server.

## When to use

Use when tuning or validating the batch count of Galaxy-methyl's pipelined call-methylation module: given FAST5 signals plus basecalled Nanopore reads, you need the batch count with lowest end-to-end wall-clock time on your hardware.

## When NOT to use

- You are tuning batch counts on a different dataset or server configuration — the latency-optimal batch count (3 for D6 on Serverlarge) need not transfer to other hardware or input sizes.
- You are comparing call-methylation tools on accuracy/methylation output rather than optimizing execution time — this skill only measures wall-clock performance.
- Your FAST5/signal data is for R10.4 flowcells, which nanopolish does not support for methylation calling per the README.

## Inputs

- Dataset D6 FAST5 signal files (ENA accession ERR167672, project PRJEB13021)
- Basecalled Nanopore reads (FASTQ) for D6
- GRCh38/hg38 FASTA human reference genome used for D5/D6

## Outputs

- Sorted, indexed BAM of D6 reads aligned to the human reference
- Table of end-to-end wall-clock execution times for batch counts 3-15
- Plot of execution time versus batch count identifying batch count 3 as the minimum

## How to apply

Download dataset D6 (ENA accession ERR167672 from project PRJEB13021) with its FAST5 signal files and the human reference genome (GRCh38/hg38 FASTA). Build the Galaxy-methyl index linking read ids to signal-level data in the FAST5 files, then align basecalled reads to the reference with minimap2 (-a -x map-ont) and pipe into samtools sort/index to obtain a sorted BAM. For each batch count from 3 to 15, run Galaxy-methyl call-methylation with 4 CPU threads (-t 4), timing each run end-to-end; the rationale is that the pipelined design trades batch granularity against data-loading latency, so the timing curve is expected to bottom out at small batch counts (3) rather than monotonically improve. Tabulate the wall-clock times and plot execution time versus batch count; the minimum of the curve is the optimal configuration.

## Related tools

- **Galaxy-methyl** (Runs the pipelined call-methylation module whose batch count is swept; executed once per batch count value with -t 4 CPU threads) — https://github.com/fengyilin118/Galaxy-methyl
- **minimap2** (Aligns the basecalled Nanopore reads to the GRCh38/hg38 reference with the map-ont preset before call-methylation timing runs) — https://github.com/lh3/minimap2
- **samtools** (Sorts and indexes the minimap2 BAM output to produce the sorted BAM consumed by call-methylation) — https://www.htslib.org/

## Examples

```
git clone https://github.com/fengyilin118/Galaxy-methyl.git; cd Galaxy-methyl; make cuda=1 CUDA_LIB=/path/to/cuda/library/
```

## Evaluation signals

- The execution-time versus batch-count curve has its minimum at batch count 3 among the swept values 3-15, matching Figure 7 of the paper.
- Execution time increases for batch counts larger than 3, consistent with rising data-loading latency at larger batch counts.
- Every sweep run used identical resources: 4 CPU threads (-t 4), same D6 input, same Serverlarge machine.
- Each timed run produces a complete, valid sorted BAM plus methylation calls, so timing differences are not due to truncated runs.
- Call-methylation outputs remain consistent with Nanopolish/f5c semantics (Galaxy-methyl is a parallelized call-methylation module in Nanopolish).

## Limitations

- Larger batch counts increase data-loading latency, so sweeping upward from small values is expected, not monotonically improving.
- The optimum (batch count 3) was determined for dataset D6 on Serverlarge; it may differ on other hardware or datasets.
- Nanopolish does not support R10.4 flowcells for variant and methylation calling, and f5c requires --pore r10 (or rna004) for FAST5 input on newer chemistries, which affects which inputs can be swept.

## Evidence

- [methods] create an index file that links read ids with their signal-level data in the FAST5 files: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [methods] The output is piped directly into samtools sort to get a sorted bam file: "The output is piped directly into samtools sort to get a sorted bam file"
- [methods] Finally, detect methylated bases. -t INT denotes how many threads are launched on CPU cores: "Finally, detect methylated bases. -t INT denotes how many threads are launched on CPU cores"
- [abstract] then pipelines the four steps of the call-methylation module: "then pipelines the four steps of the call-methylation module"
- [readme] Galaxy-methyl is an parallelized and optimized call-methylation module in Nanopolish: "Galaxy-methyl is an parallelized and optimized call-methylation module in Nanopolish"
- [readme] nanopolish does not support R10.4 flowcells as variant and methylation calling is accurate enough to not require signal-level analysis: [evidence span withheld — anchor status: external_doc]

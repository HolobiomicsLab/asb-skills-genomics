---
name: performance-curve-reproduction-and-verification
description: Use when a task asks to reproduce a reported optimal setting (e.g. batch
  count 3) of a pipelined bioinformatics tool by re-running timed end-to-end sweeps
  on the same dataset, reference, and thread count as the original study.
when_to_use_negative:
- The goal is accuracy/quality evaluation of methylation calls (e.g. comparing Galaxy-methyl
  vs Nanopolish speedup 3x–5x or f5c 35% time reduction) rather than reproducing a
  parameter-sweep timing curve
- The pipeline parameter to sweep or the dataset/hardware differs from the published
  setup, so Figure 7 curve agreement cannot be expected as validation
- Signal data are unavailable (e.g. no FAST5 files), since the index linking read
  ids to signal data is a prerequisite
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Galaxy-methyl
  role: GPU-accelerated, pipelined call-methylation module whose batch count is swept
    and timed end-to-end
  repo: https://github.com/fengyilin118/Galaxy-methyl
- name: minimap2
  role: Aligns basecalled ONT reads to the human reference with -a -x map-ont before
    methylation calling
  repo: https://github.com/lh3/minimap2
- name: samtools
  role: sort/index the BAM produced by minimap2 alignment
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac088
    title: GPU-accelerated and pipelined methylation calling
  source_content_hash: sha256:70b958b1f98d73a488911839ea2047c4ffb948584577f8ce42e88f76b2c18e98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fengyilin118/Galaxy-methyl
  tools:
  - Galaxy-methyl
  - minimap2
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# performance-curve-reproduction-and-verification

## Summary

Reproduce and verify a published performance-optimization curve by sweeping a pipeline parameter (Galaxy-methyl's call-methylation batch count over 3–15) end-to-end on a fixed dataset and hardware, then comparing wall-clock timings against the paper's figure.

## When to use

Use when a task asks to reproduce a reported optimal setting (e.g. batch count 3) of a pipelined bioinformatics tool by re-running timed end-to-end sweeps on the same dataset, reference, and thread count as the original study.

## When NOT to use

- The goal is accuracy/quality evaluation of methylation calls (e.g. comparing Galaxy-methyl vs Nanopolish speedup 3x–5x or f5c 35% time reduction) rather than reproducing a parameter-sweep timing curve
- The pipeline parameter to sweep or the dataset/hardware differs from the published setup, so Figure 7 curve agreement cannot be expected as validation
- Signal data are unavailable (e.g. no FAST5 files), since the index linking read ids to signal data is a prerequisite

## Inputs

- Dataset D6: ENA accession ERR167672 from project PRJEB13021, with FAST5 raw signal files
- GRCh38 (hg38) FASTA human reference genome primary assembly
- Basecalled Nanopore reads (FASTQ)
- Built Galaxy-methyl source (make cuda=1)

## Outputs

- Table of wall-clock execution times per batch count (3–15)
- Plot of end-to-end execution time versus pipeline batch count
- Verified conclusion that batch count 3 yields minimum execution time (Figure 7 reproduction)

## How to apply

Download the exact dataset (ENA accession ERR167672 from project PRJEB13021, i.e. D6) plus the GRCh38/hg38 primary assembly human reference used for D5/D6. Build the Galaxy-methyl index linking read ids to FAST5 signal data, align basecalled reads with minimap2 (-a -x map-ont) and pipe into samtools sort/index, then run Galaxy-methyl call-methylation for each batch count from 3 to 15 with 4 CPU threads (-t 4), timing each end-to-end run. Collect wall-clock execution times into a table and plot execution time versus batch count. The correct outcome is that batch count 3 minimizes end-to-end execution time on D6 on Serverlarge, matching Figure 7; larger batch counts should increase data-loading latency.

## Related tools

- **Galaxy-methyl** (GPU-accelerated, pipelined call-methylation module whose batch count is swept and timed end-to-end) — https://github.com/fengyilin118/Galaxy-methyl
- **minimap2** (Aligns basecalled ONT reads to the human reference with -a -x map-ont before methylation calling) — https://github.com/lh3/minimap2
- **samtools** (sort/index the BAM produced by minimap2 alignment)

## Examples

```
./minimap2 -ax map-ont ref.fa ont.fq.gz > aln.sam; samtools sort -o aln.bam aln.sam; samtools index aln.bam
```

## Evaluation signals

- The timing-versus-batch-count curve shows its minimum at batch count 3 among swept values 3–15, matching Figure 7
- Larger batch counts exhibit increased data-loading latency (rising wall-clock time), consistent with the paper's explanation
- Every run uses identical settings otherwise: 4 CPU threads (-t 4), same ERR167672 dataset, same GRCh38 reference
- Alignment stage output is a sorted, indexed BAM (minimap2 piped into samtools sort/index) before call-methylation runs
- Reported times are end-to-end wall-clock durations per batch count, collected into a table and plotted

## Limitations

- The optimal batch count (3) is dataset- and hardware-specific (D6 on Serverlarge); other datasets or machines may shift the optimum
- Sweep range limited to batch counts 3–15; behavior outside this range is not established
- Galaxy-methyl requires GPU support (built with make cuda=1 and a CUDA library path); without CUDA the pipelined/GPU behavior cannot be reproduced

## Evidence

- [methods] workflow sweep: "align the basecalled reads to the reference genome.  minimap2 is used to map reads to the human genome"
- [abstract] pipelining rationale: "pipelines the four steps of the call-methylation module"
- [methods] index step: "create an index file that links read ids with their signal-level data in the FAST5 files"
- [readme] build command: "git clone https://github.com/fengyilin118/Galaxy-methyl.git"
- [methods] dataset provenance: "Three datasets under accession ERR1676719, ERR1676724 and ERR167672 in the project PRJEB13021"

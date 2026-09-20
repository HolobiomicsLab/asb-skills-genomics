---
name: assembly-contiguity-accuracy-benchmarking
description: Use when comparing genome assemblers on real long-read/short-read datasets
  and you need contiguity, accuracy, runtime, memory, and failure-rate metrics compiled
  into a benchmark table.
when_to_use_negative:
- Input is a single assembly already produced — QUAST evaluation alone suffices; no
  comparative benchmarking is needed
- Datasets lack matched short reads for a hybrid assembler, since HASLR-style hybrid
  comparison requires both long and short reads from the same sample
- The goal is simulated-read benchmarking rather than comparison on the real publicly
  available datasets (E. coli, yeast, C. elegans, human)
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3693
tools:
- name: HASLR
  role: Hybrid assembler under benchmark; run via haslr.py with -x {pacbio,nanopore},
    -g genome size, --cov-lr 25, and k=49 Minia contigs
  repo: https://github.com/vpc-ccg/haslr
- name: PBcR
  role: Comparator hybrid assembler run under matched compute resources
- name: MaSuRCA
  role: Comparator hybrid assembler run under matched compute resources
- name: minimap2
  role: HASLR dependency that aligns short read contigs onto long reads during assembly
  repo: https://github.com/lh3/minimap2
- name: Minia
  role: HASLR dependency that assembles Illumina short reads into contigs (k=49)
  repo: https://github.com/GATB/minia
- name: SPOA
  role: HASLR dependency for consensus calling of long reads between anchor contigs
  repo: https://github.com/rvaser/spoa
- name: QUAST
  role: Assembly evaluation producing contiguity and accuracy metrics (NGA50, genome
    fraction, misassemblies) for the benchmark table
- name: fastutils
  role: FASTA/FASTQ manipulation utilities (stat, length, subsample) used when preparing
    long/short read inputs
  repo: https://github.com/haghshenas/fastutils
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101389
    title: 'HASLR: Fast Hybrid Assembly of Long Reads'
  source_content_hash: sha256:8ee70a7896a3099f393196bfe97fe2e619ca31928955cb1caeb1657347cc4521
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/vpc-ccg/haslr
  repo_url: https://github.com/vpc-ccg/haslr
  tools:
  - HASLR
  - MaSuRCA
  - Minia
  - Oxford Nanopore
  - PBcR
  - Pilon
  - QUAST
  - Racon
  - SPOA
  - fastutils
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3693
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# assembly-contiguity-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark hybrid genome assemblers on real datasets by evaluating assemblies with QUAST (NGA50, genome fraction, misassemblies) and recording wall-clock runtime, peak memory, and tool failures per dataset/tool combination.

## When to use

Use when comparing genome assemblers on real long-read/short-read datasets and you need contiguity, accuracy, runtime, memory, and failure-rate metrics compiled into a benchmark table.

## When NOT to use

- Input is a single assembly already produced — QUAST evaluation alone suffices; no comparative benchmarking is needed
- Datasets lack matched short reads for a hybrid assembler, since HASLR-style hybrid comparison requires both long and short reads from the same sample
- The goal is simulated-read benchmarking rather than comparison on the real publicly available datasets (E. coli, yeast, C. elegans, human)

## Inputs

- Paired long-read and short-read FASTA/FASTQ datasets (ONT or PacBio long reads plus Illumina short reads) from the same sample
- Assembler outputs (contig FASTA files) from each tool/dataset combination
- Compute resource logs (wall-clock runtime, peak memory, failure status per run)

## Outputs

- QUAST evaluation metrics per assembly (NGA50, genome fraction, misassemblies)
- Runtime, peak memory, and tool-failure records per dataset/tool combination
- Compiled benchmark table (e.g., benchmark_results.csv)

## How to apply

Retrieve matched long-read (PacBio or Oxford Nanopore) and Illumina short-read datasets per organism — e.g., E. coli ONT ultra-long reads from the Loman lab plus MG1655 MiSeq reads, yeast PacBio ERX1725434/ERX1725435/ERX1725441 with Illumina ERX1943903, C. elegans PacBio from the PacificBiosciences DevNet wiki with Illumina SRR065390, and CHM1 human PacBio from SRA study SRP044331 with Illumina SRX652547. Assemble each dataset with the assembler under test (e.g., haslr.py with 64 threads, -x {pacbio,nanopore} to set long-read type, -g with k/m/g genome-size suffixes, --cov-lr 25 default long-read coverage, and k=49 Minia short-read contigs) and with comparators such as PBcR and MaSuRCA under matched compute resources. Evaluate every assembly with QUAST to obtain NGA50, genome fraction, and misassembly counts. Record wall-clock time, peak memory, and any failures (e.g., out-of-memory kills or non-terminating runs) per dataset/tool combination, then compile everything into a benchmark results table (e.g., benchmark_results.csv).

## Related tools

- **HASLR** (Hybrid assembler under benchmark; run via haslr.py with -x {pacbio,nanopore}, -g genome size, --cov-lr 25, and k=49 Minia contigs) — https://github.com/vpc-ccg/haslr
- **PBcR** (Comparator hybrid assembler run under matched compute resources)
- **MaSuRCA** (Comparator hybrid assembler run under matched compute resources)
- **minimap2** (HASLR dependency that aligns short read contigs onto long reads during assembly) — https://github.com/lh3/minimap2
- **Minia** (HASLR dependency that assembles Illumina short reads into contigs (k=49)) — https://github.com/GATB/minia
- **SPOA** (HASLR dependency for consensus calling of long reads between anchor contigs) — https://github.com/rvaser/spoa
- **QUAST** (Assembly evaluation producing contiguity and accuracy metrics (NGA50, genome fraction, misassemblies) for the benchmark table)
- **fastutils** (FASTA/FASTQ manipulation utilities (stat, length, subsample) used when preparing long/short read inputs) — https://github.com/haghshenas/fastutils

## Examples

```
haslr.py -t 64 -x nanopore -g 4.6m --lr ont_reads.fastq --sr illumina_R1.fastq illumina_R2.fastq -o asm_out && quast.py asm_out/assembly.fasta
```

## Evaluation signals

- QUAST reports NGA50, genome fraction, and misassembly counts for every assembly in the table; missing rows indicate unexplained failures
- The reproduced table shows the reported pattern: HASLR is the fastest assembler with the lowest number of misassemblies on most samples (e.g., CHM1 human assembled in less than 10 hours using 64 CPU threads)
- Runtime and peak memory are recorded for each dataset/tool combination, including failures such as Flye/SPAdes/hybridSPAdes/Unicycler exceeding memory limits on human
- All four real datasets (E. coli, yeast, C. elegans, human) are assembled and evaluated with matched compute resources across assemblers
- benchmark_results.csv is complete with no missing QUAST metrics for successfully completed assemblies

## Limitations

- HASLR usually has a higher indel rate compared with other tools; most small local mistakes can be fixed by further polishing
- HASLR sometimes generates assemblies with relatively lower genome fraction and/or NGA50 compared with other tools, especially Canu on the human genome
- Regions with high heterozygosity are not resolved because Minia short read contigs are fragmented and filtered out
- The current version of HASLR does not generate a phased assembly for diploid genomes
- Connectivity of the backbone graph depends on the existence of unique SR contigs; lack of them results in multiple connected components

## Evidence

- [abstract] Our experiments show that HASLR is not only the fastest assembler but also the one with the lowest number of misassemblies on most of the samples: "Our experiments show that HASLR is not only the fastest assembler but also the one with the lowest number of misassemblies on most of the samples"
- [methods] it can assemble a CHM1 human dataset in less than 10 hours using 64 CPU threads: "it can assemble a CHM1 human dataset in less than 10 hours using 64 CPU threads."
- [discussion] PacBio reads can be obtained at https://trace.ncbi.nlm.nih.gov/Traces/sra/?study=SRP044331: "PacBio reads can be obtained at https://trace.ncbi.nlm.nih.gov/Traces/sra/?study=SRP044331"
- [discussion] compared with other tools, HASLR usually has a higher indel rate: "compared with other tools, HASLR usually has a higher indel rate"
- [readme] HASLR is a hybrid tool which means it requires long reads generated by Third Generation Sequencing technologies: "HASLR is a hybrid tool which means it requires long reads generated by Third Generation Sequencing technologies"

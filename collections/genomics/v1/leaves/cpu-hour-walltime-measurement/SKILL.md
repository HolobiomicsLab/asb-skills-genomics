---
name: cpu-hour-walltime-measurement
description: Use when benchmarking runtime, peak memory (GB), thread count, and CPU
  hours of VolcanoSV-asm or alternative assemblers (hifiasm, Flye) on HG002 HiFi/CLR/ONT
  WGS BAMs to populate a resource_usage table.
when_to_use_negative:
- The goal is SV-calling accuracy evaluation rather than resource profiling — that
  requires Truvari benchmarking (p=0.5, P=0.5, r=500, O=0.01) against the GIAB gold
  standard, not walltime/memory capture
- Running the single-chromosome test example on chr10 from Zenodo — its cost is not
  representative of whole-genome resource usage
- The sample is CHM13 or another homozygous/single-haploid source, for which the phased
  diploid assembly component is unsuitable and its resource profile is not the published
  benchmark
tools:
- name: VolcanoSV
  role: volcanosv-asm.py is the pipeline being measured; per-chromosome assembly with
    selectable assembler (-dtype Hifi/CLR/ONT)
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: Default assembler used for the HiFi (Hifi_L1) resource-usage measurement
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: Default assembler used for the CLR (CLR_L1) and ONT (ONT_L1) resource-usage
    measurements
  repo: https://github.com/fenderglass/Flye
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:96b8e018e945e2cab7450468d58d4c2b34978625cc6c250e65b199868161a449
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Flye
  - VolcanoSV
  - hifiasm
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cpu-hour-walltime-measurement

## Summary

Protocol for reproducing the VolcanoSV-asm computational resource benchmark: measuring wall-clock runtime, peak memory, ncpu, and derived CPU hours (walltime × ncpu) for per-chromosome diploid assembly on HG002 long-read BAMs, and comparing against alternative pipelines (hifiasm for HiFi; Flye+HapDup for CLR/ONT).

## When to use

Use when benchmarking runtime, peak memory (GB), thread count, and CPU hours of VolcanoSV-asm or alternative assemblers (hifiasm, Flye) on HG002 HiFi/CLR/ONT WGS BAMs to populate a resource_usage table.

## When NOT to use

- The goal is SV-calling accuracy evaluation rather than resource profiling — that requires Truvari benchmarking (p=0.5, P=0.5, r=500, O=0.01) against the GIAB gold standard, not walltime/memory capture
- Running the single-chromosome test example on chr10 from Zenodo — its cost is not representative of whole-genome resource usage
- The sample is CHM13 or another homozygous/single-haploid source, for which the phased diploid assembly component is unsuitable and its resource profile is not the published benchmark

## Inputs

- HG002 PacBio HiFi WGS BAM (Hifi_L1, ~56x)
- HG002 PacBio CLR WGS BAM (CLR_L1, ~89x)
- HG002 Nanopore WGS BAM (ONT_L1, ~48x)
- Reference FASTA (e.g. hg19 genome.fa) with .fai index
- chrom_list text file listing autosome contig names, one per line

## Outputs

- resource_usage.tsv with columns Data Type, Coverage, Memory, ncpu, Run time, CPU hours
- Final diploid contig files (phased assembly output) per data type

## How to apply

Obtain the HG002 long-read inputs — Hifi_L1 (~56x PacBio HiFi), CLR_L1 (~89x PacBio CLR), and ONT_L1 (~48x Nanopore) — plus the reference FASTA and its .fai index. Because the VolcanoSV assembly pipeline is designed to run by chromosomes, run ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py in WGS mode with a --chrom_list file of autosome names, using -dtype Hifi with the default hifiasm assembler and -t 30 for the HiFi BAM, -dtype CLR with the default Flye assembler and -t 20 for CLR, and -dtype ONT with Flye and -t 30 for ONT, wrapping each run with a timing/memory monitor (e.g. /usr/bin/time -v or sacct) to capture walltime and peak RSS. For each run, record Data Type, Coverage, Memory, ncpu, Run time, and CPU hours computed as walltime × ncpu, and write the three-row table to resource_usage.tsv. Validate by comparing against the published values (HiFi: 168GB, 2-01:07:59, 1474 CPU hours; CLR: 259GB, 5-09:05:59, 2582 CPU hours; ONT: 85GB, 6-02:33:59, 4397 CPU hours) and by checking each run produces a final contig file of roughly the expected assembly size.

## Related tools

- **VolcanoSV** (volcanosv-asm.py is the pipeline being measured; per-chromosome assembly with selectable assembler (-dtype Hifi/CLR/ONT)) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (Default assembler used for the HiFi (Hifi_L1) resource-usage measurement) — https://github.com/chhylp123/hifiasm
- **Flye** (Default assembler used for the CLR (CLR_L1) and ONT (ONT_L1) resource-usage measurements) — https://github.com/fenderglass/Flye

## Examples

```
/usr/bin/time -v python volcanosv-asm.py -bam HG002_Hifi_L1.bam -ref genome.fa -o out_hifi -t 30 -dtype Hifi -cl chrom_list.txt 2> hifi_time.log
```

## Evaluation signals

- Reproduced table matches published values: HiFi 168GB / 2-01:07:59 / 1474 CPU hours; CLR 259GB / 5-09:05:59 / 2582 CPU hours; ONT 85GB / 6-02:33:59 / 4397 CPU hours
- CPU hours column equals walltime × ncpu for every row (e.g. HiFi: 2-01:07:59 × 30 ≈ 1474 CPU hours)
- Each run produces a final contig file of roughly the expected assembly size (~6Gb diploid)
- Relative ordering holds: VolcanoSV-asm uses less memory and less runtime than Flye+HapDup, and less memory but more runtime than hifiasm
- Thread settings match the protocol: -t 30 for HiFi and ONT, -t 20 for CLR

## Limitations

- VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes differs, so resource profiles exclude chrX/chrY
- Published values are tied to specific thread counts and hardware; absolute runtime and memory will vary on different machines or ncpu settings
- Cross-pipeline comparison confounds assembler choice with the phasing and local assembly modules of VolcanoSV-asm

## Evidence

- [abstract] utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly: "utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly"
- [readme] The VolcanoSV assembly pipeline is designed to run by chromosomes: "The VolcanoSV assembly pipeline is designed to run by chromosomes."
- [readme] n_thread N_THREAD, -t N_THREAD argument controls threads used in the benchmarked runs: "--n_thread N_THREAD, -t N_THREAD"
- [intro] VolcanoSV is compatible with all mainstream long-read sequencing platforms, which vary considerably in sequencing error rates: "VolcanoSV is compatible with all mainstream long-read sequencing platforms, which vary considerably in sequencing error rates"

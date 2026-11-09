---
name: long-read-alignment-benchmarking
description: Use when comparing computational cost (wall-clock runtime, CPU hours,
  peak memory) of TE callers on PacBio or ONT long-read BAM files, or when reproducing
  the sTELLeR vs TLDR/xTEA efficiency claims on GIAB trio data.
when_to_use_negative:
- Input reads are short-read (Illumina) or single-cell data rather than PacBio HiFi/CLR
  or ONT long reads — sTELLeR's --style flags and detection logic are designed and
  tested for long-read data
- The goal is assessing variant-calling accuracy/precision rather than computational
  cost — this skill measures runtime and CPU hours, not TE detection performance metrics
- Input is an unaligned assembly or reads without a reference BAM — the benchmarking
  protocol requires minimap2-aligned BAM files per sample and platform
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: sTELLeR
  role: Primary TE caller being benchmarked; run with --style pb or --style ont on
    each trio sample BAM with default thresholds (sr 3, mq 20, maxreads 100)
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Aligns PacBio and ONT long reads to GRCh38 to produce the BAM files consumed
    identically by all benchmarked callers
- name: samtools
  role: Dependency for BAM handling during sTELLeR execution
- name: pysam
  role: Python dependency sTELLeR uses to read BAM records during TE detection
- name: TLDR
  role: Competitive TE caller run on identical BAM inputs as the runtime/CPU-hours
    baseline
- name: xTEA
  role: Competitive TE caller run on identical BAM inputs as the runtime/CPU-hours
    baseline
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:4d6bd87b77ae60a31b32f02c00f8d1249124c7d8127aa614bc4eb865fedb7b69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kristinebilgrav/sTELLeR
  tools:
  - TLDR
  - minimap2
  - pysam
  - python3
  - sTELLeR
  - samtools
  - xTEA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# long-read-alignment-benchmarking

## Summary

Benchmark the runtime, CPU hours, and memory footprint of long-read transposable element callers (sTELLeR vs TLDR and xTEA) on identical BAM inputs from the GIAB Ashkenazim trio, validating sTELLeR's reported speed advantage (5-48x faster, <2% CPU hours).

## When to use

Use when comparing computational cost (wall-clock runtime, CPU hours, peak memory) of TE callers on PacBio or ONT long-read BAM files, or when reproducing the sTELLeR vs TLDR/xTEA efficiency claims on GIAB trio data.

## When NOT to use

- Input reads are short-read (Illumina) or single-cell data rather than PacBio HiFi/CLR or ONT long reads — sTELLeR's --style flags and detection logic are designed and tested for long-read data
- The goal is assessing variant-calling accuracy/precision rather than computational cost — this skill measures runtime and CPU hours, not TE detection performance metrics
- Input is an unaligned assembly or reads without a reference BAM — the benchmarking protocol requires minimap2-aligned BAM files per sample and platform

## Inputs

- PacBio and ONT long-read fastq files for GIAB Ashkenazim trio samples (HG002, HG003, HG004)
- GRCh38 reference genome (e.g. HG38.fa)
- Fasta file of TE insertion sequences to detect (e.g. TEfastasequences.fa with L1 and Alu consensus sequences)
- Alignment BAM files produced by minimap2, one per sample and platform
- TE fasta optionally including SVA and HERV sequences (TEsequences_SVA_HERV_ALU_L1.fasta)

## Outputs

- sTELLeR variant calls in VCF format (variants named INS:ME) per sample and platform
- TLDR and xTEA variant calls on the same BAM inputs
- te_caller_runtime_benchmark.csv with per-caller, per-sample wall-clock runtime, CPU hours, and peak memory

## How to apply

Obtain GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio and ONT fastq reads and align them to GRCh38 with minimap2 to produce per-sample, per-platform BAM files. Run sTELLeR (steller.py) on each BAM with --style pb for PacBio or --style ont for ONT, keeping default thresholds (-r/--sr 3, -m/--mq 20, -mr/--maxreads 100). Run TLDR and xTEA on the identical BAM inputs so cost comparisons are apples-to-apples. Profile every run with a process monitor such as /usr/bin/time to record wall-clock runtime, CPU hours (cores x time), and peak memory. Aggregate per-caller, per-sample measurements into te_caller_runtime_benchmark.csv and check the result reproduces sTELLeR completing a 60x PacBio genome in under 30 minutes.

## Related tools

- **sTELLeR** (Primary TE caller being benchmarked; run with --style pb or --style ont on each trio sample BAM with default thresholds (sr 3, mq 20, maxreads 100)) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Aligns PacBio and ONT long reads to GRCh38 to produce the BAM files consumed identically by all benchmarked callers)
- **samtools** (Dependency for BAM handling during sTELLeR execution)
- **pysam** (Python dependency sTELLeR uses to read BAM records during TE detection)
- **TLDR** (Competitive TE caller run on identical BAM inputs as the runtime/CPU-hours baseline)
- **xTEA** (Competitive TE caller run on identical BAM inputs as the runtime/CPU-hours baseline)

## Examples

```
python steller/steller.py --ref HG38_chr22.fa --TE_fasta fasta/TEfastasequences.fa --bam testdata.bam --sr 4 --style pb -o testdata_res -mr 80
```

## Evaluation signals

- sTELLeR completes a 60x PacBio genome in under 30 minutes
- sTELLeR is 5-48x as fast as TLDR and xTEA on the same BAM inputs
- sTELLeR uses <2% of the CPU hours of the competitive callers
- All callers consume byte-identical BAM inputs per sample/platform, so runtime differences are attributable to the callers
- Sanity-check the pipeline first: the chr22 test data should yield a total of 7 variants matching testbamTRUTH.txt before benchmarking at scale

## Limitations

- Minimum supporting split reads (--sr, default 3) depends on sample coverage and desired sensitivity; high-coverage samples may require --maxreads (default 100) to avoid calls in regions exceeding the given coverage
- Intermediate files (ID_chr_reads.fasta, ID_chr_repeats.sam) are deleted unless -k/--keep_intermediates is set, complicating post-hoc profiling of per-stage cost
- The ONT and pb styles are distinct code paths; benchmark results for one platform do not automatically transfer to the other

## Evidence

- [abstract] The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers.: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
- [methods] sTELLeR completing a 60x PacBio genome in under 30 minutes: "python steller/steller.py --ref < genome ref_file > --TE_fasta < TE sequence fasta file > --bam < bamfile > --sr 4 --style pb -o < output prefix > -mr 80"
- [readme] Default supporting-read threshold: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"
- [readme] Benchmark pipeline sanity check on chr22 test data: "a total of 7 variants should be identified"

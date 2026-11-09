---
name: computational-resource-profiling
description: Use when benchmarking the computational cost (runtime, CPU hours, peak
  memory) of variant or TE callers on matched long-read BAM inputs, e.g. comparing
  sTELLeR against TLDR and xTEA on the same samples and platforms.
when_to_use_negative:
- The goal is TE calling accuracy (precision/sensitivity) rather than computational
  efficiency — use a benchmarking skill against a truth set instead
- Input is short-read data — sTELLeR is designed and tested for long-read data (PacBio
  or ONT)
- Only a single caller needs to be run, with no comparative runtime/CPU-hour measurement
  required
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: sTELLeR
  role: TE caller under test; run with --style pb or ont on each BAM while profiling
    resource usage
  repo: https://github.com/kristinebilgrav/sTELLeR
- name: minimap2
  role: Align PacBio/ONT reads to GRCh38 to produce per-sample BAM inputs
- name: samtools
  role: Dependency for sTELLeR; BAM handling during alignment and calling
- name: pysam
  role: Dependency for sTELLeR's python-based caller
- name: TLDR
  role: Competitor TE caller run on identical BAMs for runtime comparison
- name: xTEA
  role: Competitor TE caller run on identical BAMs for runtime comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae686
    title: Detecting transposable elements in long-read genomes using sTELLeR
  source_content_hash: sha256:f27b6f68a2b7dddac04bd033cd5d2ba45dcf7bef85be5c9d485ac12eca35563a
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# computational-resource-profiling

## Summary

Profile the wall-clock runtime, CPU hours, and peak memory of TE callers (sTELLeR vs TLDR and xTEA) when run on GIAB Ashkenazim trio long-read BAMs, to benchmark computational efficiency across callers and platforms.

## When to use

Use when benchmarking the computational cost (runtime, CPU hours, peak memory) of variant or TE callers on matched long-read BAM inputs, e.g. comparing sTELLeR against TLDR and xTEA on the same samples and platforms.

## When NOT to use

- The goal is TE calling accuracy (precision/sensitivity) rather than computational efficiency — use a benchmarking skill against a truth set instead
- Input is short-read data — sTELLeR is designed and tested for long-read data (PacBio or ONT)
- Only a single caller needs to be run, with no comparative runtime/CPU-hour measurement required

## Inputs

- GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio fastq reads
- GIAB Ashkenazim trio (HG002, HG003, HG004) ONT fastq reads
- GRCh38 reference genome
- BAM files aligned per sample and platform (via minimap2)
- TE fasta sequences (e.g. TEfastasequences.fa)

## Outputs

- Per-caller, per-sample benchmark table of wall-clock runtime, CPU hours (cores x time), and peak memory (e.g. te_caller_runtime_benchmark.csv)
- TE insertion calls per caller (sTELLeR output as VCF, INS:ME variants)

## How to apply

Align GIAB Ashkenazim trio (HG002, HG003, HG004) PacBio and ONT fastq reads to GRCh38 with minimap2, producing one BAM per sample and platform. Run each caller on the identical BAM inputs: sTELLeR via steller.py with --style pb for PacBio and --style ont for ONT at default thresholds (-r/--sr 3, -m/--mq 20, -mr/--maxreads 100), and the competitors TLDR and xTEA on the same files. Profile each run with a process monitor such as /usr/bin/time to record wall-clock time, CPU hours (cores × time), and peak memory. Aggregate per-caller, per-sample measurements into a single benchmark table (e.g. te_caller_runtime_benchmark.csv). The rationale is that sTELLeR is designed to be CPU-light, so the benchmark directly tests the claimed efficiency advantage over competing TE callers.

## Related tools

- **sTELLeR** (TE caller under test; run with --style pb or ont on each BAM while profiling resource usage) — https://github.com/kristinebilgrav/sTELLeR
- **minimap2** (Align PacBio/ONT reads to GRCh38 to produce per-sample BAM inputs)
- **samtools** (Dependency for sTELLeR; BAM handling during alignment and calling)
- **pysam** (Dependency for sTELLeR's python-based caller)
- **TLDR** (Competitor TE caller run on identical BAMs for runtime comparison)
- **xTEA** (Competitor TE caller run on identical BAMs for runtime comparison)

## Examples

```
python steller/steller.py --ref GRCh38.fa --TE_fasta fasta/TEfastasequences.fa --bam HG002_pb.bam --style pb -o HG002_pb_res -mr 100
```

## Evaluation signals

- sTELLeR completes a 60x PacBio genome in under 30 minutes of wall-clock time
- sTELLeR is 5-48x as fast as the competitive callers across the benchmark matrix
- sTELLeR uses <2% of the CPU hours of TLDR and xTEA on identical inputs
- Runtime and CPU-hour measurements are recorded for every caller x sample x platform combination in the benchmark table
- Each caller ran on identical BAM inputs per sample and platform, so differences are attributable to the caller

## Limitations

- Benchmark results depend on the specific hardware and process monitor used; CPU hours are computed as cores x time, which may not reflect all schedulers
- sTELLeR timing assumes default thresholds (-r 3, -m 20, -mr 100); changing --sr/--mq/--maxreads alters both runtime and sensitivity

## Evidence

- [abstract] sTELLeR is substantially faster and more CPU-efficient than competitive callers TLDR and xTEA: "The caller is 5-48× as fast and uses <2% of the CPU hours compared to competitive callers."
- [intro] sTELLeR designed to be fast and CPU light: "sTELLeR is a fast and CPU light tool designed and tested for detection of transposable element insertion in long-read data"
- [readme] steller.py invocation requires bam, reference, TE fasta: "sTELLeR needs a bam or cram file, the reference used to align it, and a fasta file with insertions sequences to look for (--TE_fasta)"
- [readme] default thresholds for supporting reads, mapping quality, max reads: "-r , --sr         Minimum number of supporting split reads/insertions to call a variant (default 3)"

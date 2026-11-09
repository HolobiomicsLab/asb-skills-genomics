---
name: computational-resource-profiling
description: Use when benchmarking assembly or SV-calling pipelines on the same datasets
  and you must report memory consumption, runtime, ncpu, and CPU-hours per run to
  compare against published resource tables.
when_to_use_negative:
- When the goal is SV-calling accuracy (recall/precision/F1 vs GIAB gold standard)
  rather than resource footprint — use Truvari benchmarking instead.
- When profiling sex chromosomes, since VolcanoSV's phasing is designed only for autosomes.
- For CHM13-style homozygous samples, where VolcanoSV-asm is unsuitable for haplotype
  phasing.
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: VolcanoSV
  role: volcanosv-asm.py is the assembly pipeline being profiled; integrated assemblers
    are selected via -asm and data type via -dtype
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: default assembler for the HiFi run being timed and memory-profiled
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: default assembler for the CLR and ONT runs being timed and memory-profiled
  repo: https://github.com/fenderglass/Flye
- name: wtdbg2
  role: one of the alternative assemblers selectable via the -asm option of volcanosv-asm.py
  repo: https://github.com/ruanjue/wtdbg2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:ed6efce6c024c7b2c410aeef7d074a03aff9d70144a73c14fc0c2e08a10fa5d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Flye
  - VolcanoSV
  - hifiasm
  - wtdbg2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# computational-resource-profiling

## Summary

Profiles wall-clock runtime, thread count, peak memory, and CPU hours (walltime × ncpu) of long-read assembly pipelines (VolcanoSV-asm vs hifiasm vs Flye+HapDup) on HG002 long-read datasets so resource footprints can be tabulated and compared across sequencing technologies.

## When to use

Use when benchmarking assembly or SV-calling pipelines on the same datasets and you must report memory consumption, runtime, ncpu, and CPU-hours per run to compare against published resource tables.

## When NOT to use

- When the goal is SV-calling accuracy (recall/precision/F1 vs GIAB gold standard) rather than resource footprint — use Truvari benchmarking instead.
- When profiling sex chromosomes, since VolcanoSV's phasing is designed only for autosomes.
- For CHM13-style homozygous samples, where VolcanoSV-asm is unsuitable for haplotype phasing.

## Inputs

- HG002 PacBio HiFi BAM (~56x, Hifi_L1)
- HG002 PacBio CLR BAM (~89x, CLR_L1)
- HG002 Oxford Nanopore BAM (~48x, ONT_L1)
- Reference FASTA (e.g. hg19 genome.fa with .fai index)
- chrom_list text file listing one chromosome/contig name per line for WGS mode

## Outputs

- resource_usage.tsv with columns Data Type, Coverage, Memory, ncpu, Run time, CPU hours (walltime × ncpu)
- Final contig files per run (~expected assembly size) used as a sanity check that each run completed

## How to apply

Run each assembler on identical inputs with recorded timing and peak-memory capture: volcanosv-asm.py on the ~56x PacBio HiFi HG002 BAM (Hifi_L1) with -dtype Hifi and the default hifiasm assembler using -t 30 threads; on the ~89x CLR BAM (CLR_L1) with -dtype CLR and the default Flye assembler using -t 20; and on the ~48x ONT BAM (ONT_L1) with -dtype ONT, Flye default, -t 30. For each run, tabulate Data Type, Coverage, Memory, ncpu, Run time, and CPU hours (walltime × ncpu), writing the rows to resource_usage.tsv. Validation compares against published values (HiFi: 168 GB, 2-01:07:59, 1474 CPU hours; CLR: 259 GB, 5-09:05:59, 2582 CPU hours; ONT: 85 GB, 6-02:33:59, 4397 CPU hours); the expected qualitative result is that VolcanoSV-asm uses less memory and time than Flye+HapDup but more time (less memory) than hifiasm.

## Related tools

- **VolcanoSV** (volcanosv-asm.py is the assembly pipeline being profiled; integrated assemblers are selected via -asm and data type via -dtype) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (default assembler for the HiFi run being timed and memory-profiled) — https://github.com/chhylp123/hifiasm
- **Flye** (default assembler for the CLR and ONT runs being timed and memory-profiled) — https://github.com/fenderglass/Flye
- **wtdbg2** (one of the alternative assemblers selectable via the -asm option of volcanosv-asm.py) — https://github.com/ruanjue/wtdbg2

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam HG002_hifi.bam -ref genome.fa -o hifi_asm -dtype Hifi -asm hifiasm -t 30
```

## Evaluation signals

- Reproduced HiFi row matches 168 GB memory, 2-01:07:59 run time, and 1474 CPU hours.
- Reproduced CLR row matches 259 GB memory, 5-09:05:59 run time, and 2582 CPU hours.
- Reproduced ONT row matches 85 GB memory, 6-02:33:59 run time, and 4397 CPU hours.
- Each run produces a final contig file of roughly the expected assembly size (i.e. the pipeline completed).
- CPU hours column equals walltime × ncpu for each row, and the qualitative ordering holds: less memory/time than Flye+HapDup, more time/less memory than hifiasm.

## Limitations

- VolcanoSV is designed only for autosome chromosomes because the phasing procedure for sex chromosomes is very different.
- Resource measurements are sensitive to thread counts (e.g. -t 30 for HiFi/ONT vs -t 20 for CLR) and will differ across hardware; compare walltime × ncpu rather than raw walltime.
- VolcanoSV-asm requires more runtime than hifiasm even though it consumes less memory, so it may be impractical under tight walltime limits.

## Evidence

- [results] VolcanoSV-asm demonstrated lower memory consumption and runtime than Flye + HapDup, while requiring more runtime and less memory compared to hifiasm across the three representative long-read datasets.: [evidence span withheld — anchor status: not_found]
- [readme] The assembly pipeline is run per chromosome with selectable assemblers and thread counts.: "The VolcanoSV assembly pipeline is designed to run by chromosomes. **We integrated multiple state-of-the-art assemblers into the pipeline, including [hifiasm]"
- [readme] Sex chromosomes are out of scope for VolcanoSV phasing.: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
- [readme] WGS mode requires an explicit chromosome list consistent with the reference and BAM contig names.: "you must supply a `--chrom_list`/`-cl` (or `-chl`) text file listing one chromosome/contig name per line, exactly as they appear in your reference"

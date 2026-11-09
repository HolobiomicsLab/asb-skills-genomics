---
name: hifi-clr-ont-read-type-handling
description: Use when running the VolcanoSV-asm assembly component on a BAM from PacBio
  HiFi (~56x Hifi_L1), PacBio CLR (~89x CLR_L1), or ONT (~48x ONT_L1) HG002 reads
  and you must pick -dtype, assembler, and thread counts per technology.
when_to_use_negative:
- The input is short-read data or a non-long-read library — VolcanoSV is designed
  for single-molecule long-read platforms (HiFi, CLR, ONT) only
- The target is the CHM13 sample or another homozygous genome, since the phasing-based
  assembly component is unsuitable for haplotype phasing of homozygous samples
- Only sex chromosomes are of interest — VolcanoSV is designed only for autosome chromosomes
  because the phasing procedure for sex chromosomes differs
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: VolcanoSV
  role: Pipeline whose volcanosv-asm.py component is invoked with -dtype Hifi/CLR/ONT
    to select read-type-specific assembly and filtering behavior
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: Default assembler used by volcanosv-asm.py when -dtype is Hifi
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: Default assembler used by volcanosv-asm.py when -dtype is CLR or ONT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:e1890ae724d87cc29ed2791d308680c8a82a66f60a7bc79dcf963d21cd9481b7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Flye
  - VolcanoSV
  - hifiasm
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hifi-clr-ont-read-type-handling

## Summary

Configure VolcanoSV-asm correctly for the three mainstream long-read technologies (PacBio HiFi, PacBio CLR, Oxford Nanopore) by setting the -dtype argument to Hifi, CLR, or ONT, which selects the platform-appropriate assembler (hifiasm for HiFi, Flye for CLR/ONT) and downstream read-type-specific filtering parameters.

## When to use

Use when running the VolcanoSV-asm assembly component on a BAM from PacBio HiFi (~56x Hifi_L1), PacBio CLR (~89x CLR_L1), or ONT (~48x ONT_L1) HG002 reads and you must pick -dtype, assembler, and thread counts per technology.

## When NOT to use

- The input is short-read data or a non-long-read library — VolcanoSV is designed for single-molecule long-read platforms (HiFi, CLR, ONT) only
- The target is the CHM13 sample or another homozygous genome, since the phasing-based assembly component is unsuitable for haplotype phasing of homozygous samples
- Only sex chromosomes are of interest — VolcanoSV is designed only for autosome chromosomes because the phasing procedure for sex chromosomes differs

## Inputs

- WGS or chromosome-level long-read BAM aligned to hg19 (e.g., Hifi_L1 ~56x PacBio HiFi, CLR_L1 ~89x PacBio CLR, ONT_L1 ~48x Nanopore HG002 BAMs)
- Reference FASTA plus index (genome.fa and genome.fa.fai from Zenodo record 10520476)
- Chromosome list text file (--chrom_list/-cl) with one contig name per line, exactly as in the reference

## Outputs

- Per-dataset phased diploid assembly contig files (final contig FASTA of expected assembly size)
- Resource usage table (resource_usage.tsv) with columns Data Type, Coverage, Memory, ncpu, Run time, CPU hours
- Downstream SV/SNP/indel calls produced with technology-specific filtering boundaries

## How to apply

Run volcanosv-asm.py once per dataset, passing -dtype Hifi with the default hifiasm assembler and -t 30 threads for HiFi BAMs, and -dtype CLR or -dtype ONT with the default Flye assembler (-t 20 for CLR, -t 30 for ONT) for CLR/ONT BAMs. The dtype setting matters beyond assembly: supporting-signature-depth filters for large indel SVs use technology-specific empirical boundaries (HiFi: lb=0.2, rb=2.6; CLR: lb=0.19, rb=3.0; ONT: lb=0.24, rb=2.8), so an incorrect dtype invalidates downstream SV filtering as well. Reproduce the resource-usage benchmark by timing each run (walltime x ncpu = CPU hours) and recording peak memory (HiFi: 168GB; CLR: 259GB; ONT: 85GB). VolcanoSV uses phased SNPs and unique k-mer similarity analysis for haplotype-resolved assembly, which is why its phasing and local assembly modules give lower memory and runtime than Flye+HapDup for CLR/ONT data.

## Related tools

- **VolcanoSV** (Pipeline whose volcanosv-asm.py component is invoked with -dtype Hifi/CLR/ONT to select read-type-specific assembly and filtering behavior) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (Default assembler used by volcanosv-asm.py when -dtype is Hifi) — https://github.com/chhylp123/hifiasm
- **Flye** (Default assembler used by volcanosv-asm.py when -dtype is CLR or ONT)

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam ONT_L1.bam -o out_ont -ref genome.fa -dtype ONT -asm flye -t 30
```

## Evaluation signals

- The reproduced resource-usage table reports memory and run time consistent with published values: HiFi 168GB / 2-01:07:59 (1474 CPU hours), CLR 259GB / 5-09:05:59 (2582 CPU hours), ONT 85GB / 6-02:33:59 (4397 CPU hours)
- Each run produces a final contig file of roughly the expected assembly size
- The -dtype value matches the actual platform of the input BAM (Hifi for PacBio CCS, CLR for PacBio CLR, ONT for Nanopore)
- Installed environment is the volcanosv conda environment (Python 3.8.3) created from requirement.yaml, activated before any run
- Large indel SV filtering uses the correct technology-specific lb/rb boundaries (HiFi 0.2/2.6, CLR 0.19/3.0, ONT 0.24/2.8)

## Limitations

- Resource usage scales very differently per platform: ONT_L1 (48x, the lowest coverage) consumed the most CPU hours (4397), so thread/memory budgets must be technology-aware
- Detecting small indels based on assembly was ineffective for ONT data (low precision for all tools evaluated)
- VolcanoSV-asm requires more runtime than hifiasm on HiFi data, though with less memory
- The pipeline only handles autosome chromosomes; sex chromosome phasing is not supported

## Evidence

- [readme] If you wish to reproduce the result for CLR data or ONT data, you can just simply change the input BAM file and the argument "dtype" to the corresponding data type (CLR/ONT): "If you wish to reproduce the result for CLR data or ONT data, you can just simply change the input BAM file and the argument "dtype" to the corresponding"
- [methods] Run volcanosv-asm.py on the HiFi BAM with -dtype Hifi, the default hifiasm assembler, and -t 30 threads: [evidence span withheld — anchor status: not_found]
- [results] VolcanoSV-asm demonstrated lower memory consumption and runtime than Flye + HapDup, while requiring more runtime and less memory compared to hifiasm: [evidence span withheld — anchor status: not_found]
- [discussion] for Hifi data, lb = 0.2 and rb = 2.6; For CLR data, lb = 0.19 and rb = 3.0; for ONT data, lb = 0.24 and rb = 2.8: "for Hifi data, lb = 0.2 and rb = 2.6; For CLR data, lb = 0.19 and rb = 3.0; for ONT data, lb = 0.24 and rb = 2.8"

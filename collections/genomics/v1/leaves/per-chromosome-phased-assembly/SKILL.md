---
name: per-chromosome-phased-assembly
description: Use when you have a long-read WGS BAM (PacBio HiFi/CLR or ONT) from a
  diploid sample and need a phased, per-chromosome diploid assembly as input for SV
  calling or resource benchmarking.
when_to_use_negative:
- 'The sample is haploid or homozygous (e.g. CHM13): phasing is unsuitable, as the
  paper did not use VolcanoSV-asm for the CHM13 diploid assembly'
- 'You need sex chromosome assemblies: VolcanoSV is designed only for autosome chromosomes
  since sex-chromosome phasing differs'
- Input is short-read data or unaligned FASTQ without a reference — the pipeline is
  a hybrid method requiring read alignments to a reference
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Provides volcanosv-asm.py, the per-chromosome phased diploid assembly pipeline
    with phasing and local assembly modules
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: Default assembler for Hifi dtype input within VolcanoSV-asm
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: Default assembler for CLR and ONT dtype input within VolcanoSV-asm
  repo: https://github.com/fenderglass/Flye
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:33f05919bd9e9314a751e8b7e976c8a72f868ff9b60664e8483ee120347b03c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Flye
  - VolcanoSV
  - hifiasm
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# per-chromosome-phased-assembly

## Summary

Runs VolcanoSV-asm, which assembles each autosome independently with a platform-appropriate assembler (hifiasm for HiFi, Flye for CLR/ONT) and phasing/local-assembly modules to produce a haplotype-resolved diploid contig set from long-read BAMs.

## When to use

Use when you have a long-read WGS BAM (PacBio HiFi/CLR or ONT) from a diploid sample and need a phased, per-chromosome diploid assembly as input for SV calling or resource benchmarking.

## When NOT to use

- The sample is haploid or homozygous (e.g. CHM13): phasing is unsuitable, as the paper did not use VolcanoSV-asm for the CHM13 diploid assembly
- You need sex chromosome assemblies: VolcanoSV is designed only for autosome chromosomes since sex-chromosome phasing differs
- Input is short-read data or unaligned FASTQ without a reference — the pipeline is a hybrid method requiring read alignments to a reference

## Inputs

- Long-read alignment BAM (WGS or single-chromosome) — Hifi (~56x), CLR (~89x) or ONT (~48x) HG002 data
- Reference genome FASTA with .fai index (e.g. hg19 genome.fa)
- Chromosome list text file (-cl) with one contig name per line exactly as in the reference/BAM

## Outputs

- Per-chromosome phased diploid contigs (final contig FASTA of roughly the expected assembly size)
- Resource usage table (resource_usage.tsv: Data Type, Coverage, Memory, ncpu, Run time, CPU hours)
- Assembly intermediates usable downstream by VolcanoSV-vc for SV calling

## How to apply

First install VolcanoSV from GitHub (conda env 'volcanosv' from requirement.yaml) and validate on the Zenodo chr10 test data before WGS runs. Build a --chrom_list text file (one autosome name per line, matching the reference and BAM naming) because VolcanoSV processes autosomes chromosome-by-chromosome and does not handle sex chromosomes. For each chromosome, invoke volcanosv-asm.py with the BAM, reference FASTA, chromosome name, thread count, dtype (Hifi/CLR/ONT) and assembler: hifiasm is the default for Hifi data, Flye for CLR/ONT (the paper's benchmark used -t 30 for Hifi/ONT and -t 20 for CLR). For each run, record wall time, ncpu, peak memory and compute CPU hours (walltime x ncpu); the published HG002 resource table is HiFi: 168GB / 2-01:07:59 / 1474 CPU hours, CLR: 259GB / 5-09:05:59 / 2582 CPU hours, ONT: 85GB / 6-02:33:59 / 4397 CPU hours. The phased SNP + unique k-mer similarity modules are what resolve the two haplotypes, so phasing quality depends on heterozygous SNP discovery within each chromosome window.

## Related tools

- **VolcanoSV** (Provides volcanosv-asm.py, the per-chromosome phased diploid assembly pipeline with phasing and local assembly modules) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (Default assembler for Hifi dtype input within VolcanoSV-asm) — https://github.com/chhylp123/hifiasm
- **Flye** (Default assembler for CLR and ONT dtype input within VolcanoSV-asm) — https://github.com/fenderglass/Flye

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam HG002_Hifi_L1.bam -ref genome.fa -o asm_out -t 30 -dtype Hifi -asm hifiasm -cl chrom_list.txt
```

## Evaluation signals

- Each chromosome run produces a final contig file of roughly the expected assembly size
- Reproduced resource_usage.tsv matches published values: HiFi 168GB / 1474 CPU hours, CLR 259GB / 2582 CPU hours, ONT 85GB / 4397 CPU hours
- CPU hours equal wall time x ncpu in the tabulated output
- Runs with Hifi/CLR/ONT data complete only when -dtype and assembler match (hifiasm for Hifi, Flye for CLR/ONT)
- Validate installation on the provided Zenodo chr10 test data before WGS runs

## Limitations

- VolcanoSV-asm is designed only for autosome chromosomes; sex chromosome phasing is not supported
- Owing to its phasing and local assembly modules, VolcanoSV-asm needs more runtime than hifiasm (though less than Flye+HapDup) on HiFi/CLR/ONT data
- Translocation detection requires a WGS BAM and cannot be run at the single-chromosome level
- A reference genome is required — this is a hybrid, not purely de novo, assembly approach

## Evidence

- [readme] The VolcanoSV assembly pipeline is designed to run by chromosomes.: "The VolcanoSV assembly pipeline is designed to run by chromosomes."
- [readme] We integrated multiple state-of-the-art assemblers into the pipeline: "We integrated multiple state-of-the-art assemblers into the pipeline, including [hifiasm](https://github.com/chhylp123/hifiasm),[Flye](https://github.com/fenderglass/Flye),[wtdbg2](https://github.com/"
- [results] VolcanoSV-asm demonstrated lower memory consumption and runtime than Flye + HapDup: [evidence span withheld — anchor status: not_found]
- [abstract] utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly: "utilizes both a reference genome and local de novo assembly to generate a phased diploid assembly"
- [readme] VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!: "**1.VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!**"

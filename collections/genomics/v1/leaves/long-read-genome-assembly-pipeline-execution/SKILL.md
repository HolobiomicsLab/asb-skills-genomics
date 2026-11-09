---
name: long-read-genome-assembly-pipeline-execution
description: Use when you have HG002-style long-read BAM inputs (PacBio HiFi/CLR or
  ONT) and need to run a phased diploid assembly pipeline and quantify its computational
  resource usage (memory, walltime, CPU hours) versus hifiasm or Flye+HapDup.
when_to_use_negative:
- The sample is homozygous (e.g. CHM13 from a complete hydatidiform mole) — the phasing
  module is unsuitable, as the authors excluded VolcanoSV-asm for CHM13 diploid assembly
- Targeting sex chromosomes — VolcanoSV is designed only for autosome chromosomes
  since the phasing procedure for sex chromosomes is very different
- You only need SV calls from an existing assembly — use VolcanoSV-vc on a pre-built
  assembly instead of re-running the assembly pipeline
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: VolcanoSV
  role: Main pipeline executed via volcanosv-asm.py; performs phasing and local assembly
    with selectable assemblers and records resource usage
  repo: https://github.com/maiziezhoulab/VolcanoSV
- name: hifiasm
  role: Default assembler for the Hifi (-dtype Hifi) VolcanoSV-asm run, used for the
    56x HiFi HG002 dataset
  repo: https://github.com/chhylp123/hifiasm
- name: Flye
  role: Default assembler for CLR (-dtype CLR) and ONT (-dtype ONT) VolcanoSV-asm
    runs on the 89x CLR and 48x ONT HG002 datasets
  repo: https://github.com/fenderglass/Flye
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51282-0
    title: VolcanoSV enables accurate and robust structural variant calling in diploid
      genomes from single-molecule long read sequencing
  source_content_hash: sha256:b3c81101c842077107376aa20e16f24dd9764a551f7ee3105ff9302617799937
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/maiziezhoulab/VolcanoSV
  tools:
  - Flye
  - VolcanoSV
  - hifiasm
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# long-read-genome-assembly-pipeline-execution

## Summary

Execute chromosome-wise long-read diploid assembly with VolcanoSV-asm (volcanosv-asm.py), selecting the appropriate integrated assembler (hifiasm for HiFi, Flye for CLR/ONT) per data type, and record runtime and peak memory to benchmark against alternative assembly pipelines.

## When to use

Use when you have HG002-style long-read BAM inputs (PacBio HiFi/CLR or ONT) and need to run a phased diploid assembly pipeline and quantify its computational resource usage (memory, walltime, CPU hours) versus hifiasm or Flye+HapDup.

## When NOT to use

- The sample is homozygous (e.g. CHM13 from a complete hydatidiform mole) — the phasing module is unsuitable, as the authors excluded VolcanoSV-asm for CHM13 diploid assembly
- Targeting sex chromosomes — VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different
- You only need SV calls from an existing assembly — use VolcanoSV-vc on a pre-built assembly instead of re-running the assembly pipeline

## Inputs

- HG002 long-read BAM file (WGS or single-chromosome), e.g. Hifi_L1 (~56x PacBio HiFi), CLR_L1 (~89x PacBio CLR), ONT_L1 (~48x Nanopore)
- Reference genome FASTA plus .fai index (e.g. hg19 genome.fa)
- Chromosome list text file (--chrom_list/-cl), one chromosome name per line matching the reference and BAM
- Selected assembler setting: hifiasm for HiFi, Flye for CLR/ONT

## Outputs

- Final phased diploid contig file per data type (expected roughly genome-scale assembly size)
- resource_usage.tsv table with columns: Data Type, Coverage, Memory, ncpu, Run time, CPU hours (walltime x ncpu)

## How to apply

Install VolcanoSV from GitHub with `git clone --recurse-submodules` and `sh Install.sh`, activating the `volcanosv` conda environment (Python 3.8.3). Run the main script `${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py` chromosome-wise: for Hifi_L1 (~56x PacBio HiFi) use `-dtype Hifi` with the default hifiasm assembler and -t 30; for CLR_L1 (~89x PacBio CLR) use `-dtype CLR` with the default Flye assembler and -t 20; for ONT_L1 (~48x Nanopore) use `-dtype ONT` with Flye default and -t 30, always supplying the reference FASTA and a `-cl` chromosome list file. Time each run and capture peak memory, then tabulate Data Type, Coverage, Memory, ncpu, Run time, and CPU hours (walltime x ncpu) into resource_usage.tsv. VolcanoSV-asm's phasing and local assembly modules are expected to consume less memory than Flye+HapDup while using more time and less memory than hifiasm. Validate that each run produces a final contig file of roughly the expected assembly size.

## Related tools

- **VolcanoSV** (Main pipeline executed via volcanosv-asm.py; performs phasing and local assembly with selectable assemblers and records resource usage) — https://github.com/maiziezhoulab/VolcanoSV
- **hifiasm** (Default assembler for the Hifi (-dtype Hifi) VolcanoSV-asm run, used for the 56x HiFi HG002 dataset) — https://github.com/chhylp123/hifiasm
- **Flye** (Default assembler for CLR (-dtype CLR) and ONT (-dtype ONT) VolcanoSV-asm runs on the 89x CLR and 48x ONT HG002 datasets) — https://github.com/fenderglass/Flye

## Examples

```
python ${path_to_volcanosv}/bin/VolcanoSV-asm/volcanosv-asm.py -bam Hifi_L1.bam -ref genome.fa -o hifi_out -dtype Hifi -asm hifiasm -t 30 -cl chrom_list.txt
```

## Evaluation signals

- Reproduced resource table matches published values: HiFi 168GB / 2-01:07:59 / 1474 CPU hours; CLR 259GB / 5-09:05:59 / 2582 CPU hours; ONT 85GB / 6-02:33:59 / 4397 CPU hours
- Each run produces a final contig file of roughly the expected assembly size
- CPU hours equals walltime multiplied by ncpu in resource_usage.tsv
- VolcanoSV-asm shows lower memory than Flye+HapDup but higher runtime and lower memory than hifiasm across all three datasets
- Correct dtype-assembler pairing confirmed: hifiasm for Hifi, Flye for CLR and ONT

## Limitations

- VolcanoSV is designed only for autosome chromosomes; the phasing procedure for sex chromosomes is very different
- Resource usage is large — e.g. ONT runs needed ~85GB memory and 4397 CPU hours (6 days at 30 threads); plan compute accordingly
- VolcanoSV-asm requires more runtime than hifiasm, so it may not suit time-constrained comparisons despite lower memory

## Evidence

- [results] Resource comparison across assemblers: [evidence span withheld — anchor status: not_found]
- [readme] Chromosome-wise design with integrated assemblers: "The VolcanoSV assembly pipeline is designed to run by chromosomes. **We integrated multiple state-of-the-art assemblers into the pipeline, including [hifiasm](https://github.com/chhylp123/hifiasm),[Fl"
- [readme] Chromosome list requirement: "you must supply a `--chrom_list`/`-cl` (or `-chl`) text file listing one chromosome/contig name per line, exactly as they appear in your reference"
- [readme] Autosome-only scope: "VolcanoSV is designed only for autosome chromosomes since the phasing procedure for sex chromosomes is very different!"
- [results] CHM13 exclusion rationale: "We did not use the VolcanoSV-asm component for the diploid assembly since the CHM13 sample derives from a single homozygous complete hydatidiform mole"

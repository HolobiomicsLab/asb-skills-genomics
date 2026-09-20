---
name: haplotype-aware-methylation-phasing
description: Use when the input is PacBio HiFi (CCS) BAM/subread files plus a reference
  genome and the goal is haplotype-specific CpG methylation (bedmethyl per haplotype)
  or allele-specific methylation / DMR detection.
when_to_use_negative:
- Detection of non-CpG 5mC, 6mA, or other DNA modifications is needed — ccsmeth/ccsmethphase
  only target 5mCpG.
- Hemimethylated CpG detection is required — the methods are incapable of detecting
  hemimethylated CpGs.
- Input is short-read, non-PacBio data (e.g. nanopore or Illumina) — the kinetics-based
  ccsmeth models require PacBio CCS reads.
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3295
tools:
- name: ccsmethphase
  role: Nextflow pipeline orchestrating methylation phasing and ASM detection from
    CCS reads
  repo: https://github.com/PengNi/ccsmethphase
- name: ccsmeth
  role: Deep-learning detection of per-read 5mCpG calls producing modBAM files from
    HiFi reads
  repo: https://github.com/PengNi/ccsmeth
- name: pbmm2
  role: Aligns ccsmeth modBAMs to the reference genome
  repo: https://github.com/PacificBiosciences/pbmm2
- name: clair3
  role: Calls SNVs from aligned modBAMs used for phasing
- name: whatshap
  role: Phases PASS SNVs into haplotype-resolved VCFs and BAMs
- name: Nextflow
  role: Workflow engine running the ccsmethphase pipeline
- name: DSS
  role: Differential methylation analysis to call DMLs and DMRs between haplotypes
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:6b9fd1a77562aa5ee392c79ab8d174790b592e79c193f7745be41cca65c58657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmethphase
  tools:
  - DSS
  - Nextflow
  - Singularity
  - ccsmeth
  - ccsmethphase
  - clair3
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# haplotype-aware-methylation-phasing

## Summary

Run the ccsmethphase Nextflow pipeline to call haplotype-resolved 5mCpG methylation and allele-specific methylation (ASM) directly from PacBio HiFi/CCS reads, without BS-seq or nanopore data, by combining ccsmeth modBAM calls with clair3 SNV calling and whatshap phasing.

## When to use

Use when the input is PacBio HiFi (CCS) BAM/subread files plus a reference genome and the goal is haplotype-specific CpG methylation (bedmethyl per haplotype) or allele-specific methylation / DMR detection.

## When NOT to use

- Detection of non-CpG 5mC, 6mA, or other DNA modifications is needed — ccsmeth/ccsmethphase only target 5mCpG.
- Hemimethylated CpG detection is required — the methods are incapable of detecting hemimethylated CpGs.
- Input is short-read, non-PacBio data (e.g. nanopore or Illumina) — the kinetics-based ccsmeth models require PacBio CCS reads.

## Inputs

- PacBio HiFi/CCS read BAM files (hifi.bam or subreads.bam) for one or more flowcells
- Human (or other) reference genome FASTA
- Input sheet TSV with columns Group_ID, Sample_ID, Type, Path
- Optional: BS-seq bed file (e.g. hg002_bsseq_chr20_demo.bed) and/or nanopore methylation calls for benchmarking

## Outputs

- Phased modBAM files (SNV_PASS_whatshap BAMs)
- Haplotype-specific bedmethyl files for hp1 and hp2 under ccsmethphase_results/mods_freq
- clair3 SNV VCFs and whatshap phased VCFs
- DML/DMR tables from the DSS-based differential methylation analysis

## How to apply

Build an input sheet TSV (columns Group_ID, Sample_ID, Type, Path) listing all HG002 hifi flowcell BAMs and supply the human reference FASTA, then launch 'nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity' (the singularity profile caches ccsmeth and clair3 images locally and GPU is enabled via CUDA_VISIBLE_DEVICES). ccsmeth converts aligned HiFi reads into modified-base BAM (modBAM) files with per-read 5mCpG calls; the modBAMs are aligned with pbmm2, SNVs are called with clair3, and PASS SNVs are phased with whatshap into haplotype-resolved VCFs and BAMs (filter step: SNV_PASS; --include_all_ctgs true/false controls whether only chr1-22,X,Y are analyzed). Per-haplotype site-level methylation frequencies are aggregated into bedmethyl files (hp1 and hp2) under the mods_freq directory, and a DSS-based differential methylation analysis calls CCS DMLs and DMRs, which are compared against HG002 BS-seq bed files and nanopore calls using Pearson correlation.

## Related tools

- **ccsmethphase** (Nextflow pipeline orchestrating methylation phasing and ASM detection from CCS reads) — https://github.com/PengNi/ccsmethphase
- **ccsmeth** (Deep-learning detection of per-read 5mCpG calls producing modBAM files from HiFi reads) — https://github.com/PengNi/ccsmeth
- **pbmm2** (Aligns ccsmeth modBAMs to the reference genome) — https://github.com/PacificBiosciences/pbmm2
- **clair3** (Calls SNVs from aligned modBAMs used for phasing)
- **whatshap** (Phases PASS SNVs into haplotype-resolved VCFs and BAMs)
- **Nextflow** (Workflow engine running the ccsmethphase pipeline)
- **DSS** (Differential methylation analysis to call DMLs and DMRs between haplotypes)

## Examples

```
nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity
```

## Evaluation signals

- All expected outputs exist under ccsmethphase_results: phased modBAM, hp1/hp2 bedmethyl beds, clair3 and whatshap VCFs, and DML/DMR tables.
- Haplotype-level methylation correlations against BS-seq and nanopore benchmarks exceed the reported ~0.93 Pearson threshold.
- 81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs (reported HG002 comparison: 14,390 CCS DMRs vs 2,463 BS-seq and 16,250 nanopore).
- Site-level methylation frequencies correlate >0.90 with bisulfite sequencing and nanopore sequencing even using only 10x reads.
- Phased BAM names reflect the expected chain: merged_size1.SNV_PASS_whatshap, confirming only PASS SNVs were phased.

## Limitations

- Methods cannot detect hemimethylated CpGs.
- Cannot be applied to 5mC detection in non-CpG contexts or other DNA modifications such as 6mA.
- Whether the site-level model design applies to non-CpG 5mCs and other modifications has not been verified.
- The strand-specific-methylation model has significantly reduced performance due to limited subread depth.
- Recommended hardware is substantial: 128 GB RAM, 40 CPU processors, 4 TB disk storage, >=8 GB GPU.

## Evidence

- [abstract] ccsmethphase detects haplotype-aware methylation: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [methods] pipeline takes reads and reference as input: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [methods] output directory layout: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [abstract] benchmark correlation threshold: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [methods] include_all_ctgs parameter semantics: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [readme] README hardware requirements: "Recommended Hardware requirements: 128 GB RAM, 40 CPU processors, 4 TB disk storage, >=8 GB GPU"

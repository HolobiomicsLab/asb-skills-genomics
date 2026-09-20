---
name: nextflow-pipeline-execution
description: Use when you have PacBio HiFi BAMs and a reference FASTA and need haplotype-resolved
  methylation (hp1/hp2 bedmethyl) and allele-specific methylation DMRs without nanopore
  or bisulfite data.
when_to_use_negative:
- The input reads are nanopore or bisulfite sequencing data rather than PacBio CCS/HiFi
  reads — ccsmeth models are trained on CCS kinetics (IPDs and PWs) and are inapplicable.
- The goal is detection of hemimethylated CpGs, non-CpG 5mC, or other modifications
  such as 6mA — the article states these methods cannot detect them.
- A non-Nextflow, single-tool workflow is preferred and only read-level 5mCpG calls
  are needed — run ccsmeth directly instead of the full phasing pipeline.
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: ccsmethphase
  role: The Nextflow pipeline being executed; orchestrates all four stages and produces
    the ccsmethphase_results output tree
  repo: github.com/PengNi/ccsmethphase
- name: Nextflow
  role: Workflow engine used to run ccsmethphase (version >=21.10.6)
  repo: https://www.nextflow.io/
- name: ccsmeth
  role: Deep-learning step that converts HiFi BAMs into modBAM files with per-read
    5mCpG calls
  repo: github.com/PengNi/ccsmeth
- name: pbmm2
  role: Aligns and merges the ccsmeth modBAM files
  repo: github.com/PacificBiosciences/pbmm2
- name: Clair3
  role: Calls SNVs from the aligned modBAM using its hifi model; only PASS SNVs are
    retained
- name: WhatsHap
  role: Phases the PASS SNVs and then the reads to yield haplotype-tagged modBAMs
    and phased VCFs
- name: DSS
  role: Differential methylation analysis (referenced from NanoMethPhase) to call
    DMLs and DMRs
  repo: github.com/vahidAK/NanoMethPhase
- name: Singularity
  role: Container profile (-profile singularity) that caches ccsmeth and clair3 images
    to a local singularity_cache directory
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:f7045e91a5cbbefc2982fd227975c8acd9af6379ac271591ed6dbc0326330b02
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - DSS
  - Nextflow
  - Singularity
  - WhatsHap
  - ccsmeth
  - ccsmethphase
  - clair3
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# nextflow-pipeline-execution

## Summary

Execute the ccsmethphase Nextflow pipeline that chains ccsmeth 5mCpG calling, pbmm2 alignment, Clair3 SNV calling, WhatsHap phasing, and DSS-based allele-specific methylation detection from PacBio CCS (HiFi) reads only.

## When to use

Use when you have PacBio HiFi BAMs and a reference FASTA and need haplotype-resolved methylation (hp1/hp2 bedmethyl) and allele-specific methylation DMRs without nanopore or bisulfite data.

## When NOT to use

- The input reads are nanopore or bisulfite sequencing data rather than PacBio CCS/HiFi reads — ccsmeth models are trained on CCS kinetics (IPDs and PWs) and are inapplicable.
- The goal is detection of hemimethylated CpGs, non-CpG 5mC, or other modifications such as 6mA — the article states these methods cannot detect them.
- A non-Nextflow, single-tool workflow is preferred and only read-level 5mCpG calls are needed — run ccsmeth directly instead of the full phasing pipeline.

## Inputs

- PacBio HiFi BAM files (subreads.bam or hifi.bam), one row per flowcell in an input_sheet.tsv with Group_ID, Sample_ID, Type, Path columns
- Human reference genome FASTA (e.g. chr20_demo.fa for the demo)
- Optional HG002 BS-seq bed file for validation comparisons

## Outputs

- Haplotype-tagged modBAM (e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam)
- Clair3-called SNV VCFs (clair3_called) and WhatsHap-phased VCFs (whatshap_phased)
- Haplotype-specific bedmethyl methylation frequency files (hp1.bed, hp2.bed, all.bed) under mods_freq
- DML/DMR tables (e.g. ...hp_callDMR.autosomes_cf0.2.bed) under diff_methyl

## How to apply

Build an input sheet TSV with Group_ID, Sample_ID, Type='hifi', and Path columns listing each flowcell BAM, and supply the reference genome FASTA. Launch the workflow with 'nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity'; the singularity profile caches ccsmeth and clair3 images locally (a singularity_cache directory) and GPU access is controlled via CUDA_VISIBLE_DEVICES. The pipeline runs four stages: ccsmeth converts aligned HiFi reads into modBAMs with per-read 5mCpG calls, pbmm2 aligns/merges the modBAMs, Clair3 calls SNVs with its hifi model retaining only PASS SNVs (SNV_PASS filter), and WhatsHap phases SNVs then reads into haplotype-tagged modBAMs and phased VCFs. Haplotype-specific site-level methylation frequencies are aggregated into bedmethyl files (hp1.bed, hp2.bed, all.bed) under mods_freq, then DSS-based differential methylation analysis (referenced from NanoMethPhase) calls DMLs and DMRs at p<0.001 with |diff|>0.2, filtering to autosomes with cf threshold 0.2. The --include_all_ctgs flag controls contig scope: true includes all contigs, the default false restricts to [chr][1-22,X,Y].

## Related tools

- **ccsmethphase** (The Nextflow pipeline being executed; orchestrates all four stages and produces the ccsmethphase_results output tree) — github.com/PengNi/ccsmethphase
- **Nextflow** (Workflow engine used to run ccsmethphase (version >=21.10.6)) — https://www.nextflow.io/
- **ccsmeth** (Deep-learning step that converts HiFi BAMs into modBAM files with per-read 5mCpG calls) — github.com/PengNi/ccsmeth
- **pbmm2** (Aligns and merges the ccsmeth modBAM files) — github.com/PacificBiosciences/pbmm2
- **Clair3** (Calls SNVs from the aligned modBAM using its hifi model; only PASS SNVs are retained)
- **WhatsHap** (Phases the PASS SNVs and then the reads to yield haplotype-tagged modBAMs and phased VCFs)
- **DSS** (Differential methylation analysis (referenced from NanoMethPhase) to call DMLs and DMRs) — github.com/vahidAK/NanoMethPhase
- **Singularity** (Container profile (-profile singularity) that caches ccsmeth and clair3 images to a local singularity_cache directory)

## Examples

```
nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity
```

## Evaluation signals

- All expected outputs exist under ccsmethphase_results/<dsname>: phased modBAM, hp1/hp2 bedmethyl beds, clair3_called and whatshap_phased VCFs, and DML/DMR tables
- Output file names follow the documented pattern, e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.freq.aggregate.hp_callDMR.autosomes_cf0.2.bed
- Haplotype-level methylation correlations against HG002 BS-seq and nanopore benchmarks exceed the reported ~0.93 threshold
- A large majority of BS-seq DMRs fall within 10 kb of CCS-generated DMRs (81.4% on HG002 in the reported run)
- DMRs are called with the documented DSS thresholds: p<0.001, |diff|>0.2, autosomes only, cf 0.2

## Limitations

- Recommended hardware is heavy: 128 GB RAM, 40 CPU processors, 4 TB disk storage, and >=8 GB GPU
- All current methods, including ccsmeth/ccsmethphase, only consider symmetric methylation and cannot detect hemimethylated CpGs or non-CpG 5mC / 6mA
- DMRs detected by PacBio CCS for assisting haplotyping and genome assembly remain to be studied

## Evidence

- [abstract] we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [methods] Pipeline launch command: [evidence span withheld — anchor status: not_found]
- [methods] include_all_ctgs semantics: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [methods] Output naming pattern: "G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam"
- [readme] Recommended hardware requirements: "Recommended Hardware requirements: 128 GB RAM, 40 CPU processors, 4 TB disk storage, >=8 GB GPU"

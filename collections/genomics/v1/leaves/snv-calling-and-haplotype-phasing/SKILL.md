---
name: snv-calling-and-haplotype-phasing
description: 'Use when haplotype-resolved methylation or allele-specific methylation
  must be derived from PacBio HiFi/CCS modBAMs: per-read 5mCpG calls exist but reads
  are not yet haplotagged and no phased SNV VCF is available.'
when_to_use_negative:
- Input reads are nanopore or Illumina rather than PacBio CCS/HiFi — Clair3 must run
  with the hifi model and the pipeline is designed for CCS kinetics-derived modBAMs.
- The modBAMs already carry haplotype tags and a phased SNV VCF exists — re-running
  calling/phasing would be redundant.
- The goal is read-level 5mC detection without haplotype resolution — ccsmeth alone
  suffices, no SNV calling/phasing needed.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0654
tools:
- name: clair3
  role: Calls SNVs from the aligned modBAMs using its hifi model; output filtered
    to PASS SNVs
- name: whatshap
  role: Phases the PASS SNVs into haplotype-resolved VCFs and phases the reads to
    yield haplotype-tagged modBAMs
- name: pbmm2
  role: Aligns and merges ccsmeth modBAM files prior to SNV calling (merged_size1
    step)
  repo: https://github.com/PacificBiosciences/pbmm2
- name: ccsmethphase
  role: Nextflow pipeline orchestrating the SNV calling and phasing stages of methylation
    phasing
  repo: https://github.com/PengNi/ccsmethphase
- name: Nextflow
  role: Workflow engine running the ccsmethphase pipeline stages
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:9419f284a6dcbdcfe9d8c8ec7235bb476715dea5b2c614dfec8a18e400bfc85a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PacificBiosciences/pbmm2
  tools:
  - DSS
  - Nextflow
  - Singularity
  - ccsmethphase
  - clair3
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# snv-calling-and-haplotype-phasing

## Summary

Calls SNVs from ccsmeth-generated modified-base BAMs with Clair3 (hifi model) and phases PASS SNVs and reads with WhatsHap to produce haplotype-resolved VCFs and haplotagged modBAMs, enabling haplotype-aware methylation detection from PacBio CCS reads.

## When to use

Use when haplotype-resolved methylation or allele-specific methylation must be derived from PacBio HiFi/CCS modBAMs: per-read 5mCpG calls exist but reads are not yet haplotagged and no phased SNV VCF is available.

## When NOT to use

- Input reads are nanopore or Illumina rather than PacBio CCS/HiFi — Clair3 must run with the hifi model and the pipeline is designed for CCS kinetics-derived modBAMs.
- The modBAMs already carry haplotype tags and a phased SNV VCF exists — re-running calling/phasing would be redundant.
- The goal is read-level 5mC detection without haplotype resolution — ccsmeth alone suffices, no SNV calling/phasing needed.

## Inputs

- ccsmeth modified-base BAM (modBAM) files with per-read 5mCpG calls
- pbmm2-aligned/merged modBAM (merged_size1)
- Human reference genome FASTA (e.g. chr20_demo.fa or T2T-CHM13.fa)
- input_sheet.tsv with Group_ID, Sample_ID, Type=hifi, Path columns
- Clair3 hifi model

## Outputs

- clair3_called SNV VCFs
- whatshap_phased VCFs (PASS SNVs phased)
- Haplotype-tagged modBAMs (e.g. *.SNV_PASS_whatshap.bam)
- Haplotype-specific bedmethyl methylation frequency files (hp1.bed, hp2.bed, all.bed) under mods_freq
- Downstream DML/DMR tables from DSS (autosomes_cf0.2)

## How to apply

Within the ccsmethphase Nextflow pipeline, first align/merge the ccsmeth modBAM files with pbmm2 (merged_size1 step). Call SNVs from the aligned modBAMs with Clair3 using its hifi model, then retain only PASS SNVs via the SNV_PASS filter — the naming pattern G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam reflects this chain. Phase the PASS SNVs with WhatsHap to produce a phased VCF, then phase the reads so modBAMs carry HP tags. The haplotagged modBAMs feed downstream aggregation of haplotype-specific site-level methylation frequencies into bedmethyl files (hp1.bed, hp2.bed, all.bed) under mods_freq, which is what enables DSS-based DML/DMR detection at p<0.001 with |diff|>0.2 (cf threshold 0.2, autosomes).

## Related tools

- **clair3** (Calls SNVs from the aligned modBAMs using its hifi model; output filtered to PASS SNVs)
- **whatshap** (Phases the PASS SNVs into haplotype-resolved VCFs and phases the reads to yield haplotype-tagged modBAMs)
- **pbmm2** (Aligns and merges ccsmeth modBAM files prior to SNV calling (merged_size1 step)) — https://github.com/PacificBiosciences/pbmm2
- **ccsmethphase** (Nextflow pipeline orchestrating the SNV calling and phasing stages of methylation phasing) — https://github.com/PengNi/ccsmethphase
- **Nextflow** (Workflow engine running the ccsmethphase pipeline stages)

## Examples

```
nextflow run main.nf --dsname hg002_t2t --input input_sheet.tsv --genome T2T-CHM13.fa --include_all_ctgs true -profile singularity
```

## Evaluation signals

- Output directory ccsmethphase_results/<dsname> contains expected bam, vcf (clair3_called and whatshap_phased), mods_freq, and diff_methyl files matching the documented naming pattern, e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.freq.aggregate.hp_callDMR.autosomes_cf0.2.bed
- Phased modBAMs and hp1/hp2 bedmethyl beds exist under ccsmethphase_results after the run
- Haplotype-level methylation correlations against BS-seq/nanopore benchmarks exceed the reported ~0.93 threshold on HG002
- 81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs when validated on HG002
- With --include_all_ctgs false (default), output is restricted to chr1-22,X,Y contigs; setting it true must cover all contigs as requested

## Limitations

- Pipeline detects only symmetric CpG methylation and cannot detect hemimethylated CpGs or non-CpG 5mC or other modifications such as 6mA.
- Phasing quality depends on SNV density and PASS-filtered Clair3 calls; haplotype coverage of CpGs (e.g. 26.97 M phased CpGs at 15x) degrades in some repetitive regions.
- Recommended hardware is substantial: 128 GB RAM, 40 CPU processors, 4 TB disk storage, and >=8 GB GPU for Clair3.

## Evidence

- [results] The ccsmethphase pipeline proceeds through four stages: ccsmeth calls methylation states, Clair3 calls SNVs, WhatsHap phases those SNVs to generate haplotypes, and DSS detects DMRs between haplotypes.: [evidence span withheld — anchor status: not_found]
- [methods] whatshap_phsed: Phased SNVs by whatshap: "whatshap_phsed: Phased SNVs by whatshap"
- [methods] G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam: "G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam"
- [readme] --include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included.: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [abstract] we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"

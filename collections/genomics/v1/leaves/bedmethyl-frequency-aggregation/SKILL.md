---
name: bedmethyl-frequency-aggregation
description: Use when you have haplotype-resolved (haplotagged) modBAM files from
  PacBio HiFi/CCS reads and need per-haplotype CpG methylation frequency tracks (bedmethyl)
  for downstream DMR/ASM analysis or genome-wide CpG coverage quantification.
when_to_use_negative:
- Input is unphased (not haplotagged) modBAM data — per-haplotype (hp1/hp2) bedmethyl
  aggregation requires phased SNVs and whatshap haplotagging
- Detection of non-CpG 5mC, hemimethylated CpGs, or other modifications such as 6mA
  — the methods only consider symmetric CpG methylation
- Read-level (per-molecule) methylation output is needed rather than aggregated site-level
  frequencies
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: ccsmethphase
  role: Nextflow pipeline that orchestrates modBAM generation, phasing, and aggregation
    of haplotype-specific methylation frequencies into bedmethyl files
  repo: https://github.com/PengNi/ccsmethphase
- name: ccsmeth
  role: Deep-learning 5mCpG detection from PacBio CCS reads, producing the modBAM
    input to aggregation
  repo: https://github.com/PengNi/ccsmeth
- name: pbmm2
  role: Aligns/merges modBAM files against the reference genome before SNV calling
    and phasing
  repo: https://github.com/PacificBiosciences/pbmm2
- name: clair3
  role: Calls SNVs from the merged modBAMs; PASS SNVs are phased by whatshap to assign
    reads to haplotypes
- name: whatshap
  role: Phases PASS SNVs to produce haplotype-resolved BAMs that enable per-haplotype
    bedmethyl aggregation
- name: Nextflow
  role: Workflow engine executing the ccsmethphase pipeline steps
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:fc2cfdd24ddb6f96d2429e4ca65468b85a76c9e0b93eab6aa9d4b121d06f2dc5
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
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# bedmethyl-frequency-aggregation

## Summary

Aggregate per-read 5mCpG calls from haplotagged PacBio CCS modBAM files into per-haplotype site-level methylation frequency bedmethyl files (hp1/hp2), a core output step of the ccsmethphase pipeline for methylation phasing on HG002 and T2T-CHM13 analyses.

## When to use

Use when you have haplotype-resolved (haplotagged) modBAM files from PacBio HiFi/CCS reads and need per-haplotype CpG methylation frequency tracks (bedmethyl) for downstream DMR/ASM analysis or genome-wide CpG coverage quantification.

## When NOT to use

- Input is unphased (not haplotagged) modBAM data — per-haplotype (hp1/hp2) bedmethyl aggregation requires phased SNVs and whatshap haplotagging
- Detection of non-CpG 5mC, hemimethylated CpGs, or other modifications such as 6mA — the methods only consider symmetric CpG methylation
- Read-level (per-molecule) methylation output is needed rather than aggregated site-level frequencies

## Inputs

- PacBio HiFi/CCS BAM files listed in an input_sheet.tsv with Group_ID, Sample_ID (e.g., HG002), Type (hifi), and absolute Path columns
- Reference genome FASTA (e.g., T2T-CHM13.fa)
- Haplotagged/phased modBAM files produced by ccsmeth + pbmm2 + clair3 + whatshap

## Outputs

- Per-haplotype bedmethyl files (hp1 and hp2) with site-level methylation frequencies in the mods_freq output directory
- Downstream haplotype methylation frequency tables used for DML/DMR calling and CpG coverage/phasing counts (cpg_coverage_phasing_summary.tsv)

## How to apply

Run the ccsmethphase Nextflow pipeline (nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity) so ccsmeth converts aligned HiFi reads into modified-base (modbam) BAMs with per-read 5mCpG calls, pbmm2 merges them, clair3 calls SNVs, and whatshap phases the PASS SNVs. After phasing, aggregate haplotype-specific site-level methylation frequencies into per-haplotype bedmethyl files (hp1 and hp2) written to the mods_freq output directory. Use --include_all_ctgs true when analyzing contigs beyond chr1-22,X,Y (e.g., T2T-CHM13 acrocentric/cenSat contigs). The resulting bedmethyl tracks feed DSS-based DMR calling and comparisons against BS-seq or nanopore benchmarks.

## Related tools

- **ccsmethphase** (Nextflow pipeline that orchestrates modBAM generation, phasing, and aggregation of haplotype-specific methylation frequencies into bedmethyl files) — https://github.com/PengNi/ccsmethphase
- **ccsmeth** (Deep-learning 5mCpG detection from PacBio CCS reads, producing the modBAM input to aggregation) — https://github.com/PengNi/ccsmeth
- **pbmm2** (Aligns/merges modBAM files against the reference genome before SNV calling and phasing) — https://github.com/PacificBiosciences/pbmm2
- **clair3** (Calls SNVs from the merged modBAMs; PASS SNVs are phased by whatshap to assign reads to haplotypes)
- **whatshap** (Phases PASS SNVs to produce haplotype-resolved BAMs that enable per-haplotype bedmethyl aggregation)
- **Nextflow** (Workflow engine executing the ccsmethphase pipeline steps)

## Examples

```
nextflow run main.nf --dsname hg002_t2t --input input_sheet.tsv --genome T2T-CHM13.fa --include_all_ctgs true -profile singularity
```

## Evaluation signals

- The mods_freq directory exists and contains both hp1 and hp2 bedmethyl files with expected bed format (chrom, start, end, methylation frequency)
- Haplotype-level methylation frequencies correlate with HG002 BS-seq and nanopore benchmarks at Pearson r exceeding the reported ~0.93 threshold
- Downstream intersect of bedmethyl CpG calls with RepeatMasker/segmental-duplication/cenSat annotations reproduces reported coverage (32.85 M / 96.9% CpGs at 15x; 32.74 M at >=5 reads) and phasing count (26.97 M phased CpGs)
- DSS-based DMR calling on the aggregated tracks yields ~14,390 DMRs for 71.0x HG002 CCS reads, with 81.4% of BS-seq DMRs within 10 kb
- All expected ccsmethphase_results outputs present (phased modBAM, hp1/hp2 bedmethyl beds, clair3 and whatshap VCFs, DML/DMR tables)

## Limitations

- Methods cannot detect hemimethylated CpGs or 5mC in non-CpG contexts, nor other modifications such as 6mA
- The strand-specific-methylation model shows reduced performance due to limited subread depth
- Default ccsmethphase --include_all_ctgs is false, restricting analysis to chr1-22,X,Y unless explicitly set to true for T2T-CHM13 all-contig analyses

## Evidence

- [methods] mods_freq bedmethyl outputs: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [methods] include_all_ctgs default: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [abstract] ccsmethphase purpose: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [results] ccsmeth phase coverage and phasing counts: [evidence span withheld — anchor status: not_found]
- [abstract] correlation benchmark: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"

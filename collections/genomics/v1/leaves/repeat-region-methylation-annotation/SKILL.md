---
name: repeat-region-methylation-annotation
description: Use when you need CpG coverage/phasing counts or methylation-state detection
  rates in repetitive genomic regions (RepeatMasker, segmental duplications, cenSat
  annotations) from PacBio HiFi/CCS modbam and bedmethyl data aligned to T2T-CHM13.
when_to_use_negative:
- The goal is detection of hemimethylated CpGs — CCS-based methods like ccsmeth consider
  only symmetric methylation and are incapable of detecting hemimethylated CpGs.
- The target modification is non-CpG 5mC or another DNA modification such as 6mA —
  the models and CpG-intersection workflow do not apply.
- Only primary chromosomes chr1-22,X,Y matter and speed is preferred — running with
  include_all_ctgs=true is unnecessary overhead compared to the default false.
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0657
- http://edamontology.org/topic_3168
tools:
- name: ccsmethphase
  role: Nextflow pipeline that orchestrates 5mC detection, alignment, SNV calling,
    phasing, and per-haplotype bedmethyl aggregation
  repo: https://github.com/PengNi/ccsmethphase
- name: ccsmeth
  role: Deep-learning detection of 5mCpGs from PacBio CCS/HiFi reads, producing modbam
    BAM files
  repo: https://github.com/PengNi/ccsmeth
- name: pbmm2
  role: Aligns and merges ccsmeth modbam BAM files to T2T-CHM13
  repo: https://github.com/PacificBiosciences/pbmm2
- name: clair3
  role: Calls SNVs used for methylation phasing
- name: whatshap
  role: Phases PASS SNVs that anchor haplotype-specific methylation
- name: pb-CpG-tools
  role: Companion tooling for generating site methylation probabilities (bed/bigwig)
    from mapped, haplotagged HiFi reads
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:9ee7655268145bb3cdd39a2c7176d42d45ec94eee02528b32d1f47b4f78322f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmethphase
  tools:
  - Singularity
  - ccsmeth
  - ccsmethphase
  - clair3
  - pb-CpG-tools
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0657
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# repeat-region-methylation-annotation

## Summary

Quantifies how many CpGs genome-wide — and specifically in repetitive regions (RepeatMasker repeats, segmental duplications, cenSats) — are covered and haplotype-phased by PacBio CCS/HiFi methylation calls, by intersecting per-haplotype bedmethyl outputs from ccsmethphase with T2T-CHM13 repeat annotations.

## When to use

Use when you need CpG coverage/phasing counts or methylation-state detection rates in repetitive genomic regions (RepeatMasker, segmental duplications, cenSat annotations) from PacBio HiFi/CCS modbam and bedmethyl data aligned to T2T-CHM13.

## When NOT to use

- The goal is detection of hemimethylated CpGs — CCS-based methods like ccsmeth consider only symmetric methylation and are incapable of detecting hemimethylated CpGs.
- The target modification is non-CpG 5mC or another DNA modification such as 6mA — the models and CpG-intersection workflow do not apply.
- Only primary chromosomes chr1-22,X,Y matter and speed is preferred — running with include_all_ctgs=true is unnecessary overhead compared to the default false.

## Inputs

- PacBio HiFi/CCS BAM files of HG002 (15x coverage) listed in an input_sheet.tsv with Group_ID, Sample_ID, Type, absolute Path columns
- T2T-CHM13 reference genome FASTA
- T2T-CHM13 RepeatMasker repeat annotations
- Segmental duplication annotations
- centromeric satellite (cenSat) annotations

## Outputs

- Haplotype-specific per-CpG bedmethyl files (hp1/hp2) in the mods_freq output directory
- cpg_coverage_phasing_summary.tsv with 15x and >=5-read coverage counts, phased-CpG counts, and region-wise (repeats/SDs/cenSats) detection percentages

## How to apply

Build an input_sheet.tsv (Group_ID, Sample_ID, Type=hifi, absolute Path) of HG002 HiFi BAMs and launch the ccsmethphase Nextflow pipeline with --include_all_ctgs true so all T2T-CHM13 contigs beyond chr1-22,X,Y are analyzed, not just primary chromosomes. The pipeline detects 5mCpGs with ccsmeth into modbam BAMs, aligns/merges with pbmm2, calls SNVs with clair3, phases PASS SNVs with whatshap, and aggregates haplotype-specific site-level methylation frequencies into per-haplotype (hp1/hp2) bedmethyl files in the mods_freq directory. Then intersect these bedmethyl CpG calls with T2T-CHM13 RepeatMasker, segmental-duplication, and cenSat annotations to compute CpG coverage at 15x and >=5-read thresholds, phased-CpG counts, and per-region detection percentages, writing results to cpg_coverage_phasing_summary.tsv. The rationale is that long CCS reads (>10 Kb) make even repetitive regions accessible to CCS-based methylation phasing, so region-wise intersection quantifies the advantage over BS-seq (6.71 M phased CpGs) and nanopore (27.66 M).

## Related tools

- **ccsmethphase** (Nextflow pipeline that orchestrates 5mC detection, alignment, SNV calling, phasing, and per-haplotype bedmethyl aggregation) — https://github.com/PengNi/ccsmethphase
- **ccsmeth** (Deep-learning detection of 5mCpGs from PacBio CCS/HiFi reads, producing modbam BAM files) — https://github.com/PengNi/ccsmeth
- **pbmm2** (Aligns and merges ccsmeth modbam BAM files to T2T-CHM13) — https://github.com/PacificBiosciences/pbmm2
- **clair3** (Calls SNVs used for methylation phasing)
- **whatshap** (Phases PASS SNVs that anchor haplotype-specific methylation)
- **pb-CpG-tools** (Companion tooling for generating site methylation probabilities (bed/bigwig) from mapped, haplotagged HiFi reads) — https://github.com/PacificBiosciences/pb-CpG-tools

## Examples

```
nextflow run main.nf --dsname hg002_t2t --input input_sheet.tsv --genome T2T-CHM13.fa --include_all_ctgs true -profile singularity
```

## Evaluation signals

- Recomputed whole-genome CpG coverage reproduces 32.85 M CpGs (96.9%) covered at 15x and 32.74 M CpGs at >=5 reads.
- Phased-CpG count for CCS is ~26.97 M, comparable to nanopore (27.66 M) and far above BS-seq (6.71 M).
- Region-wise methylation-state detection percentages reproduce 96.8% (RepeatMasker repeats), 88.4% (segmental duplications), and 85.3% (cenSats).
- mods_freq directory contains both hp1 and hp2 per-haplotype bedmethyl files, and intermediate files follow the ...SNV_PASS_whatshap.bam naming scheme.
- Output contigs include all T2T-CHM13 sequences (not only chr1-22,X,Y) when --include_all_ctgs true was used.

## Limitations

- CCS-based 5mC methods consider only symmetric methylation and combine features from both strands, so hemimethylated CpGs are undetectable.
- The approach cannot be applied to non-CpG 5mC detection or other modifications such as 6mA.
- Without --include_all_ctgs=true, only [chr][1-22,X,Y] contigs are included, excluding the repetitive T2T-CHM13 contigs central to this analysis.
- Whether the site-level model design generalizes to non-CpG 5mCs or other modifications has not been verified.

## Evidence

- [abstract] Nextflow pipeline for all-contig phasing: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [intro] Motivation for repetitive-region analysis: "we demonstrate that PacBio CCS is now a comprehensive and accurate technology for 5mCpG detection and methylation phasing even in repetitive genomic regions"
- [methods] include_all_ctgs filter: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [methods] bedmethyl output definition: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [methods] Pipeline input requirement: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [readme] pb-CpG-tools haplotype-specific scores: [evidence span withheld — anchor status: external_doc]

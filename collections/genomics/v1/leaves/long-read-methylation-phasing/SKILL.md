---
name: long-read-methylation-phasing
description: Use when you need haplotype-phased, allele-specific CpG methylation from
  PacBio HiFi/CCS reads, especially across repetitive regions (RepeatMasker repeats,
  segmental duplications, cenSats) where short-read BS-seq fails.
when_to_use_negative:
- Detection of non-CpG 5mC, hemimethylated CpGs, or other modifications such as 6mA
  is required — ccsmeth models only cover symmetric CpG methylation
- Only short reads (Illumina BS-seq) are available — phasing capacity is limited (6.71
  M phased CpGs by BS-seq vs 26.97 M by CCS)
- Subread depth is limited, e.g., strand-specific-methylation modeling where performance
  drops due to limited subread depth (as with the HG002 15 Kb dataset, 0.85 read-level
  accuracy)
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3295
tools:
- name: ccsmethphase
  role: Nextflow pipeline orchestrating methylation detection, alignment, SNV calling,
    and phasing from PacBio CCS reads
  repo: github.com/PengNi/ccsmethphase
- name: ccsmeth
  role: Deep-learning 5mCpG detection from IPD/PW kinetics of CCS reads, producing
    modbam BAMs
  repo: github.com/PengNi/ccsmeth
- name: pbmm2
  role: Alignment/merge of modbam BAMs against the reference genome
  repo: https://github.com/PacificBiosciences/pbmm2
- name: clair3
  role: SNV calling on aligned modbam BAMs
- name: whatshap
  role: Phasing of PASS SNVs used to assign reads to haplotypes
- name: pb-CpG-tools
  role: Companion tool generating site methylation probabilities (bed/bigwig) from
    haplotagged HiFi reads
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
- name: Nextflow
  role: Workflow engine executing the ccsmethphase pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:9e19faf761e332b34a22334df5da448642164e75fc95b2506b9c9573b84fd60e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Nextflow
  - Singularity
  - ccsmeth
  - ccsmethphase
  - clair3
  - pb-CpG-tools
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# long-read-methylation-phasing

## Summary

End-to-end workflow for haplotype-aware 5mCpG detection and methylation phasing from PacBio HiFi/CCS reads using the ccsmethphase Nextflow pipeline (ccsmeth → pbmm2 → clair3 → whatshap → per-haplotype bedmethyl), enabling methylation analysis even in repetitive genomic regions against T2T-CHM13.

## When to use

Use when you need haplotype-phased, allele-specific CpG methylation from PacBio HiFi/CCS reads, especially across repetitive regions (RepeatMasker repeats, segmental duplications, cenSats) where short-read BS-seq fails.

## When NOT to use

- Detection of non-CpG 5mC, hemimethylated CpGs, or other modifications such as 6mA is required — ccsmeth models only cover symmetric CpG methylation
- Only short reads (Illumina BS-seq) are available — phasing capacity is limited (6.71 M phased CpGs by BS-seq vs 26.97 M by CCS)
- Subread depth is limited, e.g., strand-specific-methylation modeling where performance drops due to limited subread depth (as with the HG002 15 Kb dataset, 0.85 read-level accuracy)

## Inputs

- PacBio HiFi (CCS) BAM files (e.g., HG002 15x coverage aligned to T2T-CHM13)
- Reference genome FASTA (e.g., T2T-CHM13.fa)
- input_sheet.tsv with Group_ID, Sample_ID, Type, Path columns
- RepeatMasker, segmental duplication, and cenSat annotations for the reference genome

## Outputs

- Modified-base (modbam) BAM files with per-read 5mCpG calls from ccsmeth
- Clair3 SNV calls and whatshap-phased PASS SNV VCFs
- Haplotype-specific bedmethyl files (hp1/hp2) of site-level methylation frequencies in the mods_freq directory
- Region-wise CpG coverage and phasing summary (e.g., cpg_coverage_phasing_summary.tsv)

## How to apply

Build an input_sheet.tsv with Group_ID, Sample_ID, Type (hifi), and absolute Path columns for the HiFi BAMs, then launch the ccsmethphase Nextflow pipeline with `nextflow run main.nf --dsname hg002_t2t --input input_sheet.tsv --genome T2T-CHM13.fa --include_all_ctgs true -profile singularity`. The pipeline detects 5mCpGs with ccsmeth (deep-learning on IPD/PW kinetics features) to produce modbam BAMs, aligns/merges them with pbmm2, calls SNVs with clair3, and phases PASS SNVs with whatshap. Set `--include_all_ctgs true` to analyze all contigs rather than only chr1-22,X,Y — essential when working with the T2T-CHM13 reference and its centromeric/satellite sequence. Aggregate haplotype-specific site-level methylation frequencies into per-haplotype (hp1/hp2) bedmethyl files in the mods_freq output directory, then intersect with RepeatMasker, segmental-duplication, and cenSat annotations to compute CpG coverage and region-wise methylation-detection percentages.

## Related tools

- **ccsmethphase** (Nextflow pipeline orchestrating methylation detection, alignment, SNV calling, and phasing from PacBio CCS reads) — github.com/PengNi/ccsmethphase
- **ccsmeth** (Deep-learning 5mCpG detection from IPD/PW kinetics of CCS reads, producing modbam BAMs) — github.com/PengNi/ccsmeth
- **pbmm2** (Alignment/merge of modbam BAMs against the reference genome) — https://github.com/PacificBiosciences/pbmm2
- **clair3** (SNV calling on aligned modbam BAMs)
- **whatshap** (Phasing of PASS SNVs used to assign reads to haplotypes)
- **pb-CpG-tools** (Companion tool generating site methylation probabilities (bed/bigwig) from haplotagged HiFi reads) — https://github.com/PacificBiosciences/pb-CpG-tools
- **Nextflow** (Workflow engine executing the ccsmethphase pipeline)

## Examples

```
nextflow run main.nf --dsname hg002_t2t --input input_sheet.tsv --genome T2T-CHM13.fa --include_all_ctgs true -profile singularity
```

## Evaluation signals

- HG002 at 15x against T2T-CHM13 yields 32.85 M CpGs covered (96.9%), with 31.33 M covered by at least 5 reads
- Region-wise methylation detection reaches 96.8% (RepeatMasker repeats), 88.4% (segmental duplications), and 85.3% (cenSats) of CpGs
- Phased-CpG count should approximate 26.97 M for CCS, comparable to nanopore (27.66 M) and far exceeding BS-seq (6.71 M)
- mods_freq output contains paired hp1/hp2 bedmethyl files with valid bedgraph-style methylation frequencies
- Pipeline log shows ccsmeth modbam generation, pbmm2 merge, clair3 SNV calling, and whatshap phasing of SNV_PASS variants completed without errors

## Limitations

- Methods cannot detect hemimethylated CpGs since only symmetric methylation is modeled
- Not applicable to 5mC detection in non-CpGs or other DNA modifications such as 6mA; site-level model design for these is unverified
- HK model comparison point: accurate 5mCpG detection with high CCS subread depth requirements (at least 20× passed subreads per CCS) limits insert size — ccsmeth mitigates this with ≥10 Kb reads
- ccsmeth read-level accuracy on HG002 15 Kb data is 0.85 for the strand-specific-methylation model due to limited subread depth

## Evidence

- [abstract] ccsmethphase pipeline definition: "we develop a Nextflow pipeline, ccsmethphase, to detect haplotype-aware methylation using CCS reads"
- [methods] pipeline input/output: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [methods] workflow toolchain: "G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam"
- [methods] include_all_ctgs filter: "--include_all_ctgs: "true" or "false". default false, means only [chr][1-22,X,Y] included."
- [methods] output bedmethyl: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [intro] repetitive-region capability: "we demonstrate that PacBio CCS is now a comprehensive and accurate technology for 5mCpG detection and methylation phasing even in repetitive genomic regions"
- [abstract] ccsmeth performance: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [discussion] key limitation: "Hence, these methods are incapable of detecting hemimethylated CpGs"
- [readme] pb-CpG-tools role: "pb-CpG-tools provides the tool `aligned_bam_to_cpg_scores`, which can generate site methylation probabilities from
mapped HiFi reads, including probabilities for each haplotype when reads are"

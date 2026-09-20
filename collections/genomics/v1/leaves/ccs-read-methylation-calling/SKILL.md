---
name: ccs-read-methylation-calling
description: Use when input is a PacBio CCS reads BAM (subreads.bam or hifi.bam) plus
  a reference genome and per-read 5mCpG methylation calls are needed, e.g. to build
  haplotype-tagged modbams for allele-specific methylation analysis.
when_to_use_negative:
- Input reads are Oxford Nanopore rather than PacBio CCS — use nanopore-specific callers
  (nanopolish, Megalodon, deepsignal2) instead
- The target modification is not CpG 5mC — ccsmeth cannot detect non-CpG 5mCs or other
  modifications such as 6mA
- Hemimethylated CpGs are the biological question — the model only considers symmetric
  methylation and combines features from both strands, so hemimethylation is invisible
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0654
tools:
- name: ccsmeth
  role: Deep-learning model that detects 5mCpG methylation states from PacBio CCS
    reads at read level, emitting modbam files
  repo: github.com/PengNi/ccsmeth
- name: ccsmethphase
  role: Nextflow pipeline that wraps ccsmeth read-level calling, pbmm2 alignment,
    Clair3 SNV calling, and WhatsHap phasing into haplotype-aware methylation detection
  repo: github.com/PengNi/ccsmethphase
- name: pbmm2
  role: Aligns/merges the ccsmeth modbam files against the reference before SNV calling
    and phasing
  repo: github.com/PacificBiosciences/pbmm2
- name: deepsignal2
  role: Predecessor deep-learning architecture; ccsmeth uses the same DNN structure
    lineage for methylation state prediction
  repo: github.com/PengNi/deepsignal2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:b3d8db6e6c2b6db235ba8ca6b3d44180f30a26baa790049a4dfef2332cf14b37
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DSS
  - Singularity
  - ccsmeth
  - ccsmethphase
  - deepsignal2
  - pbmm2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# ccs-read-methylation-calling

## Summary

Detect DNA 5mCpG methylation states from PacBio circular consensus sequencing (CCS/HiFi) reads at single-molecule (read-level) resolution using ccsmeth, a deep-learning model trained on kinetics features (IPDs and PWs) of treated DNA, producing modbam files that downstream haplotyping workflows consume.

## When to use

Use when input is a PacBio CCS reads BAM (subreads.bam or hifi.bam) plus a reference genome and per-read 5mCpG methylation calls are needed, e.g. to build haplotype-tagged modbams for allele-specific methylation analysis.

## When NOT to use

- Input reads are Oxford Nanopore rather than PacBio CCS — use nanopore-specific callers (nanopolish, Megalodon, deepsignal2) instead
- The target modification is not CpG 5mC — ccsmeth cannot detect non-CpG 5mCs or other modifications such as 6mA
- Hemimethylated CpGs are the biological question — the model only considers symmetric methylation and combines features from both strands, so hemimethylation is invisible

## Inputs

- PacBio CCS HiFi reads BAM (subreads.bam or hifi.bam), e.g. hg002.chr20_demo.hifi.bam
- Reference genome FASTA (e.g. chr20_demo.fa)
- Pre-trained ccsmeth 5mCpG model (read-level)

## Outputs

- Modified-bam (modbam) files with per-read 5mCpG methylation states
- Aligned/merged modbam files after pbmm2 (merged_size1) for downstream SNV calling
- Optional site-level bedmethyl frequency files aggregated from per-read calls

## How to apply

Take PacBio hifi.bam (or subreads.bam) reads and a reference genome, and run ccsmeth's read-level deep-learning model to predict 5mCpG states at single-molecule resolution, converting the alignment to modified-bam (modbam) files. Only symmetric methylation is considered: features from both DNA strands are combined for prediction. Accuracy depends on read length — long (>=10 Kb) CCS reads give the published 0.90 accuracy and 0.97 AUC, so verify insert size/coverage before trusting per-read calls. When embedding in the ccsmethphase Nextflow pipeline, feed the modbam output into pbmm2 alignment/merging (merged_size1) before Clair3 SNV calling and WhatsHap phasing; haplotype-specific site-level frequencies are later aggregated into bedmethyl files (hp1.bed, hp2.bed, all.bed) under mods_freq. Site-level aggregation over the per-read calls should be compared to bisulfite sequencing (BS-seq) ground truth, where ccsmeth reports >0.90 correlations even at 10x coverage.

## Related tools

- **ccsmeth** (Deep-learning model that detects 5mCpG methylation states from PacBio CCS reads at read level, emitting modbam files) — github.com/PengNi/ccsmeth
- **ccsmethphase** (Nextflow pipeline that wraps ccsmeth read-level calling, pbmm2 alignment, Clair3 SNV calling, and WhatsHap phasing into haplotype-aware methylation detection) — github.com/PengNi/ccsmethphase
- **pbmm2** (Aligns/merges the ccsmeth modbam files against the reference before SNV calling and phasing) — github.com/PacificBiosciences/pbmm2
- **deepsignal2** (Predecessor deep-learning architecture; ccsmeth uses the same DNN structure lineage for methylation state prediction) — github.com/PengNi/deepsignal2

## Examples

```
git clone https://github.com/PengNi/ccsmethphase.git
```

## Evaluation signals

- Read-level accuracy of ~0.90 and AUC ~0.97 on 5mCpG detection when CCS reads are >=10 Kb; substantially shorter reads should raise a flag
- Site-level methylation frequencies should correlate >0.90 with BS-seq ground truth (e.g. hg002_bsseq_chr20_demo.bed) even at 10x coverage
- In the ccsmethphase pipeline, output modbams follow the naming pattern G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1... with clair3_called, whatshap_phased, and mods_freq (hp1.bed, hp2.bed, all.bed) directories present
- Methylation predictions are restricted to CpG motifs; any calls at non-CpG positions indicate misconfiguration

## Limitations

- Existing approaches, including ccsmeth's design, only consider symmetric methylation and are incapable of detecting hemimethylated CpGs
- Cannot be applied for 5mC detection in non-CpGs or for detecting other DNA modifications such as 6mA; whether the site-level model extends to non-CpG 5mCs is unverified
- Strand-specific-methylation modeling performance is significantly reduced due to limited subread depth
- Prior SMRT kinetic methods (e.g. the HK model) require high CCS subread depth (at least 20x passed subreads per CCS), which constrains insert size and read length — coverage/insert-size assumptions matter

## Evidence

- [abstract] ccsmeth achieves 0.90 accuracy and 0.97 AUC read-level: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [discussion] two models for read and site level: "we designed two deep-learning models in ccsmeth for 5mCpG detection at the read level and the genome-wide site level, respectively."
- [methods] pipeline inputs are PacBio reads and reference: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [discussion] symmetric methylation assumption, both strands combined: "all these methods only consider symmetric methylation and combine the features from both DNA strands to predict the methylation states."
- [readme] ccsmeth detects methylation from PacBio CCS: [evidence span withheld — anchor status: external_doc]

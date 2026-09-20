---
name: deep-learning-model-training-kinetics-features
description: Use when PacBio CCS subreads.bam/hifi.bam reads plus a reference genome
  are available, M.SssI-treated and PCR-treated control reads exist as ground truth,
  and the goal is to train a deep-learning model on IPD/PW kinetics features for 5mCpG
  detection at read level.
when_to_use_negative:
- The target modification is not symmetric CpG methylation — the methods cannot detect
  hemimethylated CpGs or 5mC in non-CpG contexts, or other modifications such as 6mA
- Input is short-read sequencing (e.g., BS-seq, TAPS, EM-seq) or Oxford Nanopore reads
  — ccsmeth's kinetics features (IPDs and PWs) are specific to PacBio CCS data
- Only site-level aggregate methylation frequencies are needed and no per-read ground
  truth or model training is intended
tools:
- name: ccsmeth
  role: Deep-learning model trained on IPD/PW kinetics features of PacBio CCS reads
    for read-level and site-level 5mCpG detection; its 5mCpG models provide the model
    framework
  repo: github.com/PengNi/ccsmeth
- name: HK model
  role: CNN-based holistic kinetic model trained on the same 50% training split as
    the read-level 5mCpG detection baseline for comparison
- name: ccsmethphase
  role: Downstream Nextflow pipeline consuming ccsmeth-style PacBio reads (subreads.bam
    or hifi.bam) plus a reference genome for haplotype-aware methylation detection
  repo: github.com/PengNi/ccsmethphase
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:efe1b3813d448903b7a93fb3c9f6d868fe758587b8b8977b6d1c1206168479b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HK model
  - ccsmeth
  - ccsmethphase
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# deep-learning-model-training-kinetics-features

## Summary

Train and evaluate ccsmeth, a deep-learning model that detects DNA 5mCpGs at read level using kinetics features (IPDs and PWs) extracted from PacBio CCS reads, on controlled M.SssI-methyltransferase-treated versus PCR-treated DNA datasets.

## When to use

Use when PacBio CCS subreads.bam/hifi.bam reads plus a reference genome are available, M.SssI-treated and PCR-treated control reads exist as ground truth, and the goal is to train a deep-learning model on IPD/PW kinetics features for 5mCpG detection at read level.

## When NOT to use

- The target modification is not symmetric CpG methylation — the methods cannot detect hemimethylated CpGs or 5mC in non-CpG contexts, or other modifications such as 6mA
- Input is short-read sequencing (e.g., BS-seq, TAPS, EM-seq) or Oxford Nanopore reads — ccsmeth's kinetics features (IPDs and PWs) are specific to PacBio CCS data
- Only site-level aggregate methylation frequencies are needed and no per-read ground truth or model training is intended

## Inputs

- Aligned PacBio CCS reads (subreads.bam or hifi.bam) from M.SssI-methyltransferase-treated DNA
- Aligned PacBio CCS reads (subreads.bam or hifi.bam) from PCR-treated (unmethylated control) DNA
- Reference genome FASTA
- input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns
- ccsmeth 5mCpG model framework files from github.com/PengNi/ccsmeth

## Outputs

- Trained ccsmeth deep-learning model for read-level 5mCpG detection
- Read-level 5mCpG predictions on held-out test CCS reads
- Per-dataset accuracy and AUC metrics (e.g., written to ccsmeth_read_level_benchmark.csv)

## How to apply

Download the controlled M.SssI-treated versus PCR-treated PacBio CCS dataset pairs and organize aligned subreads.bam/hifi.bam files plus the reference genome in an input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns. Split each dataset pair 50/50 at the read level into training and test sets. Train ccsmeth using the ccsmeth 5mCpG model framework on the kinetics features (IPDs and PWs) of the training-split CCS reads, and train the CNN-based holistic kinetic (HK) model on the same 50% training split as a baseline. Evaluate both models at read-level 5mCpG classification on the held-out 50% test reads, computing accuracy and AUC per dataset pair. As a decision point, treat ccsmeth's use of long (≥10 Kb) CCS reads at single-molecule resolution as the regime in which its 0.90 accuracy and 0.97 AUC reference performance applies.

## Related tools

- **ccsmeth** (Deep-learning model trained on IPD/PW kinetics features of PacBio CCS reads for read-level and site-level 5mCpG detection; its 5mCpG models provide the model framework) — github.com/PengNi/ccsmeth
- **HK model** (CNN-based holistic kinetic model trained on the same 50% training split as the read-level 5mCpG detection baseline for comparison)
- **ccsmethphase** (Downstream Nextflow pipeline consuming ccsmeth-style PacBio reads (subreads.bam or hifi.bam) plus a reference genome for haplotype-aware methylation detection) — github.com/PengNi/ccsmethphase

## Evaluation signals

- Reproduced read-level accuracies on the M01&W01, M02&W02, and M03&W03 pairs match 0.9232, 0.8788, and 0.8765 within a small tolerance
- ccsmeth's AUC exceeds the HK model's by approximately 5.4%, 4.4%, and 3.7% on the three dataset pairs respectively
- Train/test read split is exactly 50/50 and performed at the read level, with both ccsmeth and the HK model trained on the identical training split
- Reference long-read regime check: ccsmeth should approach 0.90 accuracy and 0.97 AUC on 5mCpG detection using long (≥10 Kb) CCS reads at single-molecule resolution
- All per-dataset metrics are written to ccsmeth_read_level_benchmark.csv with one row per dataset pair

## Limitations

- ccsmeth and similar methods only consider symmetric methylation and are incapable of detecting hemimethylated CpGs
- The methods cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA
- Whether the site-level model design can be directly applied to non-CpG 5mCs and other modifications has not been verified
- The HK model baseline requires relatively high CCS subread depth (at least 20× passed subreads for one CCS), limiting insert size and read length

## Evidence

- [intro] we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [abstract] Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 AUC at single-molecule resolution: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [abstract] We sequence PCR-treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth: "We sequence polymerase-chain-reaction treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth"
- [methods] ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input.: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [intro] the HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS): "the HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS)"

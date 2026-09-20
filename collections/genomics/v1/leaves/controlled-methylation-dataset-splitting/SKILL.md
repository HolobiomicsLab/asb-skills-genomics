---
name: controlled-methylation-dataset-splitting
description: 'Use when benchmarking read-level 5mCpG detection models on methylation-ground-truth
  CCS data: controlled M.SssI-treated/PCR-treated paired samples exist, and a fair
  train/test read split is needed to compare model accuracies and AUCs.'
when_to_use_negative:
- The goal is methylation phasing or haplotype-aware methylation with ccsmethphase
  rather than controlled read-level benchmarking of 5mCpG callers
- Input data lacks methylation ground truth (e.g., native HG002/CHM13 reads without
  M.SssI or PCR controls), so no supervised train/test split can be labeled
- Evaluation is site-level methylation frequency rather than single-molecule (read-level)
  5mCpG classification
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ccsmeth
  role: Deep-learning 5mCpG model trained on IPD and PW kinetics features of the training
    split of the controlled CCS reads and evaluated on the held-out split
  repo: github.com/PengNi/ccsmeth
- name: HK model
  role: CNN-based holistic kinetic baseline trained on the same 50% training split
    of each dataset pair for head-to-head read-level accuracy and AUC comparison
- name: pbmm2
  role: Aligns the subreads.bam/hifi.bam CCS reads to the reference genome before
    feature extraction and splitting
  repo: github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:09300e6c236aa817923a93f533468cad7f6722f3457112869c2164235686a94c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HK model
  - ccsmeth
  - pbmm2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# controlled-methylation-dataset-splitting

## Summary

Organizes matched M.SssI-methyltransferase-treated versus PCR-treated PacBio CCS dataset pairs (e.g., M01&W01, M02&W02, M03&W03) and splits each pair 50/50 at the read level into training and test sets, enabling a controlled, ground-truth benchmark of 5mCpG callers such as ccsmeth against the HK model.

## When to use

Use when benchmarking read-level 5mCpG detection models on methylation-ground-truth CCS data: controlled M.SssI-treated/PCR-treated paired samples exist, and a fair train/test read split is needed to compare model accuracies and AUCs.

## When NOT to use

- The goal is methylation phasing or haplotype-aware methylation with ccsmethphase rather than controlled read-level benchmarking of 5mCpG callers
- Input data lacks methylation ground truth (e.g., native HG002/CHM13 reads without M.SssI or PCR controls), so no supervised train/test split can be labeled
- Evaluation is site-level methylation frequency rather than single-molecule (read-level) 5mCpG classification

## Inputs

- PacBio CCS read files (subreads.bam or hifi.bam) for M.SssI-methyltransferase-treated and PCR-treated DNA (paired samples such as M01&W01, M02&W02, M03&W03)
- Reference genome for read alignment
- input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns describing each dataset

## Outputs

- 50/50 train/test read split per dataset pair
- Read-level accuracy and AUC metrics for ccsmeth and the HK model on held-out test reads
- ccsmeth_read_level_benchmark.csv of comparison metrics

## How to apply

Download and align the paired M.SssI-methyltransferase-treated and PCR-treated PacBio CCS datasets (subreads.bam or hifi.bam) to the reference genome, then register the files in an input_sheet.tsv with Group_ID, Sample_ID, Type, and Path columns. Split each dataset pair 50/50 into training and test reads at the read level (not the site level) so that evaluation reads are fully held out. Train ccsmeth on the kinetics features (IPDs and PWs) of the training split using the ccsmeth 5mCpG model framework, and train the CNN-based HK (holistic kinetic) model on the identical training split to keep the comparison controlled. Evaluate both models at read-level 5mCpG classification on the held-out test reads, recording accuracy and AUC per dataset pair and writing metrics to ccsmeth_read_level_benchmark.csv. The 50/50 split is the key control: it prevents leakage between training and test reads and lets the reproduction be validated against the reported accuracies of 0.9232, 0.8788, and 0.8765.

## Related tools

- **ccsmeth** (Deep-learning 5mCpG model trained on IPD and PW kinetics features of the training split of the controlled CCS reads and evaluated on the held-out split) — github.com/PengNi/ccsmeth
- **HK model** (CNN-based holistic kinetic baseline trained on the same 50% training split of each dataset pair for head-to-head read-level accuracy and AUC comparison)
- **pbmm2** (Aligns the subreads.bam/hifi.bam CCS reads to the reference genome before feature extraction and splitting) — github.com/PacificBiosciences/pbmm2

## Evaluation signals

- Reproduced read-level accuracies on M01&W01, M02&W02, and M03&W03 match 0.9232, 0.8788, and 0.8765 within a small tolerance
- ccsmeth's AUC exceeds the HK model's by approximately 5.4%, 4.4%, and 3.7% on the three dataset pairs respectively
- Training and test read sets are disjoint (no read appears in both halves) after the 50/50 read-level split
- input_sheet.tsv rows resolve to existing aligned BAM paths and conform to the Group_ID/Sample_ID/Type/Path schema
- All metrics for the three dataset pairs are present in ccsmeth_read_level_benchmark.csv with accuracy and AUC columns for both models

## Limitations

- The controlled M.SssI/PCR setup detects only symmetric CpG methylation; hemimethylated CpGs and other modifications such as 6mA are not covered by these ground-truth labels
- The HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS), limiting insert size and read length in the comparison
- Metrics are specific to the 50/50 read-level split; they do not directly transfer to site-level or low-coverage evaluation regimes

## Evidence

- [abstract] Training ccsmeth on PacBio CCS reads of PCR-treated and M.SssI-methyltransferase-treated DNA: "We sequence polymerase-chain-reaction treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth"
- [intro] Controlled M.SssI-treated dataset ground truth: "we sequenced amplified and M.SssI-treated DNA of human sample NA12898 using PacBio CCS with 10 Kb insert size"
- [intro] HK model subread-depth requirement constraining the comparison: "the HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS)"
- [readme] ccsmeth input file types: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."

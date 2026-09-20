---
name: cnn-model-evaluation-on-long-reads
description: Use when comparing read-level 5mCpG detection performance of trained
  deep-learning models on long PacBio HiFi/CCS reads (≥10 Kb) against bisulfite ground
  truth and a baseline CNN caller such as primrose.
when_to_use_negative:
- The goal is site-level (genome-wide aggregated) methylation rather than single-molecule
  read-level accuracy — ccsmeth has a separate site-level model for that
- Detecting hemimethylated CpGs or non-CpG 5mC / other modifications such as 6mA —
  the methods only consider symmetric CpG methylation
- Evaluating on short (<10 Kb) CCS reads, where the reported ≥0.90 accuracy and ~0.97
  AUC performance does not apply
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3168
tools:
- name: ccsmeth
  role: Trained deep-learning 5mCpG read-level caller using IPD/PW kinetics features
    from PacBio CCS reads; source of the model under evaluation
  repo: https://github.com/PengNi/ccsmeth
- name: primrose
  role: PacBio's CNN-based 5mCpG read-level caller used as the comparison baseline
    on the same aligned CCS BAMs
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
- name: pbmm2
  role: Aligns PacBio HiFi reads to the reference genome to produce the sorted BAMs
    consumed by ccsmeth and primrose
  repo: https://github.com/PacificBiosciences/pbmm2
- name: ccs
  role: Generates the PacBio HiFi (CCS) reads evaluated in the benchmark
  repo: https://github.com/PacificBiosciences/ccs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:b0d1023065182fdf6a04d8b2ff5af33a3b9c4b0f61d36668af6f96fdc9f8018e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccs
  - ccsmeth
  - pbmm2
  - primrose
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cnn-model-evaluation-on-long-reads

## Summary

Benchmarking a CNN-based deep-learning methylation caller (ccsmeth) against a competing CNN caller (primrose) at single-molecule read level on long (≥10 Kb) PacBio CCS reads, computing accuracy, sensitivity, specificity, and AUC against bisulfite-sequencing ground truth.

## When to use

Use when comparing read-level 5mCpG detection performance of trained deep-learning models on long PacBio HiFi/CCS reads (≥10 Kb) against bisulfite ground truth and a baseline CNN caller such as primrose.

## When NOT to use

- The goal is site-level (genome-wide aggregated) methylation rather than single-molecule read-level accuracy — ccsmeth has a separate site-level model for that
- Detecting hemimethylated CpGs or non-CpG 5mC / other modifications such as 6mA — the methods only consider symmetric CpG methylation
- Evaluating on short (<10 Kb) CCS reads, where the reported ≥0.90 accuracy and ~0.97 AUC performance does not apply

## Inputs

- Five long (≥10 Kb) PacBio CCS test BAM files: NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, SD0651_P1 15 Kb
- Matched bisulfite-sequencing (BS-seq) ground-truth CpG methylation labels for each test set
- Pre-trained ccsmeth 5mCpG read-level model
- Aligned HiFi BAMs (pbmm2-aligned)

## Outputs

- read_level_accuracy_metrics.csv with one row per test set per method containing accuracy, sensitivity, specificity, and AUC
- Per-read per-CpG methylation probability calls from ccsmeth and primrose

## How to apply

Collect five long-read PacBio CCS test sets (NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, SD0651_P1 15 Kb) with matched BS-seq ground truth and the pre-trained ccsmeth 5mCpG read-level model. Align the hifi BAMs (e.g., with pbmm2), then run ccsmeth in read-level mode using kinetics features (IPD and pulse width) to assign per-CpG methylation probabilities per read. Run primrose (PacBio's CNN-based 5mCpG caller) on the same aligned CCS BAMs as the comparison baseline. Match predicted read-level 5mCpG calls from both tools against methylated/unmethylated BS-seq CpG labels, then compute accuracy, sensitivity, specificity, and ROC AUC per test set per method and assemble into a metrics CSV (one row per test set per method).

## Related tools

- **ccsmeth** (Trained deep-learning 5mCpG read-level caller using IPD/PW kinetics features from PacBio CCS reads; source of the model under evaluation) — https://github.com/PengNi/ccsmeth
- **primrose** (PacBio's CNN-based 5mCpG read-level caller used as the comparison baseline on the same aligned CCS BAMs) — https://github.com/PacificBiosciences/pb-CpG-tools
- **pbmm2** (Aligns PacBio HiFi reads to the reference genome to produce the sorted BAMs consumed by ccsmeth and primrose) — https://github.com/PacificBiosciences/pbmm2
- **ccs** (Generates the PacBio HiFi (CCS) reads evaluated in the benchmark) — https://github.com/PacificBiosciences/ccs

## Examples

```
ccsmeth call_readlevel --input aligned.hifi.bam --model ccsmeth_model_5mCpG.pkl --threads 40 --output ccsmeth_readlevel
```

## Evaluation signals

- Reproduced ccsmeth read-level metrics fall within reported ranges: accuracy 0.8721–0.9062, sensitivity 0.8621–0.8903, specificity 0.8765–0.9220, AUC 0.9464–0.9682
- ccsmeth metrics exceed primrose metrics on all five test datasets
- Results are consistent with the headline claim of ~0.90 accuracy and 0.97 AUC on long (≥10 Kb) CCS reads
- Output CSV contains exactly one row per (test set, method) pair across the five test sets
- Predicted read-level calls match BS-seq methylated/unmethylated CpG labels on matched CpG positions

## Limitations

- Methods only consider symmetric methylation and combine features from both DNA strands, so hemimethylated CpGs cannot be detected
- The approach cannot be applied to non-CpG 5mC detection or other DNA modifications such as 6mA
- Performance claims are tied to long (≥10 Kb) CCS reads; the strand-specific-methylation model shows significantly reduced performance due to limited subread depth

## Evidence

- [abstract] ccsmeth achieves 0.90 accuracy and 0.97 AUC on 5mCpG detection at single-molecule resolution: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] primrose claimed 85% read-level accuracy: "PacBio proposed another CNN-based method primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [intro] ccsmeth uses kinetics features IPDs and PWs: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [discussion] ccsmeth outperforms existing CNN-based methods: "ccsmeth outperforms two existing CNN-based methods on both read-level and site-level 5mC prediction."
- [methods] ccsmeth models on GitHub: "Check ccsmeth (https://github.com/PengNi/ccsmeth) to get ccsmeth 5mCpG models."

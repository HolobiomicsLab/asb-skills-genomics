---
name: bisulfite-ground-truth-metric-computation
description: Use when read-level 5mCpG predictions from PacBio CCS callers need objective
  scoring against BS-seq ground truth, e.g., reproducing ccsmeth's ~0.90 accuracy
  and ~0.97 AUC on long (>=10 Kb) HiFi reads.
when_to_use_negative:
- Evaluating site-level (genome-wide aggregated) methylation rather than single-molecule
  read-level calls, where correlations with BS-seq are the appropriate metric
- The goal is detecting non-CpG 5mC or other modifications such as 6mA, since these
  models were designed only for symmetric CpG methylation
- Detecting hemimethylated CpGs, which these methods are incapable of detecting
edam_operation: http://edamontology.org/operation_3203
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_3295
tools:
- name: ccsmeth
  role: Read-level 5mCpG caller using IPD/PW kinetics features and a deep-learning
    model trained on PCR-treated and M.SssI-treated PacBio CCS reads
  repo: https://github.com/PengNi/ccsmeth
- name: primrose
  role: PacBio CNN-based 5mCpG read-level caller used as the comparison baseline (~85%
    claimed read-level accuracy on long CCS reads)
  repo: https://github.com/PacificBiosciences/pb-CpG-tools
- name: pbmm2
  role: Aligns PacBio HiFi reads to the reference genome before methylation calling
  repo: https://github.com/PacificBiosciences/pbmm2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:c711946a881eff6240ea7e75b721de15d72d6af9c61686aea9521deee595ce93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccsmeth
  - pbmm2
  - primrose
  edam_operation: http://edamontology.org/operation_3203
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# bisulfite-ground-truth-metric-computation

## Summary

Benchmark read-level 5mCpG methylation callers (e.g., ccsmeth vs primrose) by matching per-CpG predictions against bisulfite-sequencing ground truth and computing accuracy, sensitivity, specificity, and AUC per test set.

## When to use

Use when read-level 5mCpG predictions from PacBio CCS callers need objective scoring against BS-seq ground truth, e.g., reproducing ccsmeth's ~0.90 accuracy and ~0.97 AUC on long (>=10 Kb) HiFi reads.

## When NOT to use

- Evaluating site-level (genome-wide aggregated) methylation rather than single-molecule read-level calls, where correlations with BS-seq are the appropriate metric
- The goal is detecting non-CpG 5mC or other modifications such as 6mA, since these models were designed only for symmetric CpG methylation
- Detecting hemimethylated CpGs, which these methods are incapable of detecting

## Inputs

- Aligned PacBio HiFi/CCS BAM files for five long-read test sets (NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, SD0651_P1 15 Kb)
- Bisulfite-sequencing (BS-seq) ground-truth CpG methylation labels per test set (e.g., BED format like hg002_bsseq_chr20_demo.bed)
- Pre-trained ccsmeth 5mCpG read-level model
- Read-level methylation probability calls from each method (ccsmeth and primrose)

## Outputs

- Per-read per-CpG methylation predictions from ccsmeth and primrose
- read_level_accuracy_metrics.csv with one row per test set per method containing accuracy, sensitivity, specificity, and AUC
- Validation report comparing reproduced metrics to reported ranges (accuracy 0.8721-0.9062, AUC 0.9464-0.9682)

## How to apply

Collect the aligned hifi BAM test sets (NA12898 chrX 10 Kb insert, HG002 15/20/24 Kb, SD0651_P1 15 Kb) together with their BS-seq ground truth and run each caller in read-level mode: ccsmeth uses kinetics features (IPD and pulse width) with its pre-trained 5mCpG model, while primrose is PacBio's CNN-based baseline. Match predicted per-read per-CpG methylated/unmethylated calls from both methods against the bisulfite-derived CpG labels, then compute accuracy, sensitivity, specificity, and ROC AUC for each method on each test set. Assemble the metrics into one row per test set per method (read_level_accuracy_metrics.csv). Validate by checking that reproduced ccsmeth metrics fall in the reported ranges: accuracy 0.8721-0.9062, sensitivity 0.8621-0.8903, specificity 0.8765-0.9220, AUC 0.9464-0.9682, confirming ccsmeth outperforms primrose on all five datasets.

## Related tools

- **ccsmeth** (Read-level 5mCpG caller using IPD/PW kinetics features and a deep-learning model trained on PCR-treated and M.SssI-treated PacBio CCS reads) — https://github.com/PengNi/ccsmeth
- **primrose** (PacBio CNN-based 5mCpG read-level caller used as the comparison baseline (~85% claimed read-level accuracy on long CCS reads)) — https://github.com/PacificBiosciences/pb-CpG-tools
- **pbmm2** (Aligns PacBio HiFi reads to the reference genome before methylation calling) — https://github.com/PacificBiosciences/pbmm2

## Examples

```
ccsmeth call_read --input aligned.hifi.bam --model ccsmeth_model_5mCpG --threads 16 --prob_thresh 0.9 --motif CG
```

## Evaluation signals

- Reproduced ccsmeth accuracy on each of the five test sets falls within 0.8721-0.9062
- Reproduced ccsmeth AUC falls within 0.9464-0.9682, consistent with the reported ~0.97 AUC on long (>=10 Kb) CCS reads
- ccsmeth metrics (accuracy, sensitivity, specificity, AUC) exceed primrose on all five datasets
- read_level_accuracy_metrics.csv contains one row per test set per method with all four metrics populated
- Predicted calls match BS-seq CpG labels only at CpG positions, using matched read-level methylated/unmethylated labels

## Limitations

- Ground truth is limited to BS-seq-derived symmetric CpG methylation; hemimethylated CpGs and non-CpG 5mC or 6mA cannot be evaluated
- The strand-specific-methylation model variant has significantly reduced performance due to limited subread depth
- Primrose's claimed baseline is ~85% read-level accuracy, so metric gaps partly reflect the baseline's maturity

## Evidence

- [abstract] ccsmeth achieves 0.90 accuracy and 0.97 AUC on 5mCpG detection at single-molecule resolution: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] primrose claimed 85% read-level accuracy: "PacBio proposed another CNN-based method primrose25, which has been claimed to have 85% read-level accuracy on 5mCpG detection from long PacBio CCS reads"
- [abstract] ccsmeth trained on PCR-treated and M.SssI-treated DNA: "We sequence polymerase-chain-reaction treated and M.SssI-methyltransferase treated DNA of one human sample using PacBio CCS for training ccsmeth"
- [intro] SD0651_P1 has matched BS-seq ground truth: "We also sequenced a human male sample SD0651_P1 using both PacBio CCS with 15 Kb insert size and BS-seq"
- [methods] ccsmethphase takes hifi.bam and a reference genome as input: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."

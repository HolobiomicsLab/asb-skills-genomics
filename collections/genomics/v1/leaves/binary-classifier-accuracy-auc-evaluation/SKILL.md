---
name: binary-classifier-accuracy-auc-evaluation
description: Use when two or more binary classifiers (e.g. ccsmeth and the HK model)
  have been trained on the same training split and must be compared on held-out reads
  using accuracy and AUC metrics.
when_to_use_negative:
- Evaluating genome-wide site-level methylation frequencies rather than single-molecule
  (read-level) CpG calls — a separate site-level evaluation is needed
- Data without methylation ground truth (e.g. native unmodified DNA) where M.SssI/PCR-controlled
  labels cannot be constructed
- Detecting modifications other than symmetric 5mCpG (e.g. hemimethylated CpGs, non-CpG
  5mCs, or 6mA), which these models are incapable of detecting
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0102
tools:
- name: ccsmeth
  role: Deep-learning model whose read-level 5mCpG predictions (accuracy 0.9232, 0.8788,
    0.8765) are evaluated against ground truth using accuracy and AUC
  repo: https://github.com/PengNi/ccsmeth
- name: HK model
  role: CNN-based holistic kinetic baseline model evaluated on the same held-out test
    reads to quantify ccsmeth's accuracy margins (5.4%, 4.4%, 3.7%)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:96f00c2d6d8df488eb6c6aa90712bc52193b3b9fd06e03c4c379eea93ec0dcee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - HK model
  - ccsmeth
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# binary-classifier-accuracy-auc-evaluation

## Summary

Evaluates binary classification performance (here, 5mCpG methylated vs unmethylated calls at read level) by computing accuracy and AUC on a held-out test split, enabling head-to-head comparison of models such as ccsmeth versus the CNN-based HK model on identical data.

## When to use

Use when two or more binary classifiers (e.g. ccsmeth and the HK model) have been trained on the same training split and must be compared on held-out reads using accuracy and AUC metrics.

## When NOT to use

- Evaluating genome-wide site-level methylation frequencies rather than single-molecule (read-level) CpG calls — a separate site-level evaluation is needed
- Data without methylation ground truth (e.g. native unmodified DNA) where M.SssI/PCR-controlled labels cannot be constructed
- Detecting modifications other than symmetric 5mCpG (e.g. hemimethylated CpGs, non-CpG 5mCs, or 6mA), which these models are incapable of detecting

## Inputs

- Held-out test-split PacBio CCS reads (subreads.bam/hifi.bam) with per-read 5mCpG predictions from each trained model
- Per-read methylation ground-truth labels derived from controlled M.SssI-methylated vs PCR-treated (unmethylated) dataset pairs
- Trained ccsmeth and HK model checkpoints trained on the identical 50% training split

## Outputs

- Per-dataset read-level accuracy and AUC values for each model
- Differential metrics (e.g. accuracy/AUC margins of ccsmeth over the HK model)
- Consolidated benchmark table (e.g. ccsmeth_read_level_benchmark.csv)

## How to apply

Split each dataset pair (e.g. M.SssI-treated vs PCR-treated PacBio CCS reads for M01&W01, M02&W02, M03&W03) 50/50 into training and test reads at the read level. Train each model — e.g. ccsmeth using kinetics features (IPDs and PWs) with ccsmeth 5mCpG models as framework, and the CNN-based HK (holistic kinetic) model — on the identical 50% training split. Run both trained models on the held-out 50% test reads to obtain per-read 5mCpG predictions, then compute accuracy and Area Under the Curve (AUC) per dataset. Report all metrics (e.g. ccsmeth accuracies of 0.9232, 0.8788, 0.8765 and margins of 5.4%, 4.4%, 3.7% over the HK model) to a structured table such as ccsmeth_read_level_benchmark.csv for verification.

## Related tools

- **ccsmeth** (Deep-learning model whose read-level 5mCpG predictions (accuracy 0.9232, 0.8788, 0.8765) are evaluated against ground truth using accuracy and AUC) — https://github.com/PengNi/ccsmeth
- **HK model** (CNN-based holistic kinetic baseline model evaluated on the same held-out test reads to quantify ccsmeth's accuracy margins (5.4%, 4.4%, 3.7%))

## Examples

```
python evaluate_read_level.py --datasets input_sheet.tsv --split 0.5 --models ccsmeth,hk --metrics accuracy,auc --output ccsmeth_read_level_benchmark.csv
```

## Evaluation signals

- Reproduced read-level accuracies match reported values of 0.9232 (M01&W01), 0.8788 (M02&W02), and 0.8765 (M03&W03) within a small tolerance
- ccsmeth's AUC exceeds the HK model's by approximately 5.4%, 4.4%, and 3.7% on the three dataset pairs respectively
- Both models are evaluated on the identical 50% held-out test reads of each dataset pair, ensuring a fair comparison
- All accuracy and AUC values are written to ccsmeth_read_level_benchmark.csv with one row per dataset/model combination

## Limitations

- Evaluation on controlled M.SssI-methylated vs PCR-treated DNA reflects symmetric CpG methylation only; results may not transfer to natural, partially methylated methylomes
- The HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS), constraining comparable benchmarking conditions
- The HK model's subread-depth dependence limits insert size and read length, which may bias read-level accuracy comparisons on long reads

## Evidence

- [results] ccsmeth achieves read-level accuracies of 0.9232, 0.8788, and 0.8765 on M01&W01, M02&W02, and M03&W03, respectively, which are 5.4%, 4.4%, and 3.7% higher than the HK model: [evidence span withheld — anchor status: not_found]
- [abstract] ccsmeth 0.90 accuracy 0.97 AUC: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [intro] ccsmeth uses kinetics features IPDs and PWs of PacBio CCS reads: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [intro] HK model requires at least 20x passed subreads: "the HK model requires relatively high CCS subread depth (at least 20× passed subreads for one CCS) for accurate 5mCpG detection"
- [abstract] ccsmeth and ccsmethphase are robust and accurate tools: "ccsmeth and ccsmethphase can be robust and accurate tools for detecting DNA 5-methylcytosines."

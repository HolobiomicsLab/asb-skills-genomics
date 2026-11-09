---
name: random-forest-consensus-model-training
description: Use when you have per-read methylation score files (ID/Chr/Pos/Strand/Score
  columns) from two or more nanopore CpG methylation tools run on the same reads,
  and need a consensus per-read methylation probability more accurate than either
  tool alone.
when_to_use_negative:
- You only have per-read scores from a single methylation tool — the consensus model
  requires outputs from two or more tools to ameliorate individual method deficiencies
- Your goal is per-site methylation frequency directly from one tool's output (e.g.
  Megalodon alone already gave the best single-tool performance); the consensus adds
  substantial compute (DeepSignal + Megalodon require GPUs for efficiency)
- Your training and evaluation data come from the same controlled mixture — the reported
  RMSE improvement must be demonstrated on an independent dataset (mixture dataset
  2)
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3474
tools:
- name: METEORE
  role: Provides combination_model_train.py with the optimised random forest (-c 2)
    that combines per-read scores from multiple methylation callers into a consensus
    model
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Upstream methylation caller whose per-read CpG scores form one input channel
    to the random forest consensus model
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Upstream deep-learning methylation caller whose per-read CpG scores form the
    second input channel to the random forest consensus model
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Snakemake
  role: Wraps the per-tool methylation calling pipelines whose outputs feed the consensus
    model
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:1de69b891e79f17f61dd164e65369e8d8124e879952acd02e3577e4750c7b1b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  - Snakemake
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# random-forest-consensus-model-training

## Summary

Train METEORE's random forest consensus model that combines per-read 5mC methylation predictions from multiple nanopore methylation callers (e.g. Megalodon and DeepSignal) into a single consensus score with higher accuracy than any individual tool. The optimised model uses n_estimators=10 and max_depth=3.

## When to use

Use when you have per-read methylation score files (ID/Chr/Pos/Strand/Score columns) from two or more nanopore CpG methylation tools run on the same reads, and need a consensus per-read methylation probability more accurate than either tool alone.

## When NOT to use

- You only have per-read scores from a single methylation tool — the consensus model requires outputs from two or more tools to ameliorate individual method deficiencies
- Your goal is per-site methylation frequency directly from one tool's output (e.g. Megalodon alone already gave the best single-tool performance); the consensus adds substantial compute (DeepSignal + Megalodon require GPUs for efficiency)
- Your training and evaluation data come from the same controlled mixture — the reported RMSE improvement must be demonstrated on an independent dataset (mixture dataset 2)

## Inputs

- Per-read methylation score file from tool 1 (e.g. Megalodon), tab-delimited with ID/Chr/Pos/Strand/Score columns
- Per-read methylation score file from tool 2 (e.g. DeepSignal), same format, aligned on read ID and CpG position
- Training labels from a controlled methylation mixture dataset (e.g. mixture dataset 1 from the figshare deposit https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)

## Outputs

- Trained random forest model file (rf_model_max_depth_3_n_estimator_10 variant, n_estimators=10, max_depth=3) in the output_models directory
- Downstream: per-read consensus methylation predictions and per-site methylation frequency tables (augmented BED format: chromosome, start, end, coverage, methylation frequency, strandedness)

## How to apply

Prepare per-read score files for each tool in the ID/Chr/Pos/Strand/Score format and merge them on read ID and CpG position so that scores from all tools align on the same reads. Train the optimised random forest with METEORE's combination_model_train.py using -c 2 to select the optimised configuration (rf_model_max_depth_3_n_estimator_10: n_estimators=10, max_depth=3), saving the pickled model to an output_models directory. Training pairs (e.g. mixture dataset 1 from the figshare deposit with known methylation proportions) should be independent from the evaluation data (e.g. mixture dataset 2) to test generalisation. Train the model on per-read scores whose labels derive from controlled mixtures of methylated and unmethylated reads, since the model learns to regress or vote toward the true per-read methylation status. After training, apply the model with combination_model_prediction.py (-m optimized) and convert per-read consensus calls to per-site methylation frequencies with prediction_to_mod_frequency.py using a Prob_methylation threshold of 0.5.

## Related tools

- **METEORE** (Provides combination_model_train.py with the optimised random forest (-c 2) that combines per-read scores from multiple methylation callers into a consensus model) — https://github.com/comprna/METEORE
- **Megalodon** (Upstream methylation caller whose per-read CpG scores form one input channel to the random forest consensus model) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Upstream deep-learning methylation caller whose per-read CpG scores form the second input channel to the random forest consensus model) — https://github.com/bioinfomaticsCSU/deepsignal
- **Snakemake** (Wraps the per-tool methylation calling pipelines whose outputs feed the consensus model) — https://github.com/comprna/METEORE

## Examples

```
python combination_model_train.py -c 2 megalodon_per_read_scores.txt deepsignal_per_read_scores.txt output_models/  # produces rf_model_max_depth_3_n_estimator_10
```

## Evaluation signals

- The saved model file is named with the optimised hyperparameters, rf_model_max_depth_3_n_estimator_10 (n_estimators=10, max_depth=3)
- Applied to mixture dataset 2, the random forest consensus must achieve lower RMSE in per-site methylation frequency than either Megalodon or DeepSignal alone
- The consensus must increase the proportion of sites predicted within the expected 10% window of the known mixture proportion, at intermediate and high methylation sets
- Pearson r, r^2, RMSE and ROC/precision-recall curves from tenfold cross-validation should be reported for the consensus vs individual tools
- Input per-read score files must conform to the ID/Chr/Pos/Strand/Score format and merge cleanly on read ID and position before training

## Limitations

- Combining multiple tools can be impractical for routine analyses due to running times of methods without GPU support (Megalodon and DeepSignal both rely on GPUs to be time efficient)
- The optimised model was tuned for 5mC at CpG dinucleotides on R9.4.1 nanopore data; generalisation to other modifications or chemistries is not established here
- Control accuracy depends on methylation control quality: the M.SssI treatment used to generate the CpG-methylated control only has 95% efficiency, which limits expected agreement with methylation frequency

## Evidence

- [readme] random forest and multiple linear regression that combine the outputs from the tools: "new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools."
- [discussion] consensus motivated by low inter-tool agreement: "The observed low agreements motivated us to propose a consensus approach, METEORE, aiming to ameliorate the deficiencies from some methods"
- [other] train combination model command: [evidence span withheld — anchor status: not_found]
- [readme] per-tool score truncation option: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [methods] METEORE repo installation: "git clone https://github.com/comprna/METEORE.git"

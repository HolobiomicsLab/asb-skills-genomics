---
name: methylation-frequency-regression-modeling
description: Use when you have per-read methylation score files (ID/Chr/Pos/Strand/Score)
  from two or more nanopore 5mC tools, such as Megalodon and DeepSignal, and want
  a consensus per-site CpG methylation frequency that outperforms the individual tools.
when_to_use_negative:
- When you only have fast5 signal files and need to generate the per-read tool scores
  first — this skill assumes ID/Chr/Pos/Strand/Score score tables already exist from
  methylation-calling pipelines (Snakemake pipelines for Nanopolish, DeepSignal, Tombo,
  Guppy, Megalodon)
- 'When a single tool is sufficient: Megalodon alone provided overall the best performance,
  so consensus modeling is only warranted when you need accuracy beyond the best individual
  tool and can afford running multiple tools'
- When there is no training data with known expected methylation frequencies — both
  the random forest and the RidgeCV regression require a controlled mixture dataset
  for training
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0091
tools:
- name: METEORE
  role: Provides the consensus random forest (combination_model_train.py, combination_model_prediction.py)
    and multiple linear regression (meteore_reg.py) models, plus prediction_to_mod_frequency.py
    for per-site conversion
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Source of per-read methylation scores for one of the two combined tools; the
    strongest individual performer in the benchmark
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Source of per-read methylation scores for the second combined tool; its combination
    with Megalodon struck a good balance for accuracy and running times
  repo: https://github.com/bioinfomaticsCSU/deepsignal
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:9a73fc03235f12119c5f220b2b5e038ed11330d0f1c45ffa5d250932d5f5bdfe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepSignal
  - METEORE
  - Megalodon
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# methylation-frequency-regression-modeling

## Summary

Trains METEORE's consensus models — an optimized random forest (n_estimators=10, max_depth=3) and a RidgeCV multiple linear regression — on matched per-read methylation scores from Megalodon and DeepSignal, then applies them to held-out mixture data to produce per-site CpG methylation frequencies that are more accurate than either individual tool.

## When to use

Use when you have per-read methylation score files (ID/Chr/Pos/Strand/Score) from two or more nanopore 5mC tools, such as Megalodon and DeepSignal, and want a consensus per-site CpG methylation frequency that outperforms the individual tools.

## When NOT to use

- When you only have fast5 signal files and need to generate the per-read tool scores first — this skill assumes ID/Chr/Pos/Strand/Score score tables already exist from methylation-calling pipelines (Snakemake pipelines for Nanopolish, DeepSignal, Tombo, Guppy, Megalodon)
- When a single tool is sufficient: Megalodon alone provided overall the best performance, so consensus modeling is only warranted when you need accuracy beyond the best individual tool and can afford running multiple tools
- When there is no training data with known expected methylation frequencies — both the random forest and the RidgeCV regression require a controlled mixture dataset for training

## Inputs

- Per-read methylation score files from Megalodon with ID/Chr/Pos/Strand/Score columns
- Per-read methylation score files from DeepSignal with ID/Chr/Pos/Strand/Score columns
- METEORE training per-read scores from a controlled mixture dataset (e.g. mixture dataset 1 from the figshare deposit https://figshare.com/articles/dataset/Per-read_scores_from_different_nanopore_methylation_tools_in_controlled_mixture_dataset_1/15081264)
- Per-read scores from an independent mixture dataset (dataset 2) for evaluation

## Outputs

- Trained random forest model file rf_model_max_depth_3_n_estimator_10 in the output_models directory
- Trained RidgeCV multiple-linear-regression model with truncation, scaling, and filter parameters recorded
- Per-read consensus predictions with Prob_methylation values from combination_model_prediction.py
- Per-site methylation frequency files (augmented BED format with chromosome, start, end, coverage, methylation frequency, strandedness)
- Evaluation metrics: Pearson r, r^2, RMSE, ROC and precision-recall curves from tenfold cross-validation, and proportion of sites within the expected 10% window

## How to apply

Format per-read scores from each tool as ID/Chr/Pos/Strand/Score tables and train the optimized random forest with METEORE's combination_model_train.py using -c 2 on a controlled-mixture training set (e.g. mixture dataset 1 from the figshare deposit), saving the rf_model_max_depth_3_n_estimator_10 variant. Separately train the RidgeCV regression with meteore_reg.py on the same data, applying per-tool score truncation (--trunc_min/--trunc_max to restrict the useful range of log difference scores), MinMax 0-1 scaling, and the --filter proportion of reads removed around the mean score. Apply both trained models with combination_model_prediction.py (-m optimized) to per-read scores from an independent mixture (dataset 2), then convert per-read predictions to per-site methylation frequency with prediction_to_mod_frequency.py using a threshold of 0.5 on Prob_methylation. Evaluate per-site predictions against expected mixture frequencies with Pearson r, r^2, RMSE, ROC and precision-recall curves from tenfold cross-validation, and the proportion of sites within the expected 10% window of the known mixture proportion. The rationale is that individual tools trade off false positives and false negatives, so combining their predictions ameliorates individual deficiencies.

## Related tools

- **METEORE** (Provides the consensus random forest (combination_model_train.py, combination_model_prediction.py) and multiple linear regression (meteore_reg.py) models, plus prediction_to_mod_frequency.py for per-site conversion) — https://github.com/comprna/METEORE
- **Megalodon** (Source of per-read methylation scores for one of the two combined tools; the strongest individual performer in the benchmark) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Source of per-read methylation scores for the second combined tool; its combination with Megalodon struck a good balance for accuracy and running times) — https://github.com/bioinfomaticsCSU/deepsignal

## Examples

```
python scripts/combination_model_train.py -c 2 megalodon_deepsignal_mixture1_scores.txt output_models --model RF --rf_model_max_depth 3 --rf_model_n_estimator 10
```

## Evaluation signals

- The METEORE random forest consensus must achieve lower RMSE against expected methylation frequency than either Megalodon or DeepSignal alone on mixture dataset 2
- The consensus improves the proportion of sites predicted within the expected 10% window of the known mixture proportion, especially at intermediate and high methylation sets
- Training produces the expected model artifact named rf_model_max_depth_3_n_estimator_10 (n_estimators=10, max_depth=3) in the output_models directory
- Per-read to per-site conversion applies threshold 0.5 on Prob_methylation before computing per-site frequencies
- Tenfold cross-validation ROC and precision-recall curves are generated and per-site predictions show reduced dispersion relative to expected methylation frequency values

## Limitations

- The combination of Megalodon and DeepSignal would require a GPU for efficiency, and combining multiple tools might be impractical for routine analyses due to the running times for methods without GPU support
- The M.SssI treatment used to generate the CpG-methylated control only has ~95% efficiency, which degrades agreement with expected methylation frequency
- Regression training depends on per-tool truncation (--trunc_min/--trunc_max), MinMax scaling, and --filter settings; disabling scaling (--no_scaling) or mis-setting the filter changes model behavior

## Evidence

- [abstract] METEORE consensus models combine per-read scores: "including a consensus approach, METEORE ( https://github.com/comprna/METEORE ), based on the combination of the predictions from two or more tools"
- [readme] README describes RF and regression consensus models: "new predictive models (random forest and multiple linear regression) that combine the outputs from the tools to produce a consensus prediction with higher accuracy than the individual tools."
- [methods] Regression-specific truncation parameters: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [methods] Filter option definition: "--filter FLOAT       Percentage of total reads to remove around mean score, default -1 (disabled)"
- [discussion] Megalodon+DeepSignal GPU requirement: "The combination of Megalodon and DeepSignal struck a good balance for accuracy and running times but would require a GPU for efficiency."
- [discussion] Consensus may be impractical for routine use: "Although combining multiple tools improved the accuracy even further, it might be impractical for routine analyses, due to the running times for some methods without GPU support"

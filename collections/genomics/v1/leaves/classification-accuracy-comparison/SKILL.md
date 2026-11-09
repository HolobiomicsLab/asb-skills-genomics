---
name: classification-accuracy-comparison
description: Use when a trained classifier (e.g., a CNN splice-site model) must be
  evaluated on a fixed test dataset and its accuracy compared and ranked against multiple
  published baselines under the same data conditions.
when_to_use_negative:
- The task is model training or architecture search rather than evaluation of already-trained
  models on a fixed test set.
- The datasets are imbalanced (7500 true / 2500 false) and the imbalanced-mode evaluation,
  not the balanced accuracy comparison, is required.
- Input sequences are not the constructed splice-site datasets with sequence length
  400 and cannot be one-hot encoded to that format.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0625
tools:
- name: CNNSplice
  role: Model under evaluation; run in testing mode on balanced acceptor/donor datasets
    to obtain accuracy metrics
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: GeneSplicer
  role: Baseline splice-site predictor evaluated on the same balanced test datasets
    with author-provided models
- name: SpliceRover
  role: CNN-based baseline splice-site predictor compared against CNNSplice accuracy
- name: Splice2Deep
  role: Deep CNN ensemble baseline splice-site predictor compared against CNNSplice
    accuracy
- name: SpliceFinder
  role: CNN baseline splice-site predictor compared against CNNSplice accuracy
- name: DeepSplicer
  role: CNN baseline (five-fold cross-validation, three convolutional layers) compared
    against CNNSplice accuracy
- name: StratifiedKFold
  role: Machine learning module used for the five-fold cross-validation model selection
    behind the evaluated models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:cb1e07738cc99a67f3806dda61c8f5b0fa134ba21a9bc861560e0e73061f682e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - DeepSplicer
  - GeneSplicer
  - Splice2Deep
  - SpliceFinder
  - SpliceRover
  - StratifiedKFold
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# classification-accuracy-comparison

## Summary

Benchmark a new splice-site prediction model's classification accuracy against state-of-the-art baselines on identical balanced test datasets, then rank all methods by accuracy (highest = rank 1) so the new model's relative position (Top-1/Top-2) can be statistically confirmed.

## When to use

Use when a trained classifier (e.g., a CNN splice-site model) must be evaluated on a fixed test dataset and its accuracy compared and ranked against multiple published baselines under the same data conditions.

## When NOT to use

- The task is model training or architecture search rather than evaluation of already-trained models on a fixed test set.
- The datasets are imbalanced (7500 true / 2500 false) and the imbalanced-mode evaluation, not the balanced accuracy comparison, is required.
- Input sequences are not the constructed splice-site datasets with sequence length 400 and cannot be one-hot encoded to that format.

## Inputs

- Trained model weights (.h5) for CNNSplice
- Balanced acceptor and donor test datasets (5000 true / 5000 false sites, sequence length 400, one-hot encoded) for five organisms
- Author-provided baseline models for GeneSplicer, SpliceRover, Splice2Deep, SpliceFinder, and DeepSplicer

## Outputs

- .txt evaluation metric files stored in the log directory
- Comparison table of accuracy values for CNNSplice Model 1 and all five baselines per organism and site type
- Accuracy-to-rank conversion per method with Friedman-test significance assessment

## How to apply

Run the trained model (CNNSplice .h5 model files) in testing mode on the balanced acceptor/donor test datasets (5000 true and 5000 false sites, one-hot encoded, sequence length 400) using `python test.py -n "output_name" -m "balanced"`, which writes evaluation metrics as .txt files to the log directory. Evaluate each baseline tool (GeneSplicer, SpliceRover, Splice2Deep, SpliceFinder, DeepSplicer) on the exact same test datasets using the models provided by the original authors to keep the comparison fair. Compile the accuracy values for all methods into a comparison table per organism (Homo sapiens, Arabidopsis thaliana, Oryza sativa japonica, Drosophila melanogaster, Caenorhabditis elegans) and per site type (acceptor/donor). Convert accuracy values to ranks, assigning rank 1 to the highest accuracy and the lowest rank to the weakest model, and apply a Friedman test on these ranks to assess statistical significance of performance differences.

## Related tools

- **CNNSplice** (Model under evaluation; run in testing mode on balanced acceptor/donor datasets to obtain accuracy metrics) — https://github.com/OluwadareLab/CNNSplice.git
- **GeneSplicer** (Baseline splice-site predictor evaluated on the same balanced test datasets with author-provided models)
- **SpliceRover** (CNN-based baseline splice-site predictor compared against CNNSplice accuracy)
- **Splice2Deep** (Deep CNN ensemble baseline splice-site predictor compared against CNNSplice accuracy)
- **SpliceFinder** (CNN baseline splice-site predictor compared against CNNSplice accuracy)
- **DeepSplicer** (CNN baseline (five-fold cross-validation, three convolutional layers) compared against CNNSplice accuracy)
- **StratifiedKFold** (Machine learning module used for the five-fold cross-validation model selection behind the evaluated models)

## Examples

```
python test.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Reproduced balanced donor/acceptor accuracies fall in the reported ~94-98% range (e.g., balanced acceptor 96.94% for Homo sapiens, 96.26% for Arabidopsis thaliana, 96.85% for Oryza sativa japonica, 97.00% for Caenorhabditis elegans).
- CNNSplice Model 1 ranks 1st or 2nd (Top-1/Top-2) when all six methods are ranked by accuracy with highest = rank 1.
- All methods are evaluated on identical balanced test datasets (5000 true / 5000 false) with baseline tools run using author-provided models.
- Evaluation metric .txt files are present in the log directory after each test.py run.
- Friedman test on the rank-converted accuracy values supports the significance of performance differences.

## Limitations

- The constructed dataset permits a sequence length of 400 only; inputs outside this constraint are out of scope.
- Comparisons depend on the specific balanced data distribution; some existing methods are not robust to the data distribution of acceptor and donor SS used for training and testing.
- Existing DL-based SS prediction tools maintain inconsistent prediction accuracy across a wide range of organisms, so rankings may not transfer to new organisms without a generalizability test.

## Evidence

- [results] Run CNNSplice in testing mode: [evidence span withheld — anchor status: not_found]
- [results] fair baseline comparison: [evidence span withheld — anchor status: not_found]
- [results] rank conversion for Friedman test: "accuracy values were converted to ranks, where the highest accuracy value was assigned a rank of 1, and the lowest accuracy value received a rank equivalent to the number of models being compared."
- [readme] testing usage: "For Testing, use `python test.py -n "output_name" -m mode("balanced" or "imbalanced")`"
- [results] balanced dataset composition: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments"
- [readme] sequence length constraint: "Our constructed dataset permits a **Sequence Length of 400**"

---
name: cnn-model-inference-evaluation
description: Use when testing a pretrained CNNSplice model (Model 1 .h5 weights from
  one organism) on a different organism's constructed splice-site dataset to measure
  cross-organism generalization accuracy.
when_to_use_negative:
- 'The goal is to train new CNNSplice models from scratch rather than evaluate pretrained
  .h5 weights (use the training workflow: python train.py instead).'
- The input genomic data cannot be represented in the CNNSplice constructed-dataset
  format with a fixed sequence length of 400.
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3512
tools:
- name: CNNSplice
  role: Provides the CNN models, one-hot-encoding preprocessing, and test.py script
    used to run inference and produce evaluation metrics on cross-organism splice-site
    data.
  repo: https://github.com/OluwadareLab/CNNSplice.git
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:51fa34d18de37bd8426399bec6a2ad5ea2c684eeeebdb23b35c4bea887a3c893
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cnn-model-inference-evaluation

## Summary

Run inference and compute evaluation metrics with pretrained CNNSplice .h5 CNN models (e.g. the Model 1 balanced acceptor model) on new or cross-organism splice-site datasets, as in CNNSplice's generality test.

## When to use

Use when testing a pretrained CNNSplice model (Model 1 .h5 weights from one organism) on a different organism's constructed splice-site dataset to measure cross-organism generalization accuracy.

## When NOT to use

- The goal is to train new CNNSplice models from scratch rather than evaluate pretrained .h5 weights (use the training workflow: python train.py instead).
- The input genomic data cannot be represented in the CNNSplice constructed-dataset format with a fixed sequence length of 400.

## Inputs

- Cross-organism splice-site test dataset in CNNSplice constructed-dataset format (sequence length 400)
- Pretrained CNNSplice model weights (.h5 file, e.g. Model 1 trained on the source organism)
- Mode string selecting dataset type ('balanced' or 'imbalanced')

## Outputs

- Evaluation metrics results written to a .txt file in the log directory
- Cross-organism splice-site prediction accuracies for comparison with SpliceFinder/DeepSplicer baselines

## How to apply

Load the cross-organism test dataset (e.g. Drosophila melanogaster acceptor sites when the model was trained on Caenorhabditis elegans), respecting the fixed Sequence Length of 400 imposed by the constructed dataset format. Transform the genomic sequence inputs by one-hot encoding before feeding the CNN. Run the CNNSplice testing script with the pretrained Model 1 weights in either 'balanced' or 'imbalanced' mode via `python test.py -n "output_name" -m mode`, then collect the evaluation metrics written to a .txt file in the log directory (create the log directory beforehand). Judge success by whether the .txt metrics file is produced and whether cross-organism accuracy is consistent with the paper's generality-test results (94.77% for CNNSplice vs 86.60% SpliceFinder and 91.97% DeepSplicer on the C. elegans-to-D. melanogaster balanced acceptor comparison).

## Related tools

- **CNNSplice** (Provides the CNN models, one-hot-encoding preprocessing, and test.py script used to run inference and produce evaluation metrics on cross-organism splice-site data.) — https://github.com/OluwadareLab/CNNSplice.git

## Examples

```
python test.py -n "output_name" -m "balanced"
```

## Evaluation signals

- An evaluation-metrics .txt file is produced in the log directory after running test.py.
- Cross-organism balanced acceptor accuracy is high and consistent with the paper (94.77% for C. elegans-trained Model 1 tested on D. melanogaster, above SpliceFinder's 86.60% and DeepSplicer's 91.97%).
- All input sequences are one-hot encoded and conform to the fixed sequence length of 400 before inference.
- The mode argument matches the dataset distribution actually used ('balanced' vs 'imbalanced').

## Limitations

- The constructed dataset permits a fixed sequence length of 400; inputs not in this format require re-construction.
- A log directory must exist for the .txt output files, otherwise the evaluation metrics cannot be stored.
- Generality-test accuracy depends on the model/mode chosen; Model 1, Model 5, and Model 3 are only reported as top performers for balanced acceptor datasets.

## Evidence

- [abstract] our generality test shows CNNSplice's model ability to predict and annotate splice sites in new or poorly trained genome datasets: "our generality test shows CNNSplice's model ability to predict and annotate splice sites in new or poorly trained genome datasets"
- [readme] testing command: "For Testing, use `python test.py -n "output_name" -m mode("balanced" or "imbalanced")`"
- [readme] metrics output: ".txt: The output files (.txt) containig the evaluation metrics results is stored in the log directory."
- [methods] one-hot encoding requirement: "Genomic sequence input data should be transfomed using one-hot encoding."
- [results] cross-organism accuracy: [evidence span withheld — anchor status: not_found]

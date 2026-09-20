---
name: neural-network-dropout-regularization
description: Use when building or re-implementing the CNNSplice Model 1-5 architectures
  (Sections 2.3/Table 2) and you need the post-convolutional dropout regularization
  and training settings (Adam, lr 1e-4, cross-entropy, Softmax) for balanced/imbalanced
  splice site datasets.
when_to_use_negative:
- Input genomic sequences are not one-hot encoded or do not conform to the fixed Sequence
  Length of 400 constraint — the models will not accept them
- The goal is a non-splice-site classification task with different class structure;
  the Softmax head and cross-entropy loss are tuned for binary true/false splice site
  prediction
- Only a pretrained model is needed for inference — use test.py with an existing .h5
  file rather than rebuilding the training stack
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: CNNSplice
  role: Package containing the five model architectures with the dropout/Adam/Softmax
    training configuration, invoked via train.py
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: Python
  role: Implementation language in which the CNN models are defined, compiled, and
    trained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:de8f9031e78b751c4642cfd36586aaeaf10ed9d8ccac6cc46571de1d3a2cf218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - Python
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# neural-network-dropout-regularization

## Summary

Configure the output-layer and training stack of the five CNNSplice CNN models: a flatten layer, a 100-neuron fully connected ReLU layer with 30% dropout, and a Softmax output, compiled with Adam (learning rate 1e-4) and cross-entropy loss to avoid overfitting on splice site data.

## When to use

Use when building or re-implementing the CNNSplice Model 1-5 architectures (Sections 2.3/Table 2) and you need the post-convolutional dropout regularization and training settings (Adam, lr 1e-4, cross-entropy, Softmax) for balanced/imbalanced splice site datasets.

## When NOT to use

- Input genomic sequences are not one-hot encoded or do not conform to the fixed Sequence Length of 400 constraint — the models will not accept them
- The goal is a non-splice-site classification task with different class structure; the Softmax head and cross-entropy loss are tuned for binary true/false splice site prediction
- Only a pretrained model is needed for inference — use test.py with an existing .h5 file rather than rebuilding the training stack

## Inputs

- One-hot encoded genomic splice site sequences of length 400
- Balanced (5000 true / 5000 false) or imbalanced (7500 true / 2500 false) acceptor and donor training datasets
- Base convolutional stack definitions per Section 2.3/Table 2 (Models 1-3)

## Outputs

- Five compiled Keras-style CNN models (Models 1-5) with flatten + 100-neuron FC ReLU + 30% dropout + Softmax heads
- Trained .h5 model files produced by train.py
- Log-directory .txt files containing evaluation metrics

## How to apply

After defining each base convolutional stack (Model 1: three 1-D conv layers; Model 2: two 1-D conv layers; Model 3: three paired convolutional blocks), append a flatten layer followed by a fully connected ReLU layer of 100 neurons regularized with a 30% dropout layer to avoid overfitting. Add a Softmax output layer that transforms the prediction output to normalized probability over true/false splice site classes. Compile every model with the Adam optimizer at a learning rate of 1e-4 and cross-entropy loss. Construct the re-weighted variants by initializing Model 4 with the mean weight across Model 1 and Model 2, and Model 5 with the mean weight across Model 1 and Model 3. Input to each model is one-hot encoded genomic sequences of length 400, and the model name/mode interface ('balanced' or 'imbalanced') must match the train.py usage.

## Related tools

- **CNNSplice** (Package containing the five model architectures with the dropout/Adam/Softmax training configuration, invoked via train.py) — https://github.com/OluwadareLab/CNNSplice.git
- **Python** (Implementation language in which the CNN models are defined, compiled, and trained)

## Examples

```
python train.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Instantiating all five architectures yields layer configurations matching the published specification in Section 2.3 and Table 2
- Each model contains a 30% dropout layer immediately following the 100-neuron fully connected ReLU layer and a Softmax output layer
- Compilation settings report Adam optimizer with learning rate 1e-4 and cross-entropy loss for every model
- Models 4 and 5 verify as mean-weight initializations across Model 1/Model 2 and Model 1/Model 3 respectively
- Each compiled model accepts one-hot encoded input of length 400 without shape errors

## Limitations

- The dataset constraint fixes sequence length at 400; sequences of other lengths require retraining or re-architecting
- Only balanced ('balanced') and imbalanced ('imbalanced') dataset modes are supported through the train.py interface
- No changelog is available for the repository, so model configuration drift across versions cannot be tracked

## Evidence

- [results] Each CNNSplice model implements a 30% dropout layer to avoid overfitting and is trained using the Adam optimizer with a learning rate of 1e-4 and cross-entropy loss: [evidence span withheld — anchor status: not_found]
- [results] This involves setting a new weight obtained from the mean weight across a set weight from Model 1 and Model 2 to the model.: "This involves setting a new weight obtained from the mean weight across a set weight from Model 1 and Model 2 to the model."
- [methods] Genomic sequence input data should be transfomed using one-hot encoding.: "Genomic sequence input data should be transfomed using one-hot encoding."
- [readme] Dataset sequence length is 400.: "Dataset sequence length is 400."
- [readme] To train, type in the terminal `python train.py -n "model_name" -m mode `: "Usage: To train, type in the terminal `python train.py -n "model_name" -m mode `"

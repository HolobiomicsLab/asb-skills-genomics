---
name: convolutional-neural-network-architecture-implementation
description: Use when reimplementing or extending Nanoident's multi-column CNN for
  classifying methylation types (4mC, 5mC, 6mA) or fine-mapping methylated base positions
  from five Nanopore-derived motif features.
when_to_use_negative:
- Input is raw nanopore signal or modBAM rather than the preprocessed five-feature
  motif data in pkl format
- The task is de novo methylation motif discovery rather than typing or fine mapping
  of already known motifs
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Nanoident
  role: Multi-column CNN model being reconstructed; provides env/freeze.yml, model
    code, and pretrained best_model/Nanoident.pt
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: Language used in PyTorch to define and train the multi-scale convolutional
    architecture
- name: conda
  role: Generates the Python training environment directly from Nanoident/env/freeze.yml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:0b95ce8cb3c5239691981721a26f7e62048d67ecab796619d63f233e3ccd90eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Python
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# convolutional-neural-network-architecture-implementation

## Summary

Reconstruct Nanoident's multi-scale convolutional architecture in PyTorch: three columns of depthwise-separable convolution (DSC) blocks with large (kernel 4), medium (kernels 3 and 2), and small (three stacked kernel-2) receptive fields, whose outputs are concatenated, adaptively weighted, and fused with a 1x1 pointwise convolution for bacterial methylation motif typing and fine mapping from Nanopore features.

## When to use

Use when reimplementing or extending Nanoident's multi-column CNN for classifying methylation types (4mC, 5mC, 6mA) or fine-mapping methylated base positions from five Nanopore-derived motif features.

## When NOT to use

- Input is raw nanopore signal or modBAM rather than the preprocessed five-feature motif data in pkl format
- The task is de novo methylation motif discovery rather than typing or fine mapping of already known motifs

## Inputs

- pkl-format feature files (converted from R via data_from_R_to_python.ipynb) containing five features per targeted motif sequence extracted from Nanopore raw signals and basecalling results
- Nanoident/env/freeze.yml conda environment specification

## Outputs

- Trained multi-column CNN model (e.g. best_model/Nanoident.pt) for methylation typing and fine mapping
- Architecture with exactly 14,319 trainable parameters

## How to apply

Set up the Python training environment with conda using Nanoident/env/freeze.yml, then define the three convolutional columns in PyTorch: the Large column as a single depthwise-separable convolution with kernel size 4, the Medium column as DSC convolutions with kernel sizes 3 and 2, and the Small column as three stacked kernel-size-2 DSC blocks with residual connections around them. Concatenate the outputs of the L, M, and S columns and pass them through a learned weight block that assigns adaptive weights to the column-scale feature channels, focusing attention on the more important features. Fuse the weighted multi-scale features with a pointwise (1x1) convolution layer. Instantiate the full model and count trainable parameters to confirm the total equals 14,319, then wire the class into ./model/train_test.py for training on pkl-format feature data converted from R via data_from_R_to_python.ipynb.

## Related tools

- **Nanoident** (Multi-column CNN model being reconstructed; provides env/freeze.yml, model code, and pretrained best_model/Nanoident.pt) — https://github.com/cz-csu/Nanoident
- **Python** (Language used in PyTorch to define and train the multi-scale convolutional architecture)
- **conda** (Generates the Python training environment directly from Nanoident/env/freeze.yml)

## Examples

```
conda env create -f Nanoident/env/freeze.yml && python ./model/train_test.py
```

## Evaluation signals

- The implemented architecture instantiates without error in PyTorch
- Total trainable parameter count of the instantiated model equals 14,319
- The model class runs end-to-end via python ./model/train_test.py on pkl-format training data
- Three distinct columns (L: kernel 4; M: kernels 3 and 2; S: three stacked kernel-2 blocks) contribute concatenated outputs before the weight block and 1x1 fusion convolution

## Limitations

- Preprocessing is memory constrained: data must be preprocessed in chunks and then merged
- The original nanodisco environment only handled mean features; update.sh is required to handle four additional features (std, time, proportion of reference bases, average quality score) plus base feature
- Architecture parameter count (14,319) must match exactly; deviations indicate incorrect DSC stacking or fusion design

## Evidence

- [methods] workflow column structure: [evidence span withheld — anchor status: not_found]
- [intro] architecture description: "A multi-column convolutional neural network model for typing (the identification of methylation types) and fine mapping (the identification of methylated base positions)"
- [readme] environment setup: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
- [methods] validation script: "python ./model/train_test.py"

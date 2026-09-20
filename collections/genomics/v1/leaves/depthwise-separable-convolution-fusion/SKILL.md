---
name: depthwise-separable-convolution-fusion
description: Use when reimplementing or validating Nanoident's multi-column CNN for
  bacterial methylation motif typing/fine mapping, or when building a compact multi-scale
  feature-fusion block over per-motif nanopore signal features.
when_to_use_negative:
- Input is raw nanopore signal or modBAM files — data must first be preprocessed through
  the nanodisco/update.sh pipeline and converted to pkl format
- Task is motif discovery/enrichment rather than typing or fine mapping of known motifs
  — use the de novo motif enrichment pipeline instead
- You need ONT modified-base pileups in bedMethyl format — modkit, not Nanoident's
  CNN, is the appropriate tool
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0622
tools:
- name: Nanoident
  role: Source framework whose Multi-Scale Convolutional module (L/M/S depthwise-separable
    columns, weight block, 1x1 fusion) is being reconstructed
  repo: https://github.com/cz-csu/Nanoident
- name: Python
  role: PyTorch implementation language for the column definitions, weight block,
    and pointwise fusion layer
- name: conda
  role: Environment manager; creates the training environment directly from Nanoident/env/freeze.yml
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf397
    title: Identifying DNA methylation types and methylated base positions from bacteria
      using nanopore sequencing with multi-scale neural network
  source_content_hash: sha256:cb9d830c2e2a3a2c262aaec3842eced615cc645499cfd06aec07f4b8cd8874e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cz-csu/Nanoident
  tools:
  - Nanoident
  - Python
  - conda
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# depthwise-separable-convolution-fusion

## Summary

Reconstructs Nanoident's Multi-Scale Convolutional module in PyTorch: parallel Large/Medium/Small columns of depthwise-separable convolution blocks whose outputs are concatenated, adaptively weighted, and fused with a pointwise convolution, totaling 14,319 trainable parameters.

## When to use

Use when reimplementing or validating Nanoident's multi-column CNN for bacterial methylation motif typing/fine mapping, or when building a compact multi-scale feature-fusion block over per-motif nanopore signal features.

## When NOT to use

- Input is raw nanopore signal or modBAM files — data must first be preprocessed through the nanodisco/update.sh pipeline and converted to pkl format
- Task is motif discovery/enrichment rather than typing or fine mapping of known motifs — use the de novo motif enrichment pipeline instead
- You need ONT modified-base pileups in bedMethyl format — modkit, not Nanoident's CNN, is the appropriate tool

## Inputs

- pkl-format feature files (five features from nanopore raw signals and basecalling results) produced by data_from_R_to_python.ipynb
- Nanoident/env/freeze.yml conda environment specification
- R-preprocessed training/validation data from Nanoident_traindata.R

## Outputs

- PyTorch model class for the multi-scale convolutional module with exactly 14,319 trainable parameters
- Trained typing and fine-mapping models via ./model/train_test.py
- Weighted multi-scale feature representations (concatenated L/M/S column outputs after weight block and 1x1 fusion)

## How to apply

Set up the Python training environment with conda using Nanoident/env/freeze.yml. In PyTorch, define the Large column as a single depthwise-separable convolution with kernel size 4, the Medium column as depthwise-separable convolutions with kernel sizes 3 and 2, and the Small column as three stacked kernel-size-2 depthwise-separable convolution blocks. Concatenate the three column outputs and pass them through a learned weight block that assigns adaptive weights to each multi-scale feature channel, focusing attention on the more important scale-specific features. Fuse the weighted features with a pointwise (1x1) convolution and add residual connections around the convolution blocks. Instantiate the model, wire it into ./model/train_test.py for training on pkl-format features converted from R, and confirm the total trainable parameter count equals 14,319.

## Related tools

- **Nanoident** (Source framework whose Multi-Scale Convolutional module (L/M/S depthwise-separable columns, weight block, 1x1 fusion) is being reconstructed) — https://github.com/cz-csu/Nanoident
- **Python** (PyTorch implementation language for the column definitions, weight block, and pointwise fusion layer)
- **conda** (Environment manager; creates the training environment directly from Nanoident/env/freeze.yml)

## Examples

```
conda env create -f freeze.yml && python ./model/train_test.py
```

## Evaluation signals

- Implemented architecture instantiates without error in PyTorch
- Total trainable parameter count equals 14,319, matching the reported model size
- Model class runs end-to-end in ./model/train_test.py on pkl-format feature data converted from R
- Fusion path correctly applies the adaptive weight block to concatenated column outputs before the 1x1 convolution
- Residual connections are present around the convolution blocks

## Limitations

- The architecture is designed for motif-level classification using five signal/basecall features; it does not itself perform de novo motif discovery
- Architecture size (14,319 parameters) is validated only for Nanoident's specific feature representation; adding/removing features changes the count
- Memory constraints require chunked preprocessing upstream of training (preprocess in chunks, then merge)

## Evidence

- [methods] column concatenation and weighting: [evidence span withheld — anchor status: not_found]
- [readme] environment setup: "Use Nanoident/env/freeze.yml to generate the environment directly under conda."
- [readme] model purpose: "A multi-column convolutional neural network model for typing (the identification of methylation types) and fine mapping"
- [abstract] five features: "For each targeted motif sequence, Nanoident utilizes five different features, including statistical features extracted from both the nanopore raw signals"

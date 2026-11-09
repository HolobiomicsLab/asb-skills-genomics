---
name: splice-site-classification-model-evaluation
description: Use when reproducing or benchmarking splice site prediction accuracies
  of pre-trained CNN models (e.g., .h5 weights) on acceptor/donor test sets with balanced
  or skewed (3:1) true/false class distributions across multiple organisms.
when_to_use_negative:
- Training new models from scratch — use `python train.py -n "model_name" -m mode`
  instead of the testing workflow
- Sequences whose length differs from 400, since the CNNSplice constructed datasets
  permit a Sequence Length of 400
- Annotation of newly sequenced genomes requiring sliding-window splice site identification
  rather than evaluation of pre-labelled true/false test sets
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: CNNSplice
  role: CNN model suite (Models 1-5) whose testing script evaluates splice site prediction
    accuracy on balanced/imbalanced acceptor and donor datasets
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: Docker
  role: Containerized runtime environment (oluwadarelab/cnnsplice:latest image) in
    which the CNNSplice testing script is executed with the working directory mounted
- name: SpliceFinder
  role: Benchmark method for comparison in cross-organism generality testing (86.60%
    vs CNNSplice's 94.77% on D. melanogaster)
- name: DeepSplicer
  role: Benchmark method for comparison in cross-organism generality testing (91.97%
    vs CNNSplice's 94.77% on D. melanogaster)
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:24b3eb95b1b6a58cb6f45dee8ebfd930b34b0cb933dd3f1c307b37bfc2dca136
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - DeepSplicer
  - Docker
  - SpliceFinder
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# splice-site-classification-model-evaluation

## Summary

Evaluate CNN-based splice site (acceptor/donor) prediction models on balanced (5000 true/5000 false) and imbalanced (7500 true/2500 false) test datasets across five organisms (Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, Caenorhabditis elegans) using CNNSplice's testing script inside its Docker environment.

## When to use

Use when reproducing or benchmarking splice site prediction accuracies of pre-trained CNN models (e.g., .h5 weights) on acceptor/donor test sets with balanced or skewed (3:1) true/false class distributions across multiple organisms.

## When NOT to use

- Training new models from scratch — use `python train.py -n "model_name" -m mode` instead of the testing workflow
- Sequences whose length differs from 400, since the CNNSplice constructed datasets permit a Sequence Length of 400
- Annotation of newly sequenced genomes requiring sliding-window splice site identification rather than evaluation of pre-labelled true/false test sets

## Inputs

- Pre-trained CNNSplice model weights (.h5 files, Models 1-5)
- Balanced acceptor/donor test datasets (5000 true / 5000 false) for five organisms
- Imbalanced acceptor/donor test datasets (7500 true / 2500 false) for five organisms
- Cross-organism (generality) test datasets constructed with fixed sequence length of 400

## Outputs

- .txt evaluation metrics files (prediction accuracy etc.) stored in the log directory
- Reproduced accuracy values per model, mode (balanced/imbalanced), organism, and splice site type
- Cross-organism generality-test metrics for comparison against SpliceFinder and DeepSplicer

## How to apply

Clone the OluwadareLab/CNNSplice repository, pull the oluwadarelab/cnnsplice:latest Docker image, and run the container with the working directory mounted (docker run -v ${PWD}:${PWD} -p 8050:8050 -it oluwadarelab/cnnsplice). Load the pre-trained .h5 model weights and the balanced (5000 true/5000 false) or imbalanced (7500 true/2500 false) acceptor/donor test datasets, noting the fixed Sequence Length of 400. One-hot encode the genomic sequence inputs before inference, then run `python test.py -n "output_name" -m mode` with mode set to 'balanced' or 'imbalanced' for each model/organism combination. Collect the .txt evaluation metrics files written to the log directory (create the log directory first) and compare accuracies against published values, e.g., Model 2 imbalanced donor 96.46% (H. sapiens) and 97.62% (A. thaliana), Model 4 imbalanced donor 95.19% (O. sativa japonica), within roughly one percentage point tolerance.

## Related tools

- **CNNSplice** (CNN model suite (Models 1-5) whose testing script evaluates splice site prediction accuracy on balanced/imbalanced acceptor and donor datasets) — https://github.com/OluwadareLab/CNNSplice.git
- **Docker** (Containerized runtime environment (oluwadarelab/cnnsplice:latest image) in which the CNNSplice testing script is executed with the working directory mounted)
- **SpliceFinder** (Benchmark method for comparison in cross-organism generality testing (86.60% vs CNNSplice's 94.77% on D. melanogaster))
- **DeepSplicer** (Benchmark method for comparison in cross-organism generality testing (91.97% vs CNNSplice's 94.77% on D. melanogaster))

## Examples

```
docker run -v ${PWD}:${PWD} -p 8050:8050 -it oluwadarelab/cnnsplice; python test.py -n "output_name" -m "imbalanced"
```

## Evaluation signals

- Reproduced imbalanced donor accuracies match Model 2: 96.46% (H. sapiens) and 97.62% (A. thaliana), and Model 4: 95.19% (O. sativa japonica) within ~1 percentage point tolerance
- Cross-organism generality test with Model 1 yields ~94.77% on D. melanogaster (vs SpliceFinder 86.60%, DeepSplicer 91.97%)
- A .txt evaluation metrics file is produced in the log directory after each test run
- Input sequences are one-hot encoded and conform to the fixed Sequence Length of 400
- The correct mode string ('balanced' or 'imbalanced') matches the true/false composition of the loaded test dataset (5000/5000 vs 7500/2500)

## Limitations

- The constructed dataset permits a fixed Sequence Length of 400; other lengths are not supported without adaptation
- Imbalanced evaluation uses a specific 3:1 skew (7500 true / 2500 false); other skew ratios are not covered
- A log directory must exist for text file storage, or output will fail
- Some existing DL-based SS prediction tools maintain inconsistent prediction accuracy across a wide range of organisms — accuracy on one organism does not guarantee performance on another (hence the generality test)

## Evidence

- [other] On the imbalanced donor datasets, CNNSplice Model 2 achieved 96.46% and 97.62% prediction accuracy for Homo sapiens and Arabidopsis thaliana respectively, and Model 4 achieved 95.19% accuracy.: [evidence span withheld — anchor status: not_found]
- [results] imbalanced dataset composition: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced"
- [methods] one-hot encoding requirement: "Genomic sequence input data should be transfomed using one-hot encoding."
- [readme] testing usage command: "For Testing, use `python test.py -n "output_name" -m mode("balanced" or "imbalanced") `"
- [results] cross-organism generality result: [evidence span withheld — anchor status: not_found]

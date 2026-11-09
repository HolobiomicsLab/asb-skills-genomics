---
name: imbalanced-dataset-benchmark-testing
description: Use when evaluating splice site prediction models on test sets where
  true:false cases are skewed (e.g. 7500 true / 2500 false), to check accuracy robustness
  rather than only balanced-set performance.
when_to_use_negative:
- When the goal is to assess performance on balanced (5000 true / 5000 false) datasets
  — use the balanced testing mode instead
- When input sequences do not conform to the constructed dataset's Sequence Length
  of 400 constraint
- When the task is de novo splice site annotation of newly sequenced genomes rather
  than benchmarking pre-trained models on fixed test sets
edam_operation: http://edamontology.org/operation_0433
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: CNNSplice
  role: CNN model suite whose pre-trained weights are evaluated on imbalanced acceptor/donor
    test datasets via the test.py script
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: Docker
  role: Container runtime used to pull and run the oluwadarelab/cnnsplice:latest image
    with the working directory mounted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:a2c631cfd5607c9a2cdaac1222652148e111f8ed42308fb057f9e6f5733abcd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OluwadareLab/CNNSplice.git
  tools:
  - CNNSplice
  - Docker
  edam_operation: http://edamontology.org/operation_0433
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# imbalanced-dataset-benchmark-testing

## Summary

Benchmark pre-trained CNNSplice CNN models on imbalanced (3:1 true:false) acceptor and donor splice site test datasets across five organisms to verify that prediction accuracy is robust to skewed class distributions.

## When to use

Use when evaluating splice site prediction models on test sets where true:false cases are skewed (e.g. 7500 true / 2500 false), to check accuracy robustness rather than only balanced-set performance.

## When NOT to use

- When the goal is to assess performance on balanced (5000 true / 5000 false) datasets — use the balanced testing mode instead
- When input sequences do not conform to the constructed dataset's Sequence Length of 400 constraint
- When the task is de novo splice site annotation of newly sequenced genomes rather than benchmarking pre-trained models on fixed test sets

## Inputs

- Pre-trained CNNSplice model weights (.h5 files) for selected architectures (e.g. Model 2, Model 4)
- Imbalanced splice site test datasets (7500 true / 2500 false) for acceptor and donor sites across five organisms (H. sapiens, O. sativa japonica, A. thaliana, D. melanogaster, C. elegans)
- Genomic sequence input data of fixed sequence length 400

## Outputs

- .txt evaluation metrics files containing prediction accuracy and related metrics, stored in the log directory
- Reproduced accuracy values per organism per model for comparison against reported benchmarks (e.g. 96.46%, 97.62%, 95.19%)

## How to apply

Set up the CNNSplice Docker environment by pulling oluwadarelab/cnnsplice:latest and running the container with the working directory mounted (docker run -v ${PWD}:${PWD} -p 8050:8050 -it oluwadarelab/cnnsplice). Load the pre-trained model weights (.h5 files) and the imbalanced datasets constructed with 7500 true and 2500 false acceptor and donor sites for Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, and Caenorhabditis elegans, respecting the fixed sequence length of 400. Transform the genomic sequence inputs with one-hot encoding as required by CNNSplice, then run the testing script in imbalanced mode (python test.py -n 'output_name' -m 'imbalanced') for each selected model (e.g. Model 2 and Model 4) per organism. Collect the .txt evaluation metrics files from the log directory and compare reproduced accuracies against reported values — Model 2 imbalanced donor 96.46% (H. sapiens) and 97.62% (A. thaliana), Model 4 imbalanced donor 95.19% (O. sativa japonica) — within approximately one percentage point tolerance.

## Related tools

- **CNNSplice** (CNN model suite whose pre-trained weights are evaluated on imbalanced acceptor/donor test datasets via the test.py script) — https://github.com/OluwadareLab/CNNSplice.git
- **Docker** (Container runtime used to pull and run the oluwadarelab/cnnsplice:latest image with the working directory mounted)

## Examples

```
docker run -v ${PWD}:${PWD} -p 8050:8050 -it oluwadarelab/cnnsplice; python test.py -n "output_name" -m "imbalanced"
```

## Evaluation signals

- Reproduced imbalanced donor accuracy for Model 2 on H. sapiens falls within ~1 percentage point of 96.46%
- Reproduced imbalanced donor accuracy for Model 2 on A. thaliana falls within ~1 percentage point of 97.62%
- Reproduced imbalanced donor accuracy for Model 4 on O. sativa japonica falls within ~1 percentage point of 95.19%
- Test run completed with '-m imbalanced' mode and produced .txt metrics files in the log directory
- Input sequences were one-hot encoded and conform to the sequence length of 400 constraint

## Limitations

- The constructed dataset permits a fixed Sequence Length of 400; sequences of other lengths are not supported
- Imbalanced experiments use a specific 3:1 skew (7500 true / 2500 false); other skew ratios are outside the validated scope
- A log directory must exist for text file storage, and genomic sequence input data must be one-hot encoded before testing

## Evidence

- [other] On the imbalanced donor datasets, CNNSplice Model 2 achieved 96.46% and 97.62% prediction accuracy for Homo sapiens and Arabidopsis thaliana respectively, and Model 4 achieved 95.19% accuracy.: [evidence span withheld — anchor status: not_found]
- [results] imbalanced experiments used 7500 true and 2500 false: "we constructed 5000 true and 5000 false acceptor and donor datasets for the balanced experiments and 7500 true and 2500 false acceptor and donor datasets for the imbalanced"
- [readme] one-hot encoding requirement and sequence length: "Genomic sequence input data should be transfomed using one-hot encoding."
- [methods] sequence length 400 constraint: "Our constructed dataset permits a **Sequence Length of 400**"
- [readme] docker run command: "Run the CNNSplice container and mount the present working directory to the container using `docker run -v ${PWD}:${PWD}  -p 8050:8050 -it oluwadarelab/cnnsplice`."

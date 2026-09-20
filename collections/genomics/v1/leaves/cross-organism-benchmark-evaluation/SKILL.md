---
name: cross-organism-benchmark-evaluation
description: Use when comparing a splice-site predictor's accuracy against Splice2Deep,
  GeneSplicer, SpliceRover, SpliceFinder, or DeepSplicer on balanced (5000 true/5000
  false) or imbalanced (7500/2500) acceptor/donor datasets across Homo sapiens, Oryza
  sativa japonica, Arabidopsis thaliana, Drosophila.
when_to_use_negative:
- Your sequences are not 400 nt in length — the constructed dataset 'permits a Sequence
  Length of 400', so other window sizes require re-engineering
- Your goal is to train new models rather than evaluate existing ones — use the training
  workflow (`train.py`) instead of this benchmark/testing skill
- You only have one organism or one site type — the ranking-vs-baselines comparison
  is designed for the five-organism, acceptor-and-donor benchmark setting
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: CNNSplice
  role: Model under test; run in testing mode per organism and site type to produce
    evaluation metrics
  repo: https://github.com/OluwadareLab/CNNSplice.git
- name: GeneSplicer
  role: Baseline evaluated on the same test datasets with author-provided models
- name: SpliceRover
  role: Baseline evaluated on the same test datasets with author-provided models
- name: Splice2Deep
  role: Baseline evaluated on the same test datasets with author-provided models
- name: SpliceFinder
  role: Baseline evaluated on the same test datasets with author-provided models
- name: DeepSplicer
  role: Baseline evaluated on the same test datasets with author-provided models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.05.031
    title: 'CNNSplice: Robust models for splice site prediction using convolutional
      neural networks'
  source_content_hash: sha256:0efc62c49e57c95fd21ccaaa20e597f367e95898dbbb9c83048bfb0b1e239c47
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-organism-benchmark-evaluation

## Summary

Benchmark a splice-site prediction model (e.g., CNNSplice Model 1) against five state-of-the-art baselines (Splice2Deep, GeneSplicer, SpliceRover, SpliceFinder, DeepSplicer) on the same balanced and imbalanced acceptor/donor test datasets from five organisms, ranking methods by accuracy.

## When to use

Use when comparing a splice-site predictor's accuracy against Splice2Deep, GeneSplicer, SpliceRover, SpliceFinder, or DeepSplicer on balanced (5000 true/5000 false) or imbalanced (7500/2500) acceptor/donor datasets across Homo sapiens, Oryza sativa japonica, Arabidopsis thaliana, Drosophila melanogaster, or Caenorhabditis elegans.

## When NOT to use

- Your sequences are not 400 nt in length — the constructed dataset 'permits a Sequence Length of 400', so other window sizes require re-engineering
- Your goal is to train new models rather than evaluate existing ones — use the training workflow (`train.py`) instead of this benchmark/testing skill
- You only have one organism or one site type — the ranking-vs-baselines comparison is designed for the five-organism, acceptor-and-donor benchmark setting

## Inputs

- Trained CNNSplice model files (.h5) for balanced or imbalanced mode
- Balanced (5000 true / 5000 false) acceptor and donor test datasets for the five organisms
- Imbalanced (7500 true / 2500 false) acceptor and donor test datasets for the five organisms
- Author-provided baseline models (GeneSplicer, SpliceRover, Splice2Deep, SpliceFinder, DeepSplicer)
- One-hot encoded genomic sequences of length 400

## Outputs

- Evaluation metrics .txt files in the log directory per organism/site-type/mode
- Comparison table of accuracy values for CNNSplice Model 1 and the five baselines
- Ranking of methods by accuracy (rank 1 = highest) and Friedman test result

## How to apply

Load the trained CNNSplice .h5 model files and the organism-specific acceptor/donor test datasets, one-hot encoding the genomic sequences (fixed sequence length 400) as required by the README. Run CNNSplice in testing mode per organism and site type with `python test.py -n <output_name> -m balanced` (or `imbalanced`), which writes evaluation metrics as .txt files to the log directory. Evaluate the five baselines on the same test datasets using the author-provided models, as the paper compared methods 'fairly' under this condition. Compile accuracies into a comparison table, then convert accuracy values to ranks (highest accuracy = rank 1, lowest = rank equal to the number of models compared) for a Friedman statistical significance test. Validation: CNNSplice Model 1's balanced acceptor accuracies were 96.94% (H. sapiens), 96.26% (A. thaliana), 96.85% (O. sativa japonica), and 97.00% (C. elegans), so reproduced values falling in the ~94-98% range and ranking 1st-2nd indicate correct application.

## Related tools

- **CNNSplice** (Model under test; run in testing mode per organism and site type to produce evaluation metrics) — https://github.com/OluwadareLab/CNNSplice.git
- **GeneSplicer** (Baseline evaluated on the same test datasets with author-provided models)
- **SpliceRover** (Baseline evaluated on the same test datasets with author-provided models)
- **Splice2Deep** (Baseline evaluated on the same test datasets with author-provided models)
- **SpliceFinder** (Baseline evaluated on the same test datasets with author-provided models)
- **DeepSplicer** (Baseline evaluated on the same test datasets with author-provided models)

## Examples

```
python test.py -n "output_name" -m "balanced"
```

## Evaluation signals

- Reproduced balanced donor/acceptor accuracies fall in the reported ~94-98% range
- CNNSplice Model 1 balanced acceptor accuracies match ~96.94% (H. sapiens), 96.26% (A. thaliana), 96.85% (O. sativa japonica), 97.00% (C. elegans)
- CNNSplice ranks 1st or 2nd (highest accuracy = rank 1) against the five baselines, matching the paper's reported evaluation
- A .txt metrics file is written to the log directory for every organism/site-type/mode combination tested
- Ranks are assigned consistently with the Friedman test convention: lowest rank for highest accuracy, rank equal to model count for lowest

## Limitations

- Dataset constraint: 'Our constructed dataset permits a Sequence Length of 400' — other lengths are unsupported
- A log directory must exist before running tests, or the .txt output files cannot be stored
- Genomic sequence input must be one-hot encoded before model inference; raw sequences will fail
- Fair comparison requires the author-provided baseline models; retraining baselines is only a fallback

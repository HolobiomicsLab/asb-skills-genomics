---
name: ivt-read-mixture-simulation
description: Use when training or benchmarking a site-level RNA modification classifier
  (m6A/m5C) that requires controlled modified/unmodified mixtures with known ground
  truth, using nanopore direct RNA IVT reads and CHEUI-solo Model 1 per-read probabilities.
when_to_use_negative:
- The input is real cellular transcriptome data without ground-truth modification
  status — use CHEUI-solo on real samples and control FDR with strict probability
  cutoffs instead of simulated mixtures
- The goal is differential modification between two conditions — CHEUI-diff, not the
  Model 2 mixture benchmark, is the appropriate workflow
- The reads are not R9.4.1 direct RNA data basecalled with Guppy and resquiggled with
  nanopolish eventalign, since CHEUI's preprocessing and trained models assume that
  signal pipeline
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CHEUI
  role: 'Two-stage deep learning method: Model 1 produces per-read m6A/m5C probability
    distributions from IVT signals; Model 2 (CNN binary classifier) consumes per-site
    distributions of Model 1 probabilities in the simulated mixtures'
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Source of observed IVT read ionic-current signals used as the raw material
    for mixture simulation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:d81ac5cfb2876da8c75ecf26e75fe9a73eef0cc6f9f23668c54a4df4a2638116
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - nanopore direct RNA sequencing
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# ivt-read-mixture-simulation

## Summary

Simulates controlled mixtures of modified/unmodified in-vitro transcript (IVT) read signals to train and benchmark CHEUI-solo Model 2, the CNN that predicts m6A and m5C at transcript-site level from distributions of per-read Model 1 probabilities.

## When to use

Use when training or benchmarking a site-level RNA modification classifier (m6A/m5C) that requires controlled modified/unmodified mixtures with known ground truth, using nanopore direct RNA IVT reads and CHEUI-solo Model 1 per-read probabilities.

## When NOT to use

- The input is real cellular transcriptome data without ground-truth modification status — use CHEUI-solo on real samples and control FDR with strict probability cutoffs instead of simulated mixtures
- The goal is differential modification between two conditions — CHEUI-diff, not the Model 2 mixture benchmark, is the appropriate workflow
- The reads are not R9.4.1 direct RNA data basecalled with Guppy and resquiggled with nanopolish eventalign, since CHEUI's preprocessing and trained models assume that signal pipeline

## Inputs

- Held-out modified and unmodified IVT read signals (Jenjaroenpun et al. direct RNA sequencing data, fast5-derived)
- CHEUI-solo Model 1 per-read modification probability distributions for m6A and m5C
- Nanopolish eventalign output preprocessed with CHEUI_preprocess_m6A.py / CHEUI_preprocess_m5C.py
- CHEUI trained model files CHEUI_m6A_model2.h5 and CHEUI_m5C_model2.h5

## Outputs

- Simulated training and testing sets of ~1.5 million synthetic sites per modification (9:1 split)
- CHEUI-solo Model 2 site-level methylation probabilities per transcript site
- Site-level ROC-AUC per modification (expected ~0.92 for m6A, ~0.953 for m5C)
- Per-site false positive rate on unmodified sites at probability > 0.99 (expected ~0.00074 m6A, ~0.00034 m5C)

## How to apply

Load held-out modified and unmodified IVT read signals from Jenjaroenpun et al. (disjoint from Model 1 training/validation/test data) together with their CHEUI-solo Model 1 per-read probability distributions. Simulate ~1.5 million synthetic sites per modification: choose modified vs unmodified with 50% probability, draw coverage between 0 and 100 with linear-decay weights ((10 - coverage) x 0.01 + 0.9), draw stoichiometry with the same decay for modified sites, and assign per-read Model 1 probabilities from the matching modified or unmodified pool. Split sites 9:1 into training and testing sets, then run CHEUI-solo Model 2, which consumes per-site distributions of Model 1 read probabilities, to obtain site-level methylation probabilities. Call sites positive when the Model 2 probability exceeds 0.99, and compute ROC-AUC per modification plus per-site false positive rate on fully unmodified sites. The rationale is that IVT mixtures provide ground-truth stoichiometry that cellular transcriptomes lack, enabling accurate measurement of site-level sensitivity and FPR.

## Related tools

- **CHEUI** (Two-stage deep learning method: Model 1 produces per-read m6A/m5C probability distributions from IVT signals; Model 2 (CNN binary classifier) consumes per-site distributions of Model 1 probabilities in the simulated mixtures) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Source of observed IVT read ionic-current signals used as the raw material for mixture simulation)

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- Reproduced site-level AUC is approximately 0.92 for m6A and 0.953 for m5C on the held-out IVT mixture test set
- Per-site false positive rate at CHEUI-solo Model 2 probability > 0.99 is approximately 0.00074 (m6A) and 0.00034 (m5C) on fully unmodified sites
- Simulated site coverage values fall in the 0–100 range drawn with the specified linear-decay weights, and sites split 9:1 into training/testing without leakage between Model 1 train and held-out read pools
- Modified/unmodified site counts are approximately balanced (50% probability draw), matching the controlled-mixture design used to train Model 2
- Positive site calls are defined exactly by Model 2 probability > 0.99 before computing AUC and FPR, mirroring the strict cutoffs used on cell transcriptome sites

## Limitations

- Strict cutoffs on cell transcriptome sites are needed to maintain a low expected false discovery rate, which reduces CHEUI sensitivity; relaxing thresholds would recover more orthogonal-method sites at the cost of potential false positives
- The nanopore signal variability limits accuracy, since differences between modified and non-modified reads are often comparable to differences among non-modified reads in a population
- Benchmark results are specific to the R9.4.1 RNA002 chemistry and Guppy 4.0.11+ basecalling used to generate the IVT training signals; newer CHEUI-based tools (SWARM) extend accuracy to RNA002 and add RNA004 and pseudouridine support

## Evidence

- [methods] CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads: "CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads"
- [methods] CHEUI-solo Model 1 trained on IVT read signals: "CHEUI-solo Model 1 was trained using read signals generated from in-vitro transcript (IVT) data26,27"
- [readme] CHEUI two-stage deep learning at read and site resolution: [evidence span withheld — anchor status: external_doc]
- [abstract] CHEUI high accuracy on synthetic standards: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [readme] nanopolish eventalign invocation required for preprocessing: [evidence span withheld — anchor status: not_found]

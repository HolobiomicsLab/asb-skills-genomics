---
name: binary-classification-metric-benchmarking
description: Use when you must choose between deep architectures (e.g., CNN, ResNet,
  TCN) for binary classification of raw DRS signals and need a defensible accuracy-vs-prediction-speed
  trade-off for real-time use.
when_to_use_negative:
- Your task targets only a single pre-selected architecture and no comparison table
  is needed
- Classification is not binary mRNA/non-mRNA (e.g., challenging lncRNA identification
  from 3' end signal, which the paper reports remains difficult)
- Real-time throughput is irrelevant and offline post-sequencing analysis suffices
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch (v1.9.0)
  role: Framework used to build, train, and test the CNN, ResNet, and TCN models
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files (FLO-MIN106, SQK-RNA002) to assign class labels for
    train/test splits
- name: minimap2 (v2.17)
  role: Maps reads to the GENCODE transcriptome with -ax map-ont --secondary=no to
    derive labels
- name: samtools (v1.10)
  role: Filters alignments with -F 2324 to retain only high-confidence forward-strand
    primary mappings
- name: BoostNano
  role: Trims sequencing adapter and poly(A) tail from raw signals before fixed-length
    input extraction
  repo: https://github.com/haotianteng/BoostNano
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:a2733f3981c28d20411bae0af37af2d8f975d1a30a96e088c5edff9596f4ab6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy (v4.0.14)
  - PyTorch (v1.9.0)
  - RISER
  - minimap2 (v2.17)
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# binary-classification-metric-benchmarking

## Summary

Benchmark candidate neural-network architectures (CNN, ResNet, TCN) for a binary classification task on raw nanopore DRS signals by training each best-tuned configuration identically and comparing accuracy, TPR, FPR, precision, AUROC, and mean prediction time per batch in a single table, enabling accuracy-vs-speed model selection for real-time deployment.

## When to use

Use when you must choose between deep architectures (e.g., CNN, ResNet, TCN) for binary classification of raw DRS signals and need a defensible accuracy-vs-prediction-speed trade-off for real-time use.

## When NOT to use

- Your task targets only a single pre-selected architecture and no comparison table is needed
- Classification is not binary mRNA/non-mRNA (e.g., challenging lncRNA identification from 3' end signal, which the paper reports remains difficult)
- Real-time throughput is irrelevant and offline post-sequencing analysis suffices

## Inputs

- Raw nanopore fast5 DRS files (model-development datasets)
- GENCODE release 34 (GRCh38.p13) transcriptome annotation with biotypes
- Best-tuned hyperparameter configurations for each candidate architecture (CNN, ResNet, TCN)
- Hardware: NVIDIA Tesla V100 GPU; software: PyTorch v1.9.0

## Outputs

- Comparison table of accuracy, TPR, FPR, precision, AUROC, TPR:FPR ratio, and mean prediction time per batch of 32 signals for each architecture at each tested input length (1-4 s)
- Ranking of architectures by accuracy and prediction speed
- Selected model configuration for the RISER deployment

## How to apply

Curate balanced mRNA/non-mRNA training and test sets by basecalling fast5 files with Guppy v4.0.14, mapping to the GENCODE release 34 transcriptome with minimap2 v2.17, retaining only high-confidence forward-strand primary alignments with samtools v1.10 (-F 2324), splitting 80/20, and undersampling the majority class to 50/50. Trim adapter and poly(A) with BoostNano, extract the first 1-4 s of median-absolute-deviation-normalized signal, and train each best-tuned architecture in PyTorch v1.9.0 on a Tesla V100 with binary cross-entropy loss and Adam for up to 100 epochs within a 48-hour limit. Evaluate each model on the reserved test set at candidate input lengths of 1-4 s, computing accuracy, TPR, FPR, precision, AUROC, the TPR:FPR ratio, and mean prediction time per batch of 32 signals. Compile all metrics into a single comparison table and adopt the architecture that maximizes validation/test accuracy while meeting the prediction-time budget for live read-until use.

## Related tools

- **PyTorch (v1.9.0)** (Framework used to build, train, and test the CNN, ResNet, and TCN models)
- **Guppy (v4.0.14)** (Basecalls fast5 files (FLO-MIN106, SQK-RNA002) to assign class labels for train/test splits)
- **minimap2 (v2.17)** (Maps reads to the GENCODE transcriptome with -ax map-ont --secondary=no to derive labels)
- **samtools (v1.10)** (Filters alignments with -F 2324 to retain only high-confidence forward-strand primary mappings)
- **BoostNano** (Trims sequencing adapter and poly(A) tail from raw signals before fixed-length input extraction) — https://github.com/haotianteng/BoostNano

## Evaluation signals

- The comparison table reports accuracy, TPR, FPR, precision, AUROC, TPR:FPR ratio, and prediction time for all three architectures at every tested input length (1-4 s)
- The highest-accuracy architecture matches the paper's ranking (ResNet and CNN above TCN, CNN selected for RISER)
- The CNN's mean prediction time per batch of 32 signals is approximately half that of ResNet and TCN
- Test set is 50/50 class-balanced after majority-class undersampling and pseudogenes were removed before labeling

## Limitations

- Training each architecture is bounded by a 48-hour limit, which constrains how many epochs/hyperparameter configurations can be explored
- Prediction speed measured on a Tesla V100 may not transfer to the CPU-only, real-time read-until deployment environment
- Accurate lncRNA identification from the 3' end signal remains challenging, so benchmark conclusions for mRNA/non-mRNA may not extend to other classes

## Evidence

- [methods] Model training details: "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
- [methods] Filtering for labels: "unmapped reads were removed using samtools34 (v1.10) with option -F 2324"
- [methods] Input length trade-off: "The input signal length to use for the RISER model was determined by considering the trade-off between needing to minimize input length"
- [readme] Deployment context: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"

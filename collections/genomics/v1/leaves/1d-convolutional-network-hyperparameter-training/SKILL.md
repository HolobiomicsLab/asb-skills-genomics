---
name: 1d-convolutional-network-hyperparameter-training
description: Decision support for choosing a 1D CNN architecture and input signal
  length when classifying RNA classes directly from raw nanopore signal windows and
  prediction speed on CPU/GPU must be traded against accuracy.
when_to_use_negative:
- Reads are already basecalled sequence — sequence-based adaptive sampling (basecalling
  + mapping to a target list) is the alternative, not signal-space CNN training
- The target RNA class has poorly characterized 3'-end signal (e.g., lncRNAs with
  cell type-specific expression), which the article notes remains challenging to identify
  accurately
- Input is bulk-fast5 playback testing of an already-trained RISER model rather than
  model development and hyperparameter selection
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: PyTorch (v1.9.0)
  role: Framework used to build, train, and test the CNN/ResNet/TCN models on a single
    NVIDIA Tesla V100 GPU
- name: BoostNano
  role: Trims the sequencing adapter and variable-length poly(A) tail from raw fast5
    signals before window extraction and network training
  repo: https://github.com/haotianteng/BoostNano
- name: RISER
  role: Downstream command-line tool that deploys the selected CNN to reject or keep
    molecules in real time via the ReadUntil API
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:4c459d7c8a3a2173ac1535d1d6ade0cf6453b72570cc2c9200e31f1c7e418f4a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# 1d-convolutional-network-hyperparameter-training

## Summary

Trains and hyperparameter-benchmarks 1D convolutional architectures (vanilla CNN, ResNet, TCN) on the first ≤4 s of raw nanopore direct RNA sequencing (DRS) signals for mRNA/non-mRNA binary classification, selecting the architecture that best balances accuracy against per-batch prediction latency for real-time read-until deployment.

## When to use

Decision support for choosing a 1D CNN architecture and input signal length when classifying RNA classes directly from raw nanopore signal windows and prediction speed on CPU/GPU must be traded against accuracy.

## When NOT to use

- Reads are already basecalled sequence — sequence-based adaptive sampling (basecalling + mapping to a target list) is the alternative, not signal-space CNN training
- The target RNA class has poorly characterized 3'-end signal (e.g., lncRNAs with cell type-specific expression), which the article notes remains challenging to identify accurately
- Input is bulk-fast5 playback testing of an already-trained RISER model rather than model development and hyperparameter selection

## Inputs

- MinION DRS fast5 files from model-development samples (human heart, GM12878, GM24385, HEK293, HeLa, KOPN8, REH)
- GENCODE release 34 transcriptome (GRCh38.p13) annotation for mRNA/non-mRNA biotype labels (pseudogenes excluded)
- Trimmed and MAD-normalized signal windows: adapter/poly(A)-removed, first 4 s of the 3'-end raw signal (~1,073,720 balanced training signals)

## Outputs

- Best-tuned PyTorch models of each 1D architecture (CNN, ResNet, TCN) applicable at 1–4 s input signal lengths
- Comparison table ranking the three architectures by accuracy, TPR, FPR, precision, AUROC, TPR:FPR ratio, and mean prediction time per batch of 32 signals
- Selected classifier (the vanilla CNN) adopted as the RISER model for read-until rejection

## How to apply

Build labels by basecalling model-development fast5 files with Guppy v4.0.14 (FLO-MIN106, SQK-RNA002), mapping to the GENCODE release 34 transcriptome (GRCh38.p13) with minimap2 v2.17 (-ax map-ont --secondary=no), and retaining high-confidence forward-strand primary alignments with samtools v1.10 (-F 2324). Split reads into mRNA/non-mRNA classes by GENCODE biotype (pseudogenes removed), split 80/20 into train/test with 20% of training for tuning, undersample the majority mRNA class to 50/50, then trim adapter and poly(A) tail with BoostNano and extract the first 4 s of MAD-normalized signal (~1,073,720 training signals). In PyTorch v1.9.0 on a Tesla V100 GPU, train each tuned architecture with binary cross-entropy loss and Adam for up to 100 epochs within 48 h — ResNet (basic blocks, kernel 19, l=10, c0=20, ×1.5 channel growth, batch 32, lr 0.001), TCN (l=10, kernel 11, dilation base 2, 32 channels, dropout 0.05, batch 32, lr 0.0001), and vanilla CNN (l=12, kernel 3, GAP_FC classifier, c0=20, ×1.5 growth, batch 32, lr 0.0001). Evaluate each best model on the held-out test set at candidate input lengths of 1–4 s (CNN from 2 s), compiling accuracy, TPR, FPR, precision, AUROC, TPR:FPR ratio, and mean prediction time per batch of 32 signals into a single ranking table.

## Related tools

- **PyTorch (v1.9.0)** (Framework used to build, train, and test the CNN/ResNet/TCN models on a single NVIDIA Tesla V100 GPU)
- **BoostNano** (Trims the sequencing adapter and variable-length poly(A) tail from raw fast5 signals before window extraction and network training) — https://github.com/haotianteng/BoostNano
- **RISER** (Downstream command-line tool that deploys the selected CNN to reject or keep molecules in real time via the ReadUntil API) — https://github.com/comprna/riser

## Evaluation signals

- The comparison table must report accuracy, TPR, FPR, precision, AUROC, TPR:FPR ratio, and mean prediction time per batch of 32 for all three architectures at each tested input length (1–4 s; CNN from 2 s)
- ResNet and CNN accuracy should exceed TCN accuracy at all input lengths, matching the paper's reported ranking
- The CNN's mean prediction time per batch of 32 signals should be roughly half that of ResNet and TCN
- Training of each best-tuned configuration completes within the 48-hour / 100-epoch limit with binary cross-entropy loss and Adam on a Tesla V100
- Class balance is 50/50 after majority-class undersampling, so accuracy values are interpretable against a ~0.5 random baseline

## Limitations

- The model was trained to distinguish mRNA from non-mRNA at the 3' end; accurate lncRNA identification from 3'-end signal remains challenging, and cell type-specific lncRNA expression hinders generalization
- Larger training datasets, especially for lowly expressed RNA classes, are needed to improve sensitivity to rare RNAs
- BoostNano only works with Python < 3.9 (tested on Python 3.7/3.8), constraining the training environment
- Hyperparameter search and evaluation were bounded by a 48-hour training limit and up to 100 epochs, so configurations are best-tuned within that budget only

## Evidence

- [readme] RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"
- [methods] All models were built, trained, and tested using PyTorch (v1.9.0): "with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
- [methods] The input signal length to use for the RISER model was determined by considering the trade-off between needing to minimize input length: "The input signal length to use for the RISER model was determined by considering the trade-off between needing to minimize input length"
- [other] Train the best-tuned configuration of each architecture in PyTorch v1.9.0 on a Tesla V100 GPU with binary cross-entropy loss and Adam for up to 100 epochs within a 48-hour limit: [evidence span withheld — anchor status: not_found]

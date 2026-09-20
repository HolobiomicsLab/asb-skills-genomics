---
name: per-transcript-classification-error-tabulation
description: Use when a deep learning signal classifier (e.g., RISER globin model)
  has been run on a labeled held-out DRS test set and you need transcript- and biotype-resolved
  error breakdowns rather than a single aggregate accuracy number.
when_to_use_negative:
- When you only need an aggregate accuracy figure for the whole test set rather than
  transcript- or biotype-resolved error rates
- When the input signals are live-run rejects/ejections (signals truncated by read-until
  reversal) rather than complete reserved test-set signals, since trimming and normalization
  at 2/3/4 s assume full translocation starts
- When the RNA class of interest is not one supported/retrained in RISER (e.g., lncRNA-specific
  identification, which the article notes remains challenging from 3' end signal)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3524
tools:
- name: RISER
  role: CNN model applied to processed signals to produce target-class probabilities
    for globin vs non-globin classification
  repo: https://github.com/comprna/riser
- name: BoostNano
  role: Trims the sequencing adapter and variable-length poly(A) tail from the start
    of raw nanopore signals before model input
  repo: https://github.com/haotianteng/BoostNano
- name: PyTorch
  role: Framework in which the RISER CNN globin model is run for inference on the
    test set
- name: minimap2
  role: Maps basecalled reads to the GENCODE reference transcriptome to derive ground-truth
    gene and biotype labels
- name: samtools
  role: Filters mappings (removes reverse-strand, secondary/supplementary, and unmapped
    reads) to retain only high-confidence ground-truth labels
- name: Guppy
  role: Basecalls fast5 files (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002) so
    reads can be assigned transcript labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:bf1eb6f7b6ab4d8e1ac4a5d2e3bbc1af1c5fd267e8056d42e56f9f67470adca9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/riser
  tools:
  - BoostNano
  - Guppy
  - PyTorch
  - RISER
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# per-transcript-classification-error-tabulation

## Summary

Tabulates how a trained RISER globin CNN classifies each transcript and biotype in a reserved whole-blood direct RNA sequencing (DRS) test set, decomposing overall accuracy into per-transcript correct/incorrect/undecided rates and per-biotype accuracies (HBA1, HBA2, HBB, mtRNA, lncRNA, other).

## When to use

Use when a deep learning signal classifier (e.g., RISER globin model) has been run on a labeled held-out DRS test set and you need transcript- and biotype-resolved error breakdowns rather than a single aggregate accuracy number.

## When NOT to use

- When you only need an aggregate accuracy figure for the whole test set rather than transcript- or biotype-resolved error rates
- When the input signals are live-run rejects/ejections (signals truncated by read-until reversal) rather than complete reserved test-set signals, since trimming and normalization at 2/3/4 s assume full translocation starts
- When the RNA class of interest is not one supported/retrained in RISER (e.g., lncRNA-specific identification, which the article notes remains challenging from 3' end signal)

## Inputs

- Reserved whole-blood DRS test set of 84,831 raw nanopore signals (fast5)
- GENCODE-based gene and biotype label mappings (positive class: HBA1, HBA2, HBB; negative class: non-globin including mtRNA and lncRNA)
- Trained RISER globin CNN model checkpoint (PyTorch)
- Classification threshold T = 0.9

## Outputs

- globin_biotype_accuracy.csv: per-biotype accuracy table over the 84,831-signal test set (HBA1, HBA2, HBB, mtRNA, lncRNA, other)
- Per-transcript tabulation of proportions correctly classified as globin, incorrectly classified, and undecided (no confident decision at T = 0.9)

## How to apply

Load the reserved globin-model test set (84,831 whole-blood DRS signals) and trim the sequencing adapter and poly(A) tail from each signal with BoostNano (commit 8715800) so evaluation isolates model performance from signal-preparation artifacts. Extract and normalize the transcript signal (median absolute deviation normalization with outlier smoothing) at the trained input lengths of 2 s, 3 s, and 4 s, then run the RISER CNN globin model in PyTorch (best validation accuracy 98.6% after epoch 6) to obtain target-class probabilities, applying classification threshold T = 0.9 to derive correct/incorrect/undecided decisions per read. Assign ground-truth labels from GENCODE-based mappings: reads from globin genes HBA1, HBA2, and HBB form the positive class, while non-globin reads (including mtRNA and lncRNA, balanced one read at a time per gene) form the negative class. Finally, tabulate per transcript and per biotype the proportions of reads correctly classified as globin, incorrectly classified, and undecided, and compute per-biotype accuracy (HBA1, HBA2, HBB separately; mtRNA, lncRNA, and remaining biotypes aggregated as 'other') into globin_biotype_accuracy.csv. The rationale is that aggregate accuracy can mask per-transcript failure modes, so per-transcript error decomposition reveals whether misclassification is concentrated in specific genes or biotypes.

## Related tools

- **RISER** (CNN model applied to processed signals to produce target-class probabilities for globin vs non-globin classification) — https://github.com/comprna/riser
- **BoostNano** (Trims the sequencing adapter and variable-length poly(A) tail from the start of raw nanopore signals before model input) — https://github.com/haotianteng/BoostNano
- **PyTorch** (Framework in which the RISER CNN globin model is run for inference on the test set)
- **minimap2** (Maps basecalled reads to the GENCODE reference transcriptome to derive ground-truth gene and biotype labels)
- **samtools** (Filters mappings (removes reverse-strand, secondary/supplementary, and unmapped reads) to retain only high-confidence ground-truth labels)
- **Guppy** (Basecalls fast5 files (v4.0.14, --flowcell FLO-MIN106 --kit SQK-RNA002) so reads can be assigned transcript labels)

## Examples

```
python boostnano_eval.py -i TEST_FAST5_FOLDER -o TRIMMED_FOLDER -m MODEL_FOLDER --replace && python riser.py --target globin --mode deplete --duration 0.1 --kit RNA002
```

## Evaluation signals

- globin_biotype_accuracy.csv exists and contains one accuracy row per biotype (HBA1, HBA2, HBB, mtRNA, lncRNA, other) computed over the 84,831-signal test set
- Globin-gene per-transcript accuracies are consistent with the paper's reported high globin classification performance (>90% correct globin classification/depletion; 98% globin detection)
- Per-biotype accuracies of correctly classified non-globin biotypes (mtRNA, lncRNA, other) exceed 99%
- Every read is assigned exactly one of three decision outcomes (correct, incorrect, undecided) under threshold T = 0.9, and per-transcript proportions of the three outcomes sum to 1
- Best validation accuracy of the checkpoint used (98.6% after epoch 6) matches the model configuration being evaluated

## Limitations

- Reads for which the model produced no confident decision at T = 0.9 are counted as undecided, so accuracy figures depend on the threshold choice
- Undecided/incorrect non-globin reads may reflect confusable biotypes; the article notes accurate lncRNA identification from 3' end signal remains challenging and cell type-specific lncRNA expression hinders generalization to new samples
- The evaluation isolates model performance only after BoostNano adapter/poly(A) trimming and MAD normalization; applying it to untrimmed signals will degrade results

## Evidence

- [results] The RISER globin model correctly detected globin mRNA with 98% accuracy and correctly classified all other RNA biotypes as non-globin with >99% accuracy.: [evidence span withheld — anchor status: not_found]
- [methods] To ensure RISER only makes high-confidence predictions, a classification threshold T is applied when translating the probability output: "To ensure RISER only makes high-confidence predictions, a classification threshold T is applied when translating the probability output"
- [methods] the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano"
- [methods] All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU): "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"
- [abstract] We also apply RISER for the depletion of globin mRNA in whole blood: "We also apply RISER for the depletion of globin mRNA in whole blood"

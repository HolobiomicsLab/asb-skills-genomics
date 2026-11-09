---
name: per-biotype-accuracy-evaluation
description: Use when you have a reserved test set of raw nanopore DRS signals with
  GENCODE-derived ground-truth gene/biotype labels and need per-biotype classification
  accuracy of a trained RISER model.
when_to_use_negative:
- You need enrichment/depletion statistics from a live read-until run rather than
  offline classification accuracy on a labeled reserved test set.
- Input reads have not been mapped to the GENCODE reference transcriptome (release
  34, GRCh38.p13), so ground-truth biotype labels cannot be assigned.
- The target RNA class is not globin and no corresponding retrained RISER model checkpoint
  is available.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0080
tools:
- name: BoostNano
  role: Trims the sequencing adapter and poly(A) tail from raw nanopore signals before
    model evaluation
  repo: https://github.com/haotianteng/BoostNano
- name: RISER
  role: CNN globin model that outputs target-class probabilities on trimmed, normalized
    signals
  repo: github.com/comprna/riser
- name: PyTorch
  role: Framework in which the RISER model is built, trained, and tested (v1.9.0,
    NVIDIA Tesla V100)
- name: minimap2
  role: Maps basecalled reads to the GENCODE reference transcriptome to derive ground-truth
    gene/biotype labels (v2.17, -ax map-ont --secondary=no)
- name: samtools
  role: Filters mappings to retain only high-confidence assignments by removing unmapped
    reads (v1.10, -F 2324)
- name: Guppy
  role: Basecalls fast5 files so reads can be mapped and labeled prior to signal-level
    evaluation (v4.0.14, FLO-MIN106, SQK-RNA002)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:3cd1256daf8ce1d2d98bd78fa4b22b3e52c1720f21c5a2d6a12ed4d63afa2985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy
  - PyTorch
  - RISER
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# per-biotype-accuracy-evaluation

## Summary

Evaluates a trained RISER globin classification model on a reserved whole-blood DRS test set by tabulating correct/incorrect/undecided decisions per transcript and per RNA biotype, producing a per-biotype accuracy table (globin_biotype_accuracy.csv).

## When to use

Use when you have a reserved test set of raw nanopore DRS signals with GENCODE-derived ground-truth gene/biotype labels and need per-biotype classification accuracy of a trained RISER model.

## When NOT to use

- You need enrichment/depletion statistics from a live read-until run rather than offline classification accuracy on a labeled reserved test set.
- Input reads have not been mapped to the GENCODE reference transcriptome (release 34, GRCh38.p13), so ground-truth biotype labels cannot be assigned.
- The target RNA class is not globin and no corresponding retrained RISER model checkpoint is available.

## Inputs

- Reserved test set of whole-blood DRS fast5 signals (84,831 signals)
- GENCODE-based gene and biotype label mappings for each read (HBA1/HBA2/HBB positive; mtRNA, lncRNA, other negative)
- Trained RISER globin CNN model checkpoint (PyTorch, best validation accuracy 98.6% after epoch 6)
- BoostNano-pretrained segmentation model

## Outputs

- globin_biotype_accuracy.csv — per-biotype accuracy table (HBA1, HBA2, HBB, mtRNA, lncRNA, other) over the 84,831-signal test set
- Per-transcript proportions of reads correctly classified, incorrectly classified, and undecided

## How to apply

Load the reserved globin-model test set (84,831 whole-blood DRS signals) and trim the sequencing adapter and poly(A) tail from each signal with BoostNano so evaluation isolates model performance. Extract and MAD-normalize the transcript signal at the trained input lengths of 2 s, 3 s, and 4 s, then run the RISER CNN globin model in PyTorch (best validation accuracy 98.6% at epoch 6) to obtain target-class probabilities, applying the high-confidence classification threshold T = 0.9 to derive correct/incorrect/undecided decisions per read. Assign ground-truth labels from GENCODE mappings: reads from globin genes HBA1, HBA2, and HBB form the positive class, while non-globin reads (including mtRNA and lncRNA biotypes, balanced one read at a time per gene) form the negative class. Finally, tabulate decisions per transcript and per biotype and compute accuracy for HBA1, HBA2, HBB, mtRNA, lncRNA, and remaining biotypes aggregated as 'other', writing the result to globin_biotype_accuracy.csv.

## Related tools

- **BoostNano** (Trims the sequencing adapter and poly(A) tail from raw nanopore signals before model evaluation) — https://github.com/haotianteng/BoostNano
- **RISER** (CNN globin model that outputs target-class probabilities on trimmed, normalized signals) — github.com/comprna/riser
- **PyTorch** (Framework in which the RISER model is built, trained, and tested (v1.9.0, NVIDIA Tesla V100))
- **minimap2** (Maps basecalled reads to the GENCODE reference transcriptome to derive ground-truth gene/biotype labels (v2.17, -ax map-ont --secondary=no))
- **samtools** (Filters mappings to retain only high-confidence assignments by removing unmapped reads (v1.10, -F 2324))
- **Guppy** (Basecalls fast5 files so reads can be mapped and labeled prior to signal-level evaluation (v4.0.14, FLO-MIN106, SQK-RNA002))

## Examples

```
python3 riser.py --target globin --mode deplete --duration 48 --kit RNA002
```

## Evaluation signals

- Output file globin_biotype_accuracy.csv exists and contains a per-biotype accuracy table computed over the full 84,831-signal test set.
- Globin-gene accuracies (HBA1, HBA2, HBB) are consistent with the paper's reported >90% correct globin classification/depletion performance (98% reported).
- All non-globin biotypes (mtRNA, lncRNA, other) show >99% correct classification as non-globin.
- Every test read falls into exactly one of three decision categories: correct, incorrect, or undecided (no confident decision at threshold T = 0.9).
- Per-biotype proportions of correct/incorrect/undecided decisions sum to 1 within each biotype.

## Limitations

- Undecided reads (below the confidence threshold T = 0.9) are not counted as either correct or incorrect, which can mask model uncertainty in aggregate accuracy figures.
- Model performance is specific to the RNA002/R9.4.1 whole-blood dataset and trained input lengths (2 s, 3 s, 4 s); the CNN's best validation accuracy (98.6% at epoch 6) may not transfer to other chemistries without retraining.
- Non-globin reads were balanced one read at a time per gene, so per-biotype accuracy may not reflect natural abundance distributions in whole blood.

## Evidence

- [other] Load the reserved globin-model test set (84,831 whole-blood DRS signals) and trim the sequencing adapter and poly(A) tail from each signal with BoostNano: [evidence span withheld — anchor status: not_found]
- [other] The RISER globin model correctly detected globin mRNA with 98% accuracy and correctly classified all other RNA biotypes as non-globin with >99% accuracy.: [evidence span withheld — anchor status: not_found]
- [methods] the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [methods] To ensure RISER only makes high-confidence predictions, a classification threshold T is applied when translating the probability output: "To ensure RISER only makes high-confidence predictions, a classification threshold T is applied when translating the probability output"
- [abstract] achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average: "achieving a decrease in globin reads by more than 90% as well as an increase in non-globin reads by 16% on average"
- [methods] All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU): "All models were built, trained, and tested using PyTorch (v1.9.0)36 with a single NVIDIA Tesla V100 graphics processing unit (GPU)"

---
name: nanopore-signal-classification
description: Use when raw nanopore DRS signals (fast5, RNA002 or RNA004 kits) must
  be classified into RNA classes such as protein_coding mRNA, mtRNA, lncRNA or globin
  (HBA1/HBA2/HBB) directly from the first few seconds of signal.
when_to_use_negative:
- Accurate lncRNA-only discrimination is the goal — the paper states accurate lncRNA
  identification from the 3′ end signal remains challenging and lncRNA cell type-specific
  expression hinders generalization
- The task requires full-length sequence output or reference-based alignment of complete
  reads — RISER classifies from only the first ~70–280 nt of raw signal and does not
  basecall
- Input signals are cDNA sequencing or non-RNA002/RNA004 nanopore chemistry not supported
  by the shipped models
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: RISER
  role: CNN that classifies RNA classes directly from the first few seconds of raw
    nanopore signals; provides pretrained mRNA, mtRNA, and globin models and the retraining
    code
  repo: github.com/comprna/riser
- name: BoostNano
  role: Trims the sequencing adapter and variable-length poly(A) tail from raw signals
    before classification so evaluation isolates model performance
  repo: https://github.com/haotianteng/BoostNano
- name: PyTorch
  role: Framework for building, training, and running the RISER CNN models (v1.9.0,
    single NVIDIA Tesla V100 GPU)
- name: Guppy
  role: Basecalls fast5 files (v4.0.14, FLO-MIN106/SQK-RNA002) to generate mappings
    used as ground-truth biotype labels
- name: minimap2
  role: Maps basecalled reads to the GENCODE transcriptome (v2.17, -ax map-ont --secondary=no)
    for label assignment
- name: samtools
  role: Filters alignments (v1.10, -F 2324) to retain only high-confidence mappings
    for training/testing labels
- name: nanoporetech read_until_api
  role: Carries RISER's reject/unblock commands to the MinION during live sequencing
    to physically eject unwanted RNAs from the pore
  repo: nanoporetech/read_until_api
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:65a791fb8a1b88dbf27217c91afe7718c0e00a9e3a3e87f20a9217016b553d4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoostNano
  - Dorado
  - Guppy
  - Guppy (v4.0.14)
  - PyTorch
  - PyTorch (v1.9.0)
  - RISER
  - minimap2
  - minimap2 (v2.17)
  - nanoporetech read_until_api
  - samtools
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 22
---

# nanopore-signal-classification

## Summary

Classify RNA biotypes (mRNA, mtRNA, globin mRNA) directly from raw nanopore direct RNA sequencing signals using the RISER convolutional neural network, without basecalling or a reference, enabling biochemical-free real-time enrichment or depletion via the read-until system.

## When to use

Use when raw nanopore DRS signals (fast5, RNA002 or RNA004 kits) must be classified into RNA classes such as protein_coding mRNA, mtRNA, lncRNA or globin (HBA1/HBA2/HBB) directly from the first few seconds of signal.

## When NOT to use

- Accurate lncRNA-only discrimination is the goal — the paper states accurate lncRNA identification from the 3′ end signal remains challenging and lncRNA cell type-specific expression hinders generalization
- The task requires full-length sequence output or reference-based alignment of complete reads — RISER classifies from only the first ~70–280 nt of raw signal and does not basecall
- Input signals are cDNA sequencing or non-RNA002/RNA004 nanopore chemistry not supported by the shipped models

## Inputs

- Raw nanopore DRS signal files (fast5, SQK-RNA002 with FLO-MIN106 or RNA004 with FLO-MIN004RA)
- Pre-trained RISER model weights (best_model.pth) and config yaml per target class (mRNA, mtRNA, globin)
- GENCODE reference transcriptome (release 34 or 43, GRCh38.p13) for ground-truth biotype assignment
- Bulk fast5 file for MinKNOW playback testing

## Outputs

- Per-read classification decisions (correct/incorrect/undecided) with target-class probabilities at threshold T = 0.9
- Per-biotype accuracy table (e.g., globin_biotype_accuracy.csv over 84,831 whole-blood test signals)
- Accuracy, precision, TPR, and FPR metrics per model (e.g., mRNA ~94%, mtRNA ~99% on HeLa)
- Real-time reject/unblock commands to the sequencer via the ONT read-until API during live runs

## How to apply

Trim the sequencing adapter and poly(A) tail from each raw signal with BoostNano so evaluation isolates model performance, then extract and normalize the transcript signal (median absolute deviation normalization with outlier smoothing) at the trained input lengths of 2 s, 3 s, or 4 s. Run the appropriate RISER CNN (mRNA, mtRNA, or globin; built in PyTorch v1.9.0, e.g. best validation accuracy 98.6% after epoch 6 for the globin model) to obtain target-class probabilities. Apply the classification threshold T = 0.9 to derive confident accept/reject decisions, treating low-probability reads as undecided. Assign ground-truth labels by basecalling with Guppy (v4.0.14, FLO-MIN106/SQK-RNA002), mapping with minimap2 (v2.17, -ax map-ont --secondary=no) to GENCODE release 34 (GRCh38.p13), and filtering with samtools (v1.10, -F 2324), retaining only reads whose primary mapping biotype matches the most frequent secondary mapping biotype. Evaluate accuracy, precision, TPR, and FPR per biotype; the expected performance on held-out HeLa data is ~94% accuracy for the mRNA model and ~99% for mtRNA, and 98% globin accuracy on whole-blood DRS signals.

## Related tools

- **RISER** (CNN that classifies RNA classes directly from the first few seconds of raw nanopore signals; provides pretrained mRNA, mtRNA, and globin models and the retraining code) — github.com/comprna/riser
- **BoostNano** (Trims the sequencing adapter and variable-length poly(A) tail from raw signals before classification so evaluation isolates model performance) — https://github.com/haotianteng/BoostNano
- **PyTorch** (Framework for building, training, and running the RISER CNN models (v1.9.0, single NVIDIA Tesla V100 GPU))
- **Guppy** (Basecalls fast5 files (v4.0.14, FLO-MIN106/SQK-RNA002) to generate mappings used as ground-truth biotype labels)
- **minimap2** (Maps basecalled reads to the GENCODE transcriptome (v2.17, -ax map-ont --secondary=no) for label assignment)
- **samtools** (Filters alignments (v1.10, -F 2324) to retain only high-confidence mappings for training/testing labels)
- **nanoporetech read_until_api** (Carries RISER's reject/unblock commands to the MinION during live sequencing to physically eject unwanted RNAs from the pore) — nanoporetech/read_until_api

## Examples

```
python3 riser.py --target mRNA --mode deplete --duration 0.1 --kit RNA004
```

## Evaluation signals

- Held-out HeLa test set (607,740 signals): mRNA model accuracy ≈94%, mtRNA model accuracy ≈99%, precision 0.99, TPR 0.94, FPR 0.05, matching Fig. 3a-d
- Globin model on 84,831 reserved whole-blood DRS signals: 98% correct globin detection and >99% correct non-globin classification, with a per-biotype accuracy table (globin_biotype_accuracy.csv) showing >90% globin classification
- Every predicted read is the product of adapter/poly(A)-trimmed signal at one of the trained input lengths (2 s, 3 s, 4 s) with decisions at threshold T = 0.9 including an 'undecided' category
- Playback test on MinKNOW: average length of non-mRNA reads exceeds that of mRNA reads being rejected (signals clipped on reject), confirming hardware communication works before live runs
- Speed benchmark check: RISER classification time per 6.2 s trimmed signal is shorter than Dorado basecalling plus minimap2 mapping, e.g. ~44× faster on GPU and ~16× faster on CPU than GPU-based adaptive sampling (~5.8 s/signal)

## Limitations

- The lag between the read-until API receiving a reject command and requesting voltage reversal, and the efficiency of the voltage reversal itself, are beyond RISER's control and vary across platforms
- RNA secondary structures may form on the trans side of the pore, impeding or completely inhibiting reverse translocation by voltage reversal
- Accurate lncRNA identification from the 3′ end signal remains challenging, and cell type-specific lncRNA expression makes it harder for models to generalize to new samples
- Larger training datasets, especially for lowly expressed RNA classes, are needed to improve sensitivity to rare RNAs

## Evidence

- [abstract] Here we introduce RISER, a biochemical-free technology for the real-time enrichment or depletion of RNA classes.: "Here we introduce RISER, a biochemical-free technology for the real-time enrichment or depletion of RNA classes."
- [intro] RISER identifies RNA classes in real-time during sequencing, directly from just the first few seconds of raw nanopore signals using a deep learning model: "RISER identifies RNA classes in real-time during sequencing, directly from just the first few seconds of raw nanopore signals using a deep learning model"
- [methods] the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano"
- [discussion] it performs the reject decision by directly classifying the raw signal rather than using real-time basecalling and mapping: "it performs the reject decision by directly classifying the raw signal rather than using real-time basecalling and mapping"
- [readme] RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"

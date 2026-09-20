---
name: direct-rna-signal-preprocessing
description: Use when raw nanopore DRS fast5 signals must be cleaned before model
  training, evaluation, or live read-until decisions — e.g., isolating model performance
  on a reserved test set or feeding trimmed signals to a CNN classifier.
when_to_use_negative:
- Input is already basecalled sequence (FASTQ/BAM) — trimming then applies at the
  sequence level, not raw signal space
- Signals were produced by non-RNA nanopore chemistries (e.g., DNA sequencing kits)
  where adapter/poly(A) stall structure differs from RNA002/RNA004
- Only post-sequencing read filtering is needed (e.g., removing globin reads bioinformatically),
  which the paper notes cannot compensate for depleted coverage
edam_operation: http://edamontology.org/operation_3237
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3524
tools:
- name: BoostNano
  role: Segments and trims the sequencing adapter and poly(A) tail from raw DRS signals
    before basecalling or model inference
  repo: https://github.com/haotianteng/BoostNano
- name: RISER
  role: Consumes the trimmed, normalized signal slices to classify RNA classes in
    real time from the first few seconds of signal
  repo: https://github.com/comprna/riser
- name: Guppy (v4.0.14)
  role: Basecalls fast5 files (options –flowcell FLO-MIN106 –kit SQK-RNA002) to derive
    ground-truth labels for validating preprocessing quality
- name: minimap2 (v2.17)
  role: Maps basecalled reads to the GENCODE reference transcriptome (release 34,
    GRCh38.p13, options -ax map-ont --secondary=no) for label assignment after signal
    processing
- name: samtools (v1.10)
  role: Filters high-confidence mappings (option -F 2324 removes unmapped and low-confidence
    alignments) after basecalling trimmed signals
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:fe2fd817153e2d7d73f00a459f4e6d048f9b51ca5c9fe4b119ec9582aed11fd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Guppy
  - Guppy (v4.0.14)
  - PyTorch
  - RISER
  - minimap2
  - minimap2 (v2.17)
  - samtools
  - samtools (v1.10)
  edam_operation: http://edamontology.org/operation_3237
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# direct-rna-signal-preprocessing

## Summary

Trim the sequencing adapter and variable-length poly(A) tail from raw ONT direct RNA sequencing (DRS) fast5 signals with BoostNano, then extract and normalize transcript signal segments (median absolute deviation normalization with outlier smoothing) so downstream classifiers such as RISER operate on clean, fixed-length signal inputs.

## When to use

Use when raw nanopore DRS fast5 signals must be cleaned before model training, evaluation, or live read-until decisions — e.g., isolating model performance on a reserved test set or feeding trimmed signals to a CNN classifier.

## When NOT to use

- Input is already basecalled sequence (FASTQ/BAM) — trimming then applies at the sequence level, not raw signal space
- Signals were produced by non-RNA nanopore chemistries (e.g., DNA sequencing kits) where adapter/poly(A) stall structure differs from RNA002/RNA004
- Only post-sequencing read filtering is needed (e.g., removing globin reads bioinformatically), which the paper notes cannot compensate for depleted coverage

## Inputs

- Raw ONT fast5 files from direct RNA sequencing (SQK-RNA002 on FLO-MIN106, or RNA004 kit)
- BoostNano pretrained model folder
- Trained model input length specification (2 s / 3 s / 4 s signal windows)

## Outputs

- Fast5 files with adapter- and poly(A)-trimmed signals written to the Signal slot (original preserved in Signal_Old)
- Segmentation/trimming boundary results (output.csv from BoostNano evaluation)
- Normalized, fixed-length signal segments ready for CNN classification or model training/testing

## How to apply

Run BoostNano (e.g., commit 8715800) on the raw fast5 files to remove the sequencing adapter and variable-length poly(A) tail from the start of each signal; with the '--replace' flag the trimmed signal overwrites the Signal slot and the original is copied to Signal_Old so basecallers can read the processed files. Next, extract the transcript signal segment and apply median absolute deviation (MAD) normalization with outlier smoothing to remove gross level shifts and spikes. Slice the normalized signal at the trained input lengths (2 s, 3 s, or 4 s worth of data, roughly the first ~70-280 nt) so it matches the model's expected input length. This trimming is also applied in real time during live runs before the RISER CNN emits target-class probabilities that are thresholded (e.g., T = 0.9) for reject decisions. Correct application is judged by whether basecalling/classification downstream of trimming matches expected accuracy on ground-truth-labeled reads mapped with minimap2 to GENCODE.

## Related tools

- **BoostNano** (Segments and trims the sequencing adapter and poly(A) tail from raw DRS signals before basecalling or model inference) — https://github.com/haotianteng/BoostNano
- **RISER** (Consumes the trimmed, normalized signal slices to classify RNA classes in real time from the first few seconds of signal) — https://github.com/comprna/riser
- **Guppy (v4.0.14)** (Basecalls fast5 files (options –flowcell FLO-MIN106 –kit SQK-RNA002) to derive ground-truth labels for validating preprocessing quality)
- **minimap2 (v2.17)** (Maps basecalled reads to the GENCODE reference transcriptome (release 34, GRCh38.p13, options -ax map-ont --secondary=no) for label assignment after signal processing)
- **samtools (v1.10)** (Filters high-confidence mappings (option -F 2324 removes unmapped and low-confidence alignments) after basecalling trimmed signals)

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- The processed fast5 Signal slot contains trimmed signal and the untrimmed original is recoverable from Signal_Old (BoostNano '--replace' invariant)
- Downstream basecalling + minimap2 mapping of trimmed signals yields high-confidence alignments to GENCODE consistent with expectations (reverse-strand, secondary, supplementary and unmapped reads removed with samtools -F 2324)
- Classifier accuracy on the trimmed reserved test set matches reported performance (e.g., RISER globin model 98% correct on globin mRNA, >99% on non-globin biotypes)
- Trimmed signal segments conform to the trained input lengths (2 s, 3 s, 4 s) before model inference
- MAD-normalized signals show no residual adapter/poly(A) stall features at the start of the transcript segment

## Limitations

- BoostNano only works with Python < 3.9 (tested on 3.7/3.8) and is Linux-targeted (Ubuntu 16.04/18.04/20.04); other platforms are untested
- During MinKNOW playback testing, rejected reads cannot be physically ejected — signal is merely clipped — so effects must be judged by average read length differences rather than ejection
- Accurate classification of some classes (e.g., lncRNA) from the trimmed 3′ end signal remains challenging, and cell type-specific lncRNA expression limits model generalization
- Lag between a reject command and voltage reversal, and reversal efficiency, vary across sequencing platforms and are outside the preprocessing/model pipeline's control

## Evidence

- [methods] the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [methods] Fast5 files were basecalled with Guppy (v4.0.14): "Fast5 files were basecalled with Guppy (v4.0.14) using options –flowcell FLO-MIN106 –kit SQK-RNA002"
- [methods] To trim the sequencing adapter and variable-length poly(A) tail from the start of individual signals during live runs, an efficient strategy was developed: "To trim the sequencing adapter and variable-length poly(A) tail from the start of individual signals during live runs, an efficient strategy was developed"
- [readme] Boostnano only works with Python < 3.9. Tested on Python 3.8 and 3.7.: [evidence span withheld — anchor status: external_doc]
- [readme] RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network, without the need for basecalling or a reference.: "RISER accurately classifies RNA molecules live during sequencing, directly from the first ~70-280nt-worth of raw nanopore signals with a convolutional neural network"

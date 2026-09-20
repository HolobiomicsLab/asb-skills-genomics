---
name: nanopore-signal-preprocessing-adapter-polya-trimming
description: Use when raw nanopore DRS fast5 signals (RNA002/R9.4.1) must have the
  adapter and poly(A) stall trimmed before basecalling (e.g., Chiron/Dorado) or before
  feeding fixed-length inputs to a signal classifier such as RISER.
when_to_use_negative:
- Input is already basecalled FASTQ/BAM — trimming operates on raw signal space in
  fast5, not on sequences
- Data come from DNA sequencing kits or non-RNA nanopore libraries; BoostNano targets
  ONT RNA reads (adapter, polyA stall, transcription start)
- Signals are already trimmed (e.g., previously processed with --replace and no Signal_Old
  to recover) — re-trimming would corrupt the transcript signal
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: BoostNano
  role: Segments/trims the sequencing adapter and poly(A) tail from raw DRS signals
    in fast5 before basecalling or signal classification
  repo: https://github.com/haotianteng/BoostNano
- name: PyTorch
  role: Deep learning framework BoostNano depends on for its segmentation model inference
- name: RISER
  role: Downstream consumer of BoostNano-trimmed signals; benchmark workflow pre-trims
    test signals so RISER and adaptive sampling act on equal inputs
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:510d28dd9dc17c7508e3ef2c8d4b94f53af573c516a95f6ccbefce07e01b100e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/haotianteng/BoostNano
  tools:
  - BoostNano
  - Dorado
  - PyTorch
  - RISER
  - minimap2
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nanopore-signal-preprocessing-adapter-polya-trimming

## Summary

Preprocess Oxford Nanopore direct RNA (DRS) raw signals by segmenting and removing the sequencing adapter and variable-length poly(A) tail from the start of each fast5 signal with BoostNano, so downstream models (e.g., RISER) or basecallers operate on the transcript portion only.

## When to use

Use when raw nanopore DRS fast5 signals (RNA002/R9.4.1) must have the leading adapter and poly(A) stall trimmed before basecalling (e.g., Chiron/Dorado) or before feeding fixed-length inputs to a signal classifier such as RISER.

## When NOT to use

- Input is already basecalled FASTQ/BAM — trimming operates on raw signal space in fast5, not on sequences
- Data come from DNA sequencing kits or non-RNA nanopore libraries; BoostNano targets ONT RNA reads (adapter, polyA stall, transcription start)
- Signals are already trimmed (e.g., previously processed with --replace and no Signal_Old to recover) — re-trimming would corrupt the transcript signal

## Inputs

- Folder of raw nanopore direct RNA sequencing fast5 files (FLO-MIN106, RNA002/R9.4.1 kit)
- BoostNano pretrained model (shipped in boostnano/models)

## Outputs

- fast5 files with the adapter/poly(A)-trimmed signal written to the original Signal slot and the original signal copied to Signal_Old (with --replace)
- Segmentation result table output.csv (when run without --replace)

## How to apply

Run BoostNano's boostnano_eval.py on the folder of raw fast5 files with its shipped pretrained model; use the --replace flag so the trimmed signal overwrites the original Signal slot and the untrimmed signal is preserved in the Signal_Old slot, making the fast5 directly consumable by a basecaller. This removes the sequencing adapter, poly(A) stall, and delineates the transcript start, which is essential because DRS signals begin with a variable-length adapter and poly(A) tail that would confound signal-space classification from the 3' end. In the RISER speed-benchmark workflow, all test signals are pre-trimmed with BoostNano before classifying mRNA vs non-mRNA so that RISER (signal-based) and adaptive sampling (Dorado basecalling + minimap2 -ax map-ont against GENCODE v34) operate on comparable inputs without differing internal trimming. If only segmentation boundaries are needed, omit --replace and read the output.csv segmentation result instead of modifying the fast5 files. Note BoostNano only works with Python < 3.9 (tested on 3.8 and 3.7) and requires PyTorch.

## Related tools

- **BoostNano** (Segments/trims the sequencing adapter and poly(A) tail from raw DRS signals in fast5 before basecalling or signal classification) — https://github.com/haotianteng/BoostNano
- **PyTorch** (Deep learning framework BoostNano depends on for its segmentation model inference)
- **RISER** (Downstream consumer of BoostNano-trimmed signals; benchmark workflow pre-trims test signals so RISER and adaptive sampling act on equal inputs) — https://github.com/comprna/riser

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- With --replace, the fast5 retains the untrimmed signal in the Signal_Old slot while the Signal slot holds the trimmed signal; a basecaller can run directly on the processed files
- Trimmed signals should begin at the transcript 3' portion with adapter/poly(A) stall segments removed, so fixed-length inputs (e.g., RISER's first few seconds of signal) reflect transcript content
- Segmentation boundaries in output.csv should delimit adapter, polyA stall, and transcription regions for each read
- In the RISER-vs-AS benchmark, trimming must be applied to all 1000 test signals before timing so neither method benefits from different internal trimming

## Limitations

- BoostNano only works with Python < 3.9 (tested on 3.8 and 3.7); the RISER pipeline paper used it on Ubuntu 16.04/18.04/20.04
- Using --replace modifies the original fast5 Signal slot; the original signal is only recoverable from the Signal_Old slot
- Trimming must be re-applied (or an equivalent internal strategy used) for live runs; during live RISER runs an efficient in-run strategy trims the adapter and variable-length poly(A) tail from the start of individual signals

## Evidence

- [methods] BoostNano used to trim adapter and poly(A) before RISER: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [readme] BoostNano purpose: [evidence span withheld — anchor status: external_doc]
- [readme] --replace semantics: [evidence span withheld — anchor status: external_doc]
- [methods] RISER live-run trimming strategy: "To trim the sequencing adapter and variable-length poly(A) tail from the start of individual signals during live runs, an efficient strategy was developed"
- [readme] Python version constraint: [evidence span withheld — anchor status: external_doc]

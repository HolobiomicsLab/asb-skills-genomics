---
name: polya-tail-adapter-signal-trimming
description: Use when raw nanopore DRS fast5 signals (SQK-RNA002/RNA004, FLO-MIN106)
  must be stripped of 5'-end adapter and poly(A) tail before signal-based classification,
  training, or live read-until rejection decisions.
when_to_use_negative:
- Input is already basecalled FASTQ/BAM sequence data rather than raw fast5 signal
  — trimming is a signal-space preprocessing step
- Signals have already been trimmed in a previous BoostNano pass (re-running would
  re-segment and the transcript signal has been replaced in the Signal slot)
- The analysis targets adapter or poly(A) tail properties themselves (e.g., tail-length
  estimation) rather than the transcript body signal
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: BoostNano
  role: Segments and trims the sequencing adapter and poly(A) tail from raw DRS signals
    before basecalling or RISER classification; also the tool whose strategy informs
    RISER's live-run trimming
  repo: https://github.com/haotianteng/BoostNano
- name: RISER
  role: 'Downstream consumer of the trimmed signals: applies equivalent in-run adapter/poly(A)
    trimming before classifying the first seconds of raw signal for read-until rejection'
  repo: https://github.com/comprna/riser
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-48673-8
    title: Biochemical-free enrichment or depletion of RNA classes in real-time during
      direct RNA sequencing with RISER
  source_content_hash: sha256:3ff56506dd5c333e3f3f72a4573c53c9e504ea8acec4fc51c5f465e635492026
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
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# polya-tail-adapter-signal-trimming

## Summary

Trims the sequencing adapter and variable-length poly(A) tail from the start of raw ONT direct RNA sequencing (DRS) signals using BoostNano, so that downstream models such as RISER classify only transcript-derived signal. This is required both for offline model development and during live read-until runs.

## When to use

Use when raw nanopore DRS fast5 signals (SQK-RNA002/RNA004, FLO-MIN106) must be stripped of 5'-end adapter and poly(A) tail before signal-based classification, training, or live read-until rejection decisions.

## When NOT to use

- Input is already basecalled FASTQ/BAM sequence data rather than raw fast5 signal — trimming is a signal-space preprocessing step
- Signals have already been trimmed in a previous BoostNano pass (re-running would re-segment and the transcript signal has been replaced in the Signal slot)
- The analysis targets adapter or poly(A) tail properties themselves (e.g., tail-length estimation) rather than the transcript body signal

## Inputs

- Raw ONT direct RNA sequencing fast5 files (FLO-MIN106 / SQK-RNA002 or RNA004)
- BoostNano pretrained segmentation model (shipped in boostnano/models)

## Outputs

- fast5 files with the raw signal slot replaced by the trimmed transcript signal (original preserved in Signal_Old slot)
- Segmentation output CSV (output.csv) with adapter/polyA/transcript boundaries per read

## How to apply

Run BoostNano (from the Chiron project, github.com/haotianteng/BoostNano, Python <3.9 with PyTorch) in inference mode over the raw fast5 files, using its shipped pretrained model; the evaluation script segments the raw signal into adapter, polyA stall, and transcript regions. With the --replace flag, the trimmed transcript signal overwrites the original Signal slot in the fast5 file and the original signal is copied to a Signal_Old slot, so downstream basecallers or signal classifiers (e.g., RISER's CNN in riser/test.py) read only the transcript portion. After trimming, extract the first 4 s (≈70-280 nt) of the remaining transcript signal normalized by median absolute deviation with outlier smoothing as model input. During live sequencing RISER applies an equivalent efficient in-run trimming strategy before prediction, since DRS proceeds from the 3' end so class-discriminative signal sits at the start after the adapter/poly(A). Rationale: leaving adapter/polyA segments in the input confounds the classifier with non-transcript signal and degrades classification accuracy.

## Related tools

- **BoostNano** (Segments and trims the sequencing adapter and poly(A) tail from raw DRS signals before basecalling or RISER classification; also the tool whose strategy informs RISER's live-run trimming) — https://github.com/haotianteng/BoostNano
- **RISER** (Downstream consumer of the trimmed signals: applies equivalent in-run adapter/poly(A) trimming before classifying the first seconds of raw signal for read-until rejection) — https://github.com/comprna/riser

## Examples

```
python boostnano_eval.py -i INPUT_FAST5_FOLDER -o OUTPUT_FOLDER -m MODEL_FOLDER --replace
```

## Evaluation signals

- BoostNano completes inference over all fast5 files without segmentation errors and produces output.csv with per-read boundaries
- With --replace, each fast5's Signal slot contains the trimmed transcript signal and Signal_Old slot retains the untrimmed original
- Trimmed signal lengths are shorter than raw signal lengths by the adapter + poly(A) segment duration; a downstream basecaller (e.g., Guppy v4.0.14) can still run on the processed fast5 files
- RISER model accuracy on HeLa test signals after trimming matches expected performance (≈94% mRNA model, ≈99% mtRNA model), confirming non-transcript signal was removed
- In playback testing, rejected (mRNA) reads show clipped signal lengths consistent with trimming occurring before the reject decision

## Limitations

- BoostNano only works with Python < 3.9 (tested on Python 3.8 and 3.7) and has not been tested on macOS or Windows
- The poly(A) tail is variable-length per read, so a fixed-offset trim cannot be used; per-read segmentation is required
- In playback mode, rejection clips the signal rather than physically ejecting the read, so trim/trim boundaries affect how rejection is observed in signal length

## Evidence

- [methods] the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano: "the sequencing adapter and poly(A) tail were removed from the raw nanopore signals using BoostNano32"
- [methods] To trim the sequencing adapter and variable-length poly(A) tail from the start of individual signals during live runs, an efficient strategy was developed: "To trim the sequencing adapter and variable-length poly(A) tail from the start of individual signals during live runs, an efficient strategy was developed"
- [readme] BOOSTNANO is a tool for preprocessing ONT-Nanopore RNA sequencing reads(before basecalling), it segments/trims the adapter, polyA stall, and the transcription from the raw signal: [evidence span withheld — anchor status: external_doc]
- [readme] This will save the segmented siganl in to the original Signal Slot in the fast5 files, and copy the old signal into the Signal_Old slot: [evidence span withheld — anchor status: external_doc]

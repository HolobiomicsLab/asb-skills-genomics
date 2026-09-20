---
name: nanopore-signal-zscore-normalization
description: Use when preparing raw nanopore signal reads (e.g., BLOW5 format from
  R9.4 chemistry) for signal-domain alignment such as sDTW-based Read Until, where
  events must be normalized and cast to 16-bit fixed point before dispatch to an accelerator.
when_to_use_negative:
- Input is already basecalled FASTQ/BAM destined for base-domain Read Until — HARU's
  signal-domain pipeline explicitly avoids the basecalling prerequisite step.
- Signal is from non-R9.4 chemistry, since the ~250-event budget and ~50 read-adapter
  prefix trim are calibrated to R9.4 flowcells.
- The normalized events need full-precision float downstream rather than the signed
  16-bit fixed-point representation (scaling factor 32) consumed by the sDTW accelerator.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HARU
  role: Provides the software preprocessing layer that performs event detection, prefix
    trimming, z-score normalization, and 16-bit fixed-point scaling before dispatch
    to the sDTW accelerator
  repo: https://github.com/beebdev/HARU
- name: RUscripts-R9
  role: Supplies the R9.4 chemistry nanopore model used during adaptive event detection
    prior to normalization
  repo: https://github.com/beebdev/RUscripts-R9
- name: Sigfish-HARU
  role: Demonstrates the proof-of-concept integration that consumes the preprocessed
    events for squiggle mapping against nCoV-2019.reference.fasta
  repo: https://github.com/beebdev/sigfish-haru
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:722a87435eaf8c0d3feeabfa3977ac99434ab657285ab66657f3e50f6aa545af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/HARU
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# nanopore-signal-zscore-normalization

## Summary

Normalizes ONT nanopore raw-signal events (after event detection and prefix trimming) using z-score standardization and then rescales to signed 16-bit fixed point with scaling factor 32, as implemented in HARU's software preprocessing layer ahead of sDTW squiggle mapping.

## When to use

Use when preparing raw nanopore signal reads (e.g., BLOW5 format from R9.4 chemistry) for signal-domain alignment such as sDTW-based Read Until, where events must be normalized and cast to 16-bit fixed point before dispatch to an accelerator.

## When NOT to use

- Input is already basecalled FASTQ/BAM destined for base-domain Read Until — HARU's signal-domain pipeline explicitly avoids the basecalling prerequisite step.
- Signal is from non-R9.4 chemistry, since the ~250-event budget and ~50 read-adapter prefix trim are calibrated to R9.4 flowcells.
- The normalized events need full-precision float downstream rather than the signed 16-bit fixed-point representation (scaling factor 32) consumed by the sDTW accelerator.

## Inputs

- Raw nanopore signal reads in BLOW5 format (e.g., SARS-CoV-2 reads_0_0.blow5, Zenodo DOI 10.5281/zenodo.7314838)
- R9.4 pore model parameters (bundled with RUscripts-R9) for adaptive event detection

## Outputs

- Per-read event arrays of z-score normalized, fixed-point-scaled values as signed 16-bit integers (scaling factor 32) in binary/TSV form
- Per-read event counts table (target ~250 events/read for R9.4, after trimming ~50 prefix events)

## How to apply

First run adaptive event detection on raw signal traces loaded from a BLOW5 file using slow5lib tooling in a multithreaded batch loop, targeting ~250 events per read for R9.4 chemistry (roughly 0.4–0.8 s of data collection, including ~50–300 read-adapter events). Trim the read prefix by removing the first ~50 adapter-associated events so the remaining event means/stds reflect the pore-current region. Apply z-score normalization to the trimmed event means and standard deviations using the R9.4 pore model bundled with RUscripts-R9. Finally, scale the normalized values to fixed point with scaling factor 32 and cast to signed 16-bit integers, writing the scaled event arrays (plus per-read event counts) for handoff to the sDTW accelerator (as demonstrated via sigfish-haru).

## Related tools

- **HARU** (Provides the software preprocessing layer that performs event detection, prefix trimming, z-score normalization, and 16-bit fixed-point scaling before dispatch to the sDTW accelerator) — https://github.com/beebdev/HARU
- **RUscripts-R9** (Supplies the R9.4 chemistry nanopore model used during adaptive event detection prior to normalization) — https://github.com/beebdev/RUscripts-R9
- **Sigfish-HARU** (Demonstrates the proof-of-concept integration that consumes the preprocessed events for squiggle mapping against nCoV-2019.reference.fasta) — https://github.com/beebdev/sigfish-haru

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Per-read event counts fall in the range 240–260 (reported ~250 for R9.4 chemistry).
- Output values are present, signed 16-bit, and within fixed-point bounds after scaling with factor 32.
- Each read's event array is shortened by ~50 prefix (read-adapter) events relative to the raw detected event count.
- The preprocessed output file exists and the per-read event-count table is populated for every read queried.
- Downstream sigfish-haru dtw produces a valid PAF alignment output (e.g., output.paf) against nCoV-2019.reference.fasta.

## Limitations

- Event budget and prefix-trim length (~250 events target, ~50 adapter events) are specific to R9.4 chemistry; other chemistries require recalibration.
- Collecting ~250 events per read query takes roughly 0.4–0.8 seconds of data collection, which constrains real-time latency on the MinION.
- Fixed-point scaling (factor 32, 16-bit) sacrifices precision relative to floating-point normalization.

## Evidence

- [other] z-score normalization of trimmed events: [evidence span withheld — anchor status: not_found]
- [other] fixed-point scaling to 16-bit: [evidence span withheld — anchor status: not_found]
- [other] R9.4 event budget per read: [evidence span withheld — anchor status: not_found]
- [other] prefix trimming of adapter events: [evidence span withheld — anchor status: not_found]
- [intro] signal-domain context for normalization: "HARU tackles the computational bottleneck by accelerating the sDTW algorithm with field-programmable gate arrays (FPGAs)"

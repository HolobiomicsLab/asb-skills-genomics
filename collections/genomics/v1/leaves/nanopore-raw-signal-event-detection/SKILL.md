---
name: nanopore-raw-signal-event-detection
description: Use when raw nanopore signal reads (e.g., SARS-CoV-2 BLOW5 from Zenodo
  10.5281/zenodo.7314838) must be converted into per-read event sequences—event detection,
  prefix trimming, z-score normalization, and scaling—before squiggle mapping or sDTW-based
  Read Until on R9.4 chemistry.
when_to_use_negative:
- Reads are already basecalled and the pipeline operates in the base domain (base-domain
  Read Until uses Minimap2-style mapping, not signal event detection).
- The nanopore chemistry is not R9.4 — the ~250-events-per-read budget and R9.4 pore
  model from RUscripts-R9 would not apply.
- Input is a FASTA/FASTQ reference or base sequence rather than raw signal reads (BLOW5);
  event detection requires the raw squiggle.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: Sigfish-HARU
  role: Proof-of-concept application that performs event detection on raw signal and
    dispatches preprocessed events to the HARU sDTW accelerator for squiggle mapping
  repo: github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Modified Read Until scripts providing the R9.4 pore model and adaptive event
    detection configuration
  repo: github.com/beebdev/RUscripts-R9
- name: HARU
  role: sDTW hardware accelerator and driver that consumes the preprocessed fixed-point
    event arrays
  repo: github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:8e0c0a49595c3e478497467437042d237dc09610c63878b51c15717405e16947
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# nanopore-raw-signal-event-detection

## Summary

Adaptive event detection and preprocessing of raw ONT nanopore signal reads (BLOW5) into ~250-event normalized, fixed-point arrays ready for sDTW squiggle alignment in the HARU Read Until pipeline.

## When to use

Use when raw nanopore signal reads (e.g., SARS-CoV-2 BLOW5 from Zenodo 10.5281/zenodo.7314838) must be converted into per-read event sequences—event detection, prefix trimming, z-score normalization, and scaling—before squiggle mapping or sDTW-based Read Until on R9.4 chemistry.

## When NOT to use

- Reads are already basecalled and the pipeline operates in the base domain (base-domain Read Until uses Minimap2-style mapping, not signal event detection).
- The nanopore chemistry is not R9.4 — the ~250-events-per-read budget and R9.4 pore model from RUscripts-R9 would not apply.
- Input is a FASTA/FASTQ reference or base sequence rather than raw signal reads (BLOW5); event detection requires the raw squiggle.

## Inputs

- Raw signal reads in BLOW5 format (e.g., SARS-CoV-2 reads_0_0.blow5, Zenodo DOI 10.5281/zenodo.7314838)
- ONT R9.4 pore model bundled with RUscripts-R9
- Reference genome FASTA (nCoV-2019.reference.fasta) if feeding downstream sDTW squiggle mapping

## Outputs

- Per-read event arrays scaled to signed 16-bit fixed point (scaling factor 32), written as binary/TSV
- Per-read event count table (target ~250 events/read for R9.4)
- Downstream: PAF alignment output from sigfish-haru squiggle mapping

## How to apply

Load raw signal reads from the BLOW5 file using slow5lib/BLOW5 tooling inside a multithreaded batch loop, then run adaptive event detection on each raw trace using the R9.4 pore model bundled with RUscripts-R9, targeting ~250 events per read for R9.4 chemistry (this typically requires roughly 0.4–0.8 s of data collection including ~50–300 read-adapter events). Trim the read prefix by removing the first ~50 adapter-associated events from each event array, then apply z-score normalization to the trimmed event means and standard deviations. Scale the normalized values to fixed point with scaling factor 32 and cast to signed 16-bit integers; the rationale is to emit a compact fixed-point event array that the HARU sDTW FPGA accelerator (e.g., on the Kria KV260 via sigfish-haru) can consume efficiently. Write the 16-bit scaled event arrays to a binary/TSV file and record per-read event counts in a separate table.

## Related tools

- **Sigfish-HARU** (Proof-of-concept application that performs event detection on raw signal and dispatches preprocessed events to the HARU sDTW accelerator for squiggle mapping) — github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Modified Read Until scripts providing the R9.4 pore model and adaptive event detection configuration) — github.com/beebdev/RUscripts-R9
- **HARU** (sDTW hardware accelerator and driver that consumes the preprocessed fixed-point event arrays) — github.com/beebdev/HARU

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Per-read event counts fall in the range 240–260 (reported ~250 for R9.4 chemistry).
- Scaled event values are present and within signed 16-bit fixed-point bounds after applying scaling factor 32.
- Trimmed event arrays are z-score normalized (event means/standard deviations) before scaling.
- Read prefix trimmed: the first ~50 adapter-associated events are removed from each event array.
- Downstream sanity check: sigfish-haru dtw run over the preprocessed BLOW5 emits a non-empty output.paf.

## Limitations

- Event budget (~250 events/read) is specific to R9.4 chemistry; other chemistries require a different pore model and target.
- Query preparation (event collection) typically takes roughly 0.4–0.8 seconds of data per read query, bounding real-time Read Until responsiveness.
- Building/running the downstream HARU accelerator requires proprietary AMD Xilinx tooling (Vivado/Vitis 2021.1) and Kria KV260 hardware.

## Evidence

- [methods] 250 events per read for R9.4 chemistry: [evidence span withheld — anchor status: not_found]
- [methods] prefix trimming and normalization: [evidence span withheld — anchor status: not_found]
- [methods] fixed-point scaling: [evidence span withheld — anchor status: not_found]
- [readme] context of Read Until: "HARU is a heterogenous compute solution for Oxford Nanopore Technologies' adaptive sampling (also known as selective sequencing and [Read Until]"
- [readme] example invocation: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"

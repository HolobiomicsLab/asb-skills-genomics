---
name: adapter-prefix-event-trimming
description: Use when preparing raw ONT signal reads (BLOW5/SLOW5 format) for signal-domain
  Read Until or squiggle mapping, specifically when you need event arrays of ~250
  events per R9.4 read with the adapter prefix removed and int16-scaled values for
  an sDTW accelerator.
when_to_use_negative:
- Input is already basecalled sequence (FASTQ/BAM) rather than raw signal — base-domain
  Read Until does not need event detection or sDTW preprocessing
- The chemistry or pore model is not R9.4 — the ~250-event budget and RUscripts-R9
  model parameters are specific to R9.4 flowcells
- You only need downstream alignment output and the HARU/sigfish-haru pipeline already
  performs preprocessing internally
tools:
- name: Sigfish-HARU
  role: Demonstrates integration of the HARU accelerator with the preprocessing pipeline;
    reads BLOW5 signal data and runs the dtw squiggle-mapping workflow
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Modified RUscripts supporting Python 3.6+, BLOW5 format, and R9.4 chemistry;
    provides the adaptive event detection used for preprocessing
  repo: https://github.com/beebdev/RUscripts-R9
- name: HARU
  role: sDTW hardware accelerator (Verilog HDL core and user-space device driver)
    that receives the preprocessed event arrays
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:e91694adf5e1234d2f59fb1edce49fd0c513cfb0e0a7ff01a41dfae6900b9684
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# adapter-prefix-event-trimming

## Summary

Reconstructs HARU's software preprocessing layer for nanopore raw signal reads: adaptive event detection on BLOW5 signal data, trimming of the read-adapter event prefix, z-score normalization, and fixed-point scaling (factor 32 to int16) before dispatch to the sDTW accelerator.

## When to use

Use when preparing raw ONT signal reads (BLOW5/SLOW5 format) for signal-domain Read Until or squiggle mapping, specifically when you need event arrays of ~250 events per R9.4 read with the adapter prefix removed and int16-scaled values for an sDTW accelerator.

## When NOT to use

- Input is already basecalled sequence (FASTQ/BAM) rather than raw signal — base-domain Read Until does not need event detection or sDTW preprocessing
- The chemistry or pore model is not R9.4 — the ~250-event budget and RUscripts-R9 model parameters are specific to R9.4 flowcells
- You only need downstream alignment output and the HARU/sigfish-haru pipeline already performs preprocessing internally

## Inputs

- Raw nanopore signal reads in BLOW5 format (e.g. reads_0_0.blow5)
- R9.4 pore model parameters from RUscripts-R9
- Reference FASTA for downstream squiggle mapping (e.g. nCoV-2019.reference.fasta)

## Outputs

- Preprocessed 16-bit fixed-point scaled event arrays written to binary/TSV
- Per-read event count table
- Query event arrays dispatched to the sDTW accelerator via sigfish-haru

## How to apply

Load raw signal reads from a SARS-CoV-2 BLOW5 file (Zenodo DOI 10.5281/zenodo.7314838) via slow5lib tooling in a multithreaded batch loop, then run adaptive event detection using the R9.4 pore model bundled with RUscripts-R9, targeting ~250 events per read query (typically 0.4–0.8 seconds of data collection, including ~50–300 read-adapter events). Trim the first ~50 adapter-associated events from each event array to remove the read-adapter prefix. Apply z-score normalization to the trimmed event means and standard deviations, then scale to fixed point with factor 32 and cast to signed 16-bit integers before writing the preprocessed arrays (binary/TSV) and per-read event counts for dispatch to the sDTW accelerator.

## Related tools

- **Sigfish-HARU** (Demonstrates integration of the HARU accelerator with the preprocessing pipeline; reads BLOW5 signal data and runs the dtw squiggle-mapping workflow) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Modified RUscripts supporting Python 3.6+, BLOW5 format, and R9.4 chemistry; provides the adaptive event detection used for preprocessing) — https://github.com/beebdev/RUscripts-R9
- **HARU** (sDTW hardware accelerator (Verilog HDL core and user-space device driver) that receives the preprocessed event arrays) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Per-read event counts fall in the range 240–260 (~250 events reported for R9.4 chemistry)
- Scaled event values are present and within signed 16-bit fixed-point bounds (scaling factor 32)
- Trimmed event arrays have the first ~50 adapter-associated events removed relative to the detected event arrays
- Output files exist: the binary/TSV scaled event arrays and the separate per-read event count table
- sigfish-haru dtw on test_data/reads_0_0.blow5 against nCoV-2019.reference.fasta produces a valid output.paf

## Limitations

- The event budget (~250 events) and adapter-prefix trim (~50 events) are calibrated for R9.4 chemistry; other flowcell chemistries are not covered
- Building the HARU core accelerator is not intuitive and requires proprietary AMD Xilinx software (Vivado/PetaLinux, 2021.1 or compatible versions)
- The hardware path targets a Xilinx Kria AI Starter Kit (Zynq Ultrascale+ MPSoC); other platforms are unsupported in the proof-of-concept

## Evidence

- [full_text] event detection with adapter-prefix trimming and 250-event budget: [evidence span withheld — anchor status: not_found]
- [full_text] trimming, normalization, scaling parameters: [evidence span withheld — anchor status: not_found]
- [methods] example pipeline invocation: "./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf"
- [discussion] tooling supports R9.4 chemistry and BLOW5 format: [evidence span withheld — anchor status: not_found]

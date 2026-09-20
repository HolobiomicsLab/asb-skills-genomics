---
name: fixed-point-16bit-signal-scaling
description: Use when preparing ONT raw-signal event arrays (e.g. BLOW5 reads, R9.4
  chemistry, ~250 events/read) for a fixed-point hardware aligner such as the HARU
  sDTW accelerator, or when reproducing HARU's preprocessing pipeline.
when_to_use_negative:
- Input signals are destined for a float-based software aligner (e.g. sigfish-cpu)
  rather than the fixed-point sDTW hardware accelerator
- Reads come from a different pore chemistry whose target event budget differs from
  the ~250-event R9.4 configuration
- The data are already basecalled FASTQ/base-domain sequences — this skill operates
  in the raw-signal/event domain only
tools:
- name: Sigfish-HARU
  role: Example application implementing the preprocessing (event detection, normalization,
    16-bit fixed-point scaling) that feeds the sDTW accelerator
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Provides the R9.4 pore model and scripts used for adaptive event detection
    prior to scaling
  repo: https://github.com/beebdev/RUscripts-R9
- name: HARU
  role: sDTW accelerator consuming the 16-bit fixed-point scaled event values
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:c7e3129e162bbdf448690ee6ae7658b221aae9630e0aa66cf159639f60a209b6
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# fixed-point-16bit-signal-scaling

## Summary

Scales z-score-normalized nanopore event values to signed 16-bit fixed point with a scaling factor of 32, as done in HARU's software preprocessing layer so that event means and standard deviations can be fed to the sDTW hardware accelerator with minimal memory footprint.

## When to use

Use when preparing ONT raw-signal event arrays (e.g. BLOW5 reads, R9.4 chemistry, ~250 events/read) for a fixed-point hardware aligner such as the HARU sDTW accelerator, or when reproducing HARU's preprocessing pipeline.

## When NOT to use

- Input signals are destined for a float-based software aligner (e.g. sigfish-cpu) rather than the fixed-point sDTW hardware accelerator
- Reads come from a different pore chemistry whose target event budget differs from the ~250-event R9.4 configuration
- The data are already basecalled FASTQ/base-domain sequences — this skill operates in the raw-signal/event domain only

## Inputs

- Raw signal reads from a BLOW5 file (e.g. SARS-CoV-2 reads_0_0.blow5, Zenodo DOI 10.5281/zenodo.7314838)
- Z-score-normalized event means and standard deviations (~250 events per R9.4 read, after ~50 adapter-event prefix trimming)

## Outputs

- Signed 16-bit fixed-point scaled event arrays (scaling factor 32) written to a binary/TSV file
- Per-read event counts recorded in a separate table

## How to apply

First run adaptive event detection on raw signal samples in multithreaded batches (Sigfish-HARU with the R9.4 pore model from RUscripts-R9), targeting ~250 events per read, and trim the first ~50 adapter-associated prefix events. Then apply z-score normalization to the trimmed event means and standard deviations. Finally scale the normalized values by a fixed-point scaling factor of 32 and cast to signed 16-bit integers before writing the event arrays to a binary/TSV file. The 16-bit fixed-point representation is what makes the signal streams compatible with the sDTW accelerator's low-memory design on the Kria KV260 Zynq Ultrascale+ MPSoC.

## Related tools

- **Sigfish-HARU** (Example application implementing the preprocessing (event detection, normalization, 16-bit fixed-point scaling) that feeds the sDTW accelerator) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Provides the R9.4 pore model and scripts used for adaptive event detection prior to scaling) — https://github.com/beebdev/RUscripts-R9
- **HARU** (sDTW accelerator consuming the 16-bit fixed-point scaled event values) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Per-read event counts fall in the 240–260 range (target ~250 for R9.4 chemistry)
- Scaled values are present and within signed 16-bit fixed-point bounds after applying scaling factor 32
- First ~50 adapter-associated prefix events removed before normalization/scaling
- Downstream sigfish-haru dtw run on the scaled events produces a valid PAF output (e.g. output.paf against nCoV-2019.reference.fasta)

## Limitations

- Scaling factor 32 and 16-bit cast are tuned for the HARU sDTW accelerator on Kria KV260; other aligners may expect different precision
- Prefix trimming removes ~50 adapter-associated events, so reads shorter than the adapter plus ~200 additional events may not meet the 240–260 event budget
- The R9.4 pore model assumptions (RUscripts-R9) mean the ~250-event target does not transfer to other ONT chemistries

## Evidence

- [results] HARU's software processing layer performs event detection on raw signal samples in multithreaded batches, collecting 250 events per read query for the R9.4 flowcell: [evidence span withheld — anchor status: not_found]
- [methods] Fixed-point scaling step of the HARU preprocessing workflow: [evidence span withheld — anchor status: not_found]
- [readme] HARU as a heterogeneous compute solution for Read Until: "HARU is a heterogenous compute solution for Oxford Nanopore Technologies' adaptive sampling (also known as selective sequencing and [Read Until]"
- [methods] Prefix trimming step preceding normalization and scaling: [evidence span withheld — anchor status: not_found]

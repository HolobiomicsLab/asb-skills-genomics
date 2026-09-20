---
name: blow5-read-batch-processing
description: Use when raw ONT signal reads in BLOW5 (or SLOW5) format must be converted
  into fixed-point event arrays as input to a squiggle-mapping/sDTW pipeline, particularly
  R9.4 chemistry Read Until workflows.
when_to_use_negative:
- Input reads are already basecalled FASTQ/BAM — signal-domain event preprocessing
  is unnecessary; use base-domain Read Until instead
- The downstream aligner expects raw squiggle samples rather than fixed-point event
  arrays
- Chemistry is not R9.4 — the ~250-event budget and R9.4 pore model parameters would
  not apply
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Sigfish-HARU
  role: Proof-of-concept application integrating the HARU accelerator; contains the
    software preprocessing path that reads BLOW5 batches and emits preprocessed event
    arrays for sDTW
  repo: https://github.com/beebdev/sigfish-haru
- name: RUscripts-R9
  role: Provides the modified Read Until scripts supporting Python 3.6+, BLOW5 format,
    and ONT R9.4 chemistry, including the pore model used for adaptive event detection
  repo: https://github.com/beebdev/RUscripts-R9
- name: HARU
  role: Consumes the 16-bit scaled event arrays for hardware sDTW squiggle mapping
    on the Kria FPGA
  repo: https://github.com/beebdev/HARU
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giad046
    title: Efficient real-time selective genome sequencing on resource-constrained
      devices
  source_content_hash: sha256:318cc22612a721cd7e1f35cc75f12d661f77036aea6a31a4c094c8135edd258c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beebdev/sigfish-haru
  tools:
  - HARU
  - RUscripts-R9
  - Sigfish-HARU
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# blow5-read-batch-processing

## Summary

Multithreaded software preprocessing of raw nanopore signal reads stored in BLOW5 format: adaptive event detection tuned for R9.4 chemistry (~250 events/read), prefix trimming, z-score normalization, and fixed-point scaling to signed 16-bit integers before dispatch to the HARU sDTW accelerator.

## When to use

Use when raw ONT signal reads in BLOW5 (or SLOW5) format must be converted into fixed-point event arrays as input to a squiggle-mapping/sDTW pipeline, particularly R9.4 chemistry Read Until workflows.

## When NOT to use

- Input reads are already basecalled FASTQ/BAM — signal-domain event preprocessing is unnecessary; use base-domain Read Until instead
- The downstream aligner expects raw squiggle samples rather than fixed-point event arrays
- Chemistry is not R9.4 — the ~250-event budget and R9.4 pore model parameters would not apply

## Inputs

- Raw nanopore signal reads in BLOW5 format (e.g., reads_0_0.blow5 from the SARS-CoV-2 dataset, Zenodo DOI 10.5281/zenodo.7314838)
- R9.4 pore model parameters from RUscripts-R9

## Outputs

- Per-read event arrays of trimmed, z-score normalized, scaled signed 16-bit event values (binary/TSV)
- Per-read event counts table (target ~250 events for R9.4)

## How to apply

Load raw signal reads from a BLOW5 file (e.g., the SARS-CoV-2 dataset at Zenodo DOI 10.5281/zenodo.7314838) using slow5lib tooling inside a multithreaded batch loop. Run adaptive event detection on each raw trace with the R9.4 pore model from RUscripts-R9, targeting ~250 events per read for the R9.4 flowcell (typically 0.4–0.8 s of signal data, including ~50–300 read-adapter events). Trim the first ~50 adapter-associated events from each event array, then apply z-score normalization to the trimmed event means and standard deviations. Scale the normalized values to fixed point with scaling factor 32 and cast to signed 16-bit integers so they can be dispatched to the sDTW hardware accelerator.

## Related tools

- **Sigfish-HARU** (Proof-of-concept application integrating the HARU accelerator; contains the software preprocessing path that reads BLOW5 batches and emits preprocessed event arrays for sDTW) — https://github.com/beebdev/sigfish-haru
- **RUscripts-R9** (Provides the modified Read Until scripts supporting Python 3.6+, BLOW5 format, and ONT R9.4 chemistry, including the pore model used for adaptive event detection) — https://github.com/beebdev/RUscripts-R9
- **HARU** (Consumes the 16-bit scaled event arrays for hardware sDTW squiggle mapping on the Kria FPGA) — https://github.com/beebdev/HARU

## Examples

```
./sigfish-haru dtw -g test_data/nCoV-2019.reference.fasta -s test_data/reads_0_0.blow5 > output.paf
```

## Evaluation signals

- Per-read event counts fall in the 240–260 range (reported ~250 events per read query for R9.4)
- Scaled event values are present and within signed 16-bit fixed-point bounds (scaling factor 32)
- Prefix trimming removed the first ~50 adapter-associated events from each event array
- Normalized event means and standard deviations are z-scored before scaling
- Preprocessed output file and per-read event-count table both exist and are populated for every read in the batch

## Limitations

- Event budget and trimming constants (~250 events, ~50 prefix events) are specific to R9.4 chemistry and the R9.4 flowcell; other chemistries need re-tuning
- Event detection requires roughly 0.4–0.8 s of signal data collection per read query, bounding how early in a read a Read Until decision can be made
- Fixed-point scaling to 16 bits with factor 32 trades dynamic range for accelerator memory efficiency

## Evidence

- [other] R9.4 event budget: [evidence span withheld — anchor status: not_found]
- [other] trimming and normalization: [evidence span withheld — anchor status: not_found]
- [other] fixed-point scaling: [evidence span withheld — anchor status: not_found]
- [readme] sigfish-haru role: "an example application called *sigfish-haru* for which the source code is available"
- [discussion] RUscripts-R9 scope: [evidence span withheld — anchor status: not_found]

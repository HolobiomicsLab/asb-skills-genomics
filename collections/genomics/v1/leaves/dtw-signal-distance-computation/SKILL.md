---
name: dtw-signal-distance-computation
description: Use when benchmarking nanopore signal simulators against real ONT R10.4.1
  DNA or RNA004 data and a raw-signal fidelity metric is needed beyond per-read sequence
  accuracy.
when_to_use_negative:
- Input consists only of basecalled FASTQ reads with no raw signal available, since
  DTW requires signal chunks
- Comparing reads against a reference sequence for accuracy/error-rate metrics — use
  minimap2 alignment identity instead of signal DTW
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Generates the simulated POD5 signal chunks whose DTW distance to real chunks
    is computed
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: seq2squiggle
  role: Competing simulator whose POD5 signals serve as DTW benchmark comparisons
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Dorado
  role: Basecalls the simulated signals so signal-level DTW can be reported alongside
    basecalling accuracy
  repo: https://github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to references to provide the companion per-read accuracy
    metrics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:8119640d9ee69099a190ffbac9ae00245776682266a0ae9b250028a875fec3f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Squigulator
  - minimap2
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# dtw-signal-distance-computation

## Summary

Computes Dynamic Time Warping (DTW) distances between simulated and real nanopore signal chunks to quantify signal-level fidelity of simulators such as NanoSimFormer, seq2squiggle, and Squigulator, complementing basecalling accuracy benchmarks.

## When to use

Use when benchmarking nanopore signal simulators against real ONT R10.4.1 DNA or RNA004 data and a raw-signal fidelity metric is needed beyond per-read sequence accuracy.

## When NOT to use

- Input consists only of basecalled FASTQ reads with no raw signal available, since DTW requires signal chunks
- Comparing reads against a reference sequence for accuracy/error-rate metrics — use minimap2 alignment identity instead of signal DTW

## Inputs

- Simulated nanopore signal files (POD5) from NanoSimFormer, seq2squiggle, or Squigulator
- Real experimental ONT signal chunks (POD5) from datasets such as HG002 chr22, HG001-HG005, bacterial genomes, MSA-1010, or HG002-RNA004

## Outputs

- Per-chunk DTW distances between simulated and real signals
- Mean DTW distance aggregated per simulator and dataset (expected 3.98-6.75 for NanoSimFormer HG001-HG005)
- Consolidated benchmark rows for simulation_benchmark_results.csv

## How to apply

Generate simulated signals (POD5) for each reference using NanoSimFormer (--preset ont_r1041_dna_5khz for DNA, ont_rna004_4khz for direct RNA), plus comparison POD5 sets from seq2squiggle v0.3.4 and Squigulator v0.4.0. Extract signal chunks and compute DTW distances between simulated chunks and corresponding real experimental signal chunks. Aggregate mean DTW per simulator per dataset; for NanoSimFormer on the five human samples (HG001-HG005), expected mean DTW distances fall within 3.98-6.75. Interpret lower DTW as closer waveform-level agreement, and report it alongside per-read accuracy and median Q-scores to disentangle sequence-error fidelity from raw-signal shape fidelity.

## Related tools

- **NanoSimFormer** (Generates the simulated POD5 signal chunks whose DTW distance to real chunks is computed) — https://github.com/BioinfoSZU/NanoSimFormer
- **seq2squiggle** (Competing simulator whose POD5 signals serve as DTW benchmark comparisons) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Dorado** (Basecalls the simulated signals so signal-level DTW can be reported alongside basecalling accuracy) — https://github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to references to provide the companion per-read accuracy metrics)

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --sample-reads 1000 --gpu 0 --preset ont_r1041_dna_5khz --basecall
```

## Evaluation signals

- NanoSimFormer HG001-HG005 mean DTW distances fall within the reported range 3.98-6.75
- DTW values are reported per simulator and dataset, allowing ordering of NanoSimFormer vs seq2squiggle vs Squigulator fidelity
- DTW results are consistent with the companion accuracy ordering (NanoSimFormer 98.74-99.65% median accuracy vs lower values for competitors)
- Aggregated metrics are written to simulation_benchmark_results.csv with per-simulator, per-dataset rows

## Limitations

- DTW is a signal-shape metric and must be interpreted jointly with basecalling accuracy; a low DTW alone does not guarantee correct basecalled sequences
- DTW benchmarks depend on the preset used (ont_r1041_dna_5khz vs ont_rna004_4khz) and cannot be directly compared across chemistries

## Evidence

- [methods] DTW distances between simulated and real signal chunks: [evidence span withheld — anchor status: not_found]
- [methods] DTW validation range: [evidence span withheld — anchor status: not_found]
- [readme] NanoSimFormer signal generation presets: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads."
- [methods] Dorado basecalling integration: [evidence span withheld — anchor status: not_found]

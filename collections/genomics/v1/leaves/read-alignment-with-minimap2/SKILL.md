---
name: read-alignment-with-minimap2
description: Use when per-read accuracy, median Q-score, or per-read mismatch/insertion/deletion
  rates must be computed from basecalled ONT reads by mapping them to a known reference
  genome (e.g., HG002 chr22, bacterial genomes) or transcriptome.
when_to_use_negative:
- Input is raw POD5 signal that has not yet been basecalled — Dorado basecalling must
  precede alignment
- The analysis goal is signal-level fidelity (Dynamic Time Warping distances between
  simulated and real signal chunks) rather than sequence-level accuracy — alignment
  accuracy metrics do not capture signal fidelity
- The simulator output has no known reference against which per-read identity can
  be computed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Long-read aligner used to map all basecalled reads to their references to
    obtain per-read alignments for accuracy and error-rate computation
- name: Dorado
  role: Basecaller that converts simulated or real POD5 signals into FASTQ reads prior
    to minimap2 alignment (v5.0.0 in the benchmark)
  repo: github.com/nanoporetech/dorado
- name: NanoSimFormer
  role: Transformer-based nanopore signal simulator whose basecalled reads (via --basecall)
    are the primary alignment inputs being benchmarked
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: seq2squiggle
  role: Competing simulator v0.3.4 whose signals are basecalled and aligned to provide
    comparison medians (93.34-97.25%)
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:e3b16dc38f3854a2a9e464e5295c0d05ea40ed5fc0ad7d927919ff7b8186b084
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Squigulator
  - minimap2
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# read-alignment-with-minimap2

## Summary

Align Dorado-basecalled nanopore reads (from real data or NanoSimFormer/seq2squiggle/Squigulator simulated POD5 signals) to their reference genomes or transcriptomes with minimap2, yielding the per-read alignments required to compute per-read accuracies, Q-scores, and mismatch/insertion/deletion error profiles in the simulator benchmark.

## When to use

Use when per-read accuracy, median Q-score, or per-read mismatch/insertion/deletion rates must be computed from basecalled ONT reads by mapping them to a known reference genome (e.g., HG002 chr22, bacterial genomes) or transcriptome.

## When NOT to use

- Input is raw POD5 signal that has not yet been basecalled — Dorado basecalling must precede alignment
- The analysis goal is signal-level fidelity (Dynamic Time Warping distances between simulated and real signal chunks) rather than sequence-level accuracy — alignment accuracy metrics do not capture signal fidelity
- The simulator output has no known reference against which per-read identity can be computed

## Inputs

- Basecalled FASTQ reads from Dorado (derived from simulated POD5 or real nanopore signals)
- Reference genome FASTA (e.g., HG002 chr22, HG001/HG003-HG005, five bacterial genomes, MSA-1010)
- Reference transcriptome or 3-column transcript profile TSV for RNA004 datasets
- Alignment output from minimap2 (SAM/BAM/PAF per-read alignments)

## Outputs

- Per-read alignments against the reference (minimap2 output)
- Per-read accuracy and median Q-score metrics
- Per-read mismatch/insertion/deletion error-rate table
- Consolidated benchmark CSV (e.g., simulation_benchmark_results.csv)

## How to apply

First basecall the signal sets (simulated POD5 from NanoSimFormer --preset ont_r1041_dna_5khz or ont_rna004_4khz, or real data) into FASTQ reads with Dorado v5.0.0, using NanoSimFormer's --basecall option which invokes Dorado automatically. Then align all basecalled reads to the same reference used for simulation with minimap2 in ONT-appropriate mode to obtain per-read alignments. From the alignments, compute per-read accuracies and median Q-scores plus per-read mismatch/insertion/deletion error rates, and aggregate median metrics per simulator and dataset. Judge correctness against the benchmark targets: NanoSimFormer DNA median read accuracies of 98.74-99.65%, median Q 23.3-24.7, error rates of mismatch 0.57% / insertion 0.28% / deletion 0.42%, while seq2squiggle (93.34-97.25%) and Squigulator (89.98-96.70%) yield lower medians. Write the consolidated per-simulator, per-dataset table (e.g., simulation_benchmark_results.csv) as the final deliverable.

## Related tools

- **minimap2** (Long-read aligner used to map all basecalled reads to their references to obtain per-read alignments for accuracy and error-rate computation)
- **Dorado** (Basecaller that converts simulated or real POD5 signals into FASTQ reads prior to minimap2 alignment (v5.0.0 in the benchmark)) — github.com/nanoporetech/dorado
- **NanoSimFormer** (Transformer-based nanopore signal simulator whose basecalled reads (via --basecall) are the primary alignment inputs being benchmarked) — https://github.com/BioinfoSZU/NanoSimFormer
- **seq2squiggle** (Competing simulator v0.3.4 whose signals are basecalled and aligned to provide comparison medians (93.34-97.25%)) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
minimap2 -ax map-ont ref.fasta basecalled.fastq > aln.sam
```

## Evaluation signals

- NanoSimFormer DNA median read accuracies computed from minimap2 alignments fall within 98.74-99.65%, close to real experimental baselines of 99.39-99.72%
- Median Q-scores fall within 23.3-24.7 for NanoSimFormer DNA simulations
- Per-read error rates match mismatch 0.57% / insertion 0.28% / deletion 0.42%
- HG001-HG005 mean DTW distances to real signal chunks fall within 3.98-6.75 for the same datasets
- Competing simulators show markedly lower median accuracies (seq2squiggle 93.34-97.25%, Squigulator 89.98-96.70%), confirming the alignment pipeline discriminates between simulators

## Limitations

- Accuracy estimates depend on the basecaller and version used (Dorado v5.0.0 in the benchmark); changing basecallers changes the medians
- RNA004 simulation requires a 3-column transcript profile TSV (--trans-profile) rather than a plain FASTA, and uses a different preset (ont_rna004_4khz)
- Reported thresholds are specific to R10.4.1 chemistry presets; NanoSimFormer notes a step-by-step procedure is needed for adapting the model to future ONT sequencing updates

## Evidence

- [other] Align all basecalled reads to their references with minimap2 to obtain per-read alignments.: [evidence span withheld — anchor status: not_found]
- [other] NanoSimFormer achieved median read accuracies of 98.74-99.65%, closely matching real experimental data (99.39-99.72%).: [evidence span withheld — anchor status: not_found]
- [methods] The `--basecall` option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [readme] Dorado is a high-performance, easy-to-use, open source analysis engine for Oxford Nanopore reads.: [evidence span withheld — anchor status: external_doc]

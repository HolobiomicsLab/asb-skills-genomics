---
name: per-read-error-profile-analysis
description: 'Use when benchmarking nanopore signal simulators or basecallers on DNA/RNA
  reads: you have POD5/FASTQ read sets plus a reference and need per-read accuracy,
  Q-score, error-type breakdowns, and DTW signal distances aggregated to medians.'
when_to_use_negative:
- Input is raw signal without a reference or basecalled reads — alignment-based per-read
  accuracy cannot be computed without minimap2 alignments against a reference.
- Comparing assembly consensus quality rather than per-read basecalling fidelity —
  use variant-calling F1 or consensus error-rate benchmarks instead.
- Evaluating methylation or modified-base calling performance — Dorado/Remora mod-calling
  workflows, not identity-based error profiling, are required.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Simulates ONT DNA/RNA POD5 signals whose basecalling error profiles are measured;
    also provides read-filter thresholds (min_identity, min_quality, min_align_ratio)
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated POD5 signals into FASTQ (--basecall option) so per-read
    accuracy can be computed
  repo: github.com/nanoporetech/dorado
- name: minimap2
  role: Aligns basecalled reads to references to obtain per-read alignments for accuracy
    and error-type calculation
- name: seq2squiggle
  role: Competing simulator whose output POD5 signals are basecalled and error-profiled
    for benchmark comparison
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Squigulator
  role: Competing simulator whose output POD5 signals are basecalled and error-profiled
    for benchmark comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:5aeae7e1243012c91e47a36edcd46c9ab516ec47f27a6821c8bce4462d85b29e
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# per-read-error-profile-analysis

## Summary

Quantify per-read basecalling accuracy and error structure (mismatch, insertion, deletion rates, Q-scores, and DTW signal distances) of simulated or real nanopore reads against their reference, as used to benchmark NanoSimFormer against seq2squiggle and Squigulator on ONT R10.4.1 DNA and RNA004 data.

## When to use

Use when benchmarking nanopore signal simulators or basecallers on DNA/RNA reads: you have POD5/FASTQ read sets plus a reference and need per-read accuracy, Q-score, error-type breakdowns, and DTW signal distances aggregated to medians.

## When NOT to use

- Input is raw signal without a reference or basecalled reads — alignment-based per-read accuracy cannot be computed without minimap2 alignments against a reference.
- Comparing assembly consensus quality rather than per-read basecalling fidelity — use variant-calling F1 or consensus error-rate benchmarks instead.
- Evaluating methylation or modified-base calling performance — Dorado/Remora mod-calling workflows, not identity-based error profiling, are required.

## Inputs

- POD5 nanopore signal files (simulated from references via --preset ont_r1041_dna_5khz or ont_rna004_4khz, or real experimental reads)
- Basecalled FASTQ reads (from Dorado --basecall)
- Reference genome FASTA (e.g. HG002 chr22, bacterial genomes) or transcriptome reference with 3-column abundance/truncation TSV profile
- Alignment BAM/SAM of reads against reference (minimap2 output)

## Outputs

- Per-read accuracy values and median Q-scores per simulator/dataset
- Per-read mismatch, insertion, and deletion error rates
- Dynamic Time Warping (DTW) distances between simulated and real signal chunks
- Aggregated benchmark table (e.g. simulation_benchmark_results.csv) with median metrics per simulator and dataset

## How to apply

Basecall simulated or raw POD5 signals with Dorado (e.g. v5.0.0) into FASTQ, then align all basecalled reads to their reference genome/transcriptome with minimap2 to obtain per-read alignments. From each alignment compute per-read accuracy, median Q-score, and mismatch/insertion/deletion error rates; optionally compute Dynamic Time Warping (DTW) distances between simulated and real signal chunks to assess signal fidelity. Optionally restrict reads to high-confidence alignments using the article's filter thresholds (min_identity 0.99, min_quality 20.0, min_align_ratio 0.995, min_aligned_read_len 1000). Aggregate metrics per simulator and dataset into a consolidated table (e.g. simulation_benchmark_results.csv) and compare medians against real experimental baselines, which in the article fall at 99.39-99.72% accuracy for R10.4.1 data.

## Related tools

- **NanoSimFormer** (Simulates ONT DNA/RNA POD5 signals whose basecalling error profiles are measured; also provides read-filter thresholds (min_identity, min_quality, min_align_ratio)) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated POD5 signals into FASTQ (--basecall option) so per-read accuracy can be computed) — github.com/nanoporetech/dorado
- **minimap2** (Aligns basecalled reads to references to obtain per-read alignments for accuracy and error-type calculation)
- **seq2squiggle** (Competing simulator whose output POD5 signals are basecalled and error-profiled for benchmark comparison) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Squigulator** (Competing simulator whose output POD5 signals are basecalled and error-profiled for benchmark comparison)

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out/ --mode Reference --sample-reads 1000 --gpu 0 --preset ont_r1041_dna_5khz --basecall
```

## Evaluation signals

- NanoSimFormer DNA median read accuracies fall within 98.74-99.65%, matching real experimental data at 99.39-99.72%.
- Median Q-scores for NanoSimFormer DNA reads fall within 23.3-24.7 (abstract reports >22.8 for DNA R10.4.1 and direct RNA).
- Per-read error rates match mismatch 0.57%, insertion 0.28%, deletion 0.42%.
- HG001-HG005 mean DTW distances between simulated and real signal chunks fall within 3.98-6.75.
- Filtered read sets respect documented cutoffs: min_identity 0.99, min_quality 20.0, min_align_ratio 0.995, min_aligned_read_len 1000, min_mapq 1, dwell 4-256 samples.

## Limitations

- Model checkpoints must be downloaded separately (Zenodo 10.5281/zenodo.20364515) unless the Docker image chobits323/nano-sim:v1.1 is used; pip/conda installs are incomplete without them.
- GPU requirements: NVIDIA GPU with CUDA compute capability >= 8.x and driver >= 525; only Linux is fully tested (Ubuntu 22.04).
- Benchmark validity depends on using identical Dorado basecalling and minimap2 alignment across all simulators; differing basecaller versions confound comparisons.
- No changelog available for the source repository.

## Evidence

- [results] NanoSimFormer achieved median read accuracies of 98.74-99.65%, closely matching real experimental data (99.39-99.72%): [evidence span withheld — anchor status: not_found]
- [methods] The `--basecall` option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [methods] --min_identity MIN_IDENTITY Minimum alignment identity to include a read. (default: 0.99): "--min_identity MIN_IDENTITY Minimum alignment identity to include a read. (default: 0.99)"
- [abstract] NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies' latest DNA R10.4.1 and direct RNA sequencing: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies' latest DNA R10.4.1 and direct RNA sequencing"
- [other] Basecall the seq2squiggle and Squigulator signals with Dorado v5.0.0 into FASTQ reads.: [evidence span withheld — anchor status: not_found]

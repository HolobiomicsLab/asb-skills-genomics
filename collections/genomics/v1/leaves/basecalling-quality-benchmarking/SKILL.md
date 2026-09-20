---
name: basecalling-quality-benchmarking
description: Use when evaluating how faithfully a nanopore signal simulator (e.g.,
  NanoSimFormer, seq2squiggle, Squigulator) reproduces real R10.4.1 DNA or RNA004
  direct-RNA basecalling accuracy and signal fidelity.
when_to_use_negative:
- The goal is variant-calling fidelity assessment (SNP/indel F1-scores) or de novo
  assembly consensus error rather than read-level basecalling accuracy and signal
  fidelity
- Input signals are already basecalled reads — only basecall-align-summarize; no simulation
  comparison is needed
- Sequencing chemistry is neither ONT DNA R10.4.1 nor direct-RNA RNA004, since the
  available presets (ont_r1041_dna_5khz, ont_rna004_4khz) do not cover other chemistries
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSimFormer
  role: Generates simulated POD5 signals from references and basecalled FASTQ via
    --basecall, the simulator under benchmark
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated signals into FASTQ reads for accuracy computation
  repo: https://github.com/nanoporetech/dorado
- name: seq2squiggle
  role: Competing simulator whose POD5 outputs are basecalled and benchmarked for
    comparison
  repo: https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Squigulator
  role: Competing simulator whose POD5 outputs are basecalled and benchmarked for
    comparison
- name: Docker
  role: Recommended installation/execution environment for NanoSimFormer simulation
    runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:c768f9cdd692bc35376bb774ad38e8188b44274cc353498f7d2ddf40ad2c438f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Squigulator
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# basecalling-quality-benchmarking

## Summary

Benchmark basecalling quality of simulated nanopore signals by basecalling simulated POD5 files with Dorado, aligning reads with minimap2, and comparing median per-read accuracies, Q-scores, error profiles, and DTW signal distances against real experimental baselines and competing simulators.

## When to use

Use when evaluating how faithfully a nanopore signal simulator (e.g., NanoSimFormer, seq2squiggle, Squigulator) reproduces real R10.4.1 DNA or RNA004 direct-RNA basecalling accuracy and signal fidelity.

## When NOT to use

- The goal is variant-calling fidelity assessment (SNP/indel F1-scores) or de novo assembly consensus error rather than read-level basecalling accuracy and signal fidelity
- Input signals are already basecalled reads — only basecall-align-summarize; no simulation comparison is needed
- Sequencing chemistry is neither ONT DNA R10.4.1 nor direct-RNA RNA004, since the available presets (ont_r1041_dna_5khz, ont_rna004_4khz) do not cover other chemistries

## Inputs

- Reference genome FASTA files (e.g., HG002 chr22, HG001/HG003–HG005, five bacterial genomes, MSA-1010)
- Transcript profile TSV (3-column) for RNA004 transcript simulation
- Simulated POD5 signal files from NanoSimFormer, seq2squiggle, and Squigulator
- Real experimental POD5/FASTQ baselines (HG002 R10.4.1 sample statistics)
- Dorado v5.0.0 basecaller and minimap2 aligner

## Outputs

- Basecalled FASTQ (or BAM via --emit-bam) reads per simulator/dataset
- Per-read accuracies and median Q-scores
- Per-read mismatch/insertion/deletion error rates
- DTW distances between simulated and real signal chunks
- Consolidated benchmark table (simulation_benchmark_results.csv)

## How to apply

Simulate signals for each reference dataset (e.g., HG002 chr22, HG001–HG005, bacterial genomes, MSA-1010) using NanoSimFormer with --preset ont_r1041_dna_5khz (or ont_rna004_4khz with a 3-column transcript profile TSV for RNA004) and --basecall so Dorado converts simulated POD5 into FASTQ. Basecall the competing simulators' POD5 outputs (seq2squiggle, Squigulator) with Dorado as well, then align all basecalled reads to their references with minimap2 to obtain per-read alignments. Compute per-read accuracies, median Q-scores, mismatch/insertion/deletion error rates, and Dynamic Time Warping (DTW) distances between simulated and real signal chunks, then aggregate median metrics per simulator and dataset. Judge success by comparing against expected ranges: NanoSimFormer DNA median accuracies of 98.74–99.65% (real data: 99.39–99.72%), median Q 23.3–24.7, error rates mismatch 0.57% / insertion 0.28% / deletion 0.42%, and HG001–HG005 mean DTW 3.98–6.75, versus markedly lower competitor accuracies (seq2squiggle 93.34–97.25%, Squigulator 89.98–96.70%).

## Related tools

- **NanoSimFormer** (Generates simulated POD5 signals from references and basecalled FASTQ via --basecall, the simulator under benchmark) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated signals into FASTQ reads for accuracy computation) — https://github.com/nanoporetech/dorado
- **seq2squiggle** (Competing simulator whose POD5 outputs are basecalled and benchmarked for comparison) — https://github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Squigulator** (Competing simulator whose POD5 outputs are basecalled and benchmarked for comparison)
- **Docker** (Recommended installation/execution environment for NanoSimFormer simulation runs)

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --sample-reads 1000 --preset ont_r1041_dna_5khz --basecall
```

## Evaluation signals

- NanoSimFormer DNA median read accuracies fall within 98.74–99.65%, close to real experimental 99.39–99.72%
- Median Q-scores fall within 23.3–24.7 (exceeding the paper's stated >22.8 threshold)
- Per-read error rates match mismatch 0.57% / insertion 0.28% / deletion 0.42%
- HG001–HG005 mean DTW distances between simulated and real signal chunks fall within 3.98–6.75
- Competitor simulators should show lower medians: seq2squiggle 93.34–97.25%, Squigulator 89.98–96.70%

## Limitations

- Requires an NVIDIA GPU with CUDA compute capability >= 8.x and driver >= 525; fully tested only on Linux (Ubuntu 22.04)
- For pip/conda installs, model checkpoints must be manually downloaded from Zenodo and unzipped into the package path (Docker image bundles them)
- Benchmark coverage is limited to ONT R10.4.1 DNA and RNA004 direct-RNA presets; adapting to new chemistries requires retraining via the train/create_dataset subcommands
- Result variability across simulators can be substantial; competitor medians span ~7 percentage points depending on dataset

## Evidence

- [results] NanoSimFormer achieved median read accuracies of 98.74-99.65%, closely matching real experimental data (99.39-99.72%): [evidence span withheld — anchor status: not_found]
- [methods] Basecalling of simulated reads with Dorado: [evidence span withheld — anchor status: not_found]
- [abstract] Median basecalling accuracy exceeding 99%: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies’ latest DNA R10.4.1 and direct RNA sequencing"
- [readme] Preset selection for DNA simulation: "--preset {ont_r1041_dna_5khz,ont_rna004_4khz}
                        ont platform preset"

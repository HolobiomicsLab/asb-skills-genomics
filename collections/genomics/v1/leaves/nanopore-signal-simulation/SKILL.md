---
name: nanopore-signal-simulation
description: Use when synthetic ONT R10.4.1 DNA or RNA004 direct-RNA read signals
  (POD5) are needed to benchmark basecalling accuracy, variant calling, assembly,
  or simulator fidelity without real sequencing runs.
when_to_use_negative:
- When simulating sequencing chemistries other than ONT DNA R10.4.1 (5 kHz) or direct-RNA
  RNA004 (4 kHz), unless adapting the model to new chemistry via the paper's extensibility
  guide
- When the goal is empirical analysis of real experimental reads rather than synthetic
  signal generation
- When training a custom model is the goal — use the separate create_dataset/train
  subcommands workflow instead of simulate
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: NanoSimFormer
  role: End-to-end Transformer-based simulator generating synthetic POD5 signals from
    references or reads; presets ont_r1041_dna_5khz and ont_rna004_4khz; --basecall
    triggers Dorado basecalling
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls simulated POD5 signals into FASTQ/BAM so accuracy and variant-calling
    fidelity can be evaluated
  repo: github.com/nanoporetech/dorado
- name: seq2squiggle
  role: Competing deep-learning simulator used as a baseline for accuracy comparison
    (93.34-97.25% median accuracy)
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: Squigulator
  role: Signal-based competing simulator baseline (89.98-96.70% median accuracy)
- name: minimap2
  role: Aligns basecalled simulated reads to references to compute per-read accuracy,
    Q-scores, and error profiles
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  - task_002
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:a6acd8566f8ea0c4ad2b702fb8cafa41544f02cd9593d896c8f1cc422de46dd6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - DeepSimulator
  - Docker
  - Dorado
  - Git
  - NanoSimFormer
  - Python
  - RTG tools
  - Squigulator
  - conda
  - flash-attn
  - linmdtw
  - mamba
  - minimap2
  - mrmsdtw
  - nox
  - pip
  - poetry
  - pre-commit
  - seq2squiggle
  - squigulator
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# nanopore-signal-simulation

## Summary

Generate high-fidelity synthetic Oxford Nanopore sequencing signals (POD5) from reference genomes/transcriptomes or basecalled reads using NanoSimFormer's Transformer model with basecaller guidance, optionally basecalling simulated signals to FASTQ with Dorado for downstream benchmarking.

## When to use

Use when synthetic ONT R10.4.1 DNA or RNA004 direct-RNA read signals (POD5) are needed to benchmark basecalling accuracy, variant calling, assembly, or simulator fidelity without real sequencing runs.

## When NOT to use

- When simulating sequencing chemistries other than ONT DNA R10.4.1 (5 kHz) or direct-RNA RNA004 (4 kHz), unless adapting the model to new chemistry via the paper's extensibility guide
- When the goal is empirical analysis of real experimental reads rather than synthetic signal generation
- When training a custom model is the goal — use the separate create_dataset/train subcommands workflow instead of simulate

## Inputs

- Reference genome FASTA (DNA R10.4.1 simulation, e.g., chr22.fasta)
- Transcriptome FASTA with 3-column TSV transcript abundance/truncation profile (--trans-profile, RNA004 preset)
- Basecalled reads FASTQ (Read mode simulation)
- Pre-trained NanoSimFormer model checkpoints (Zenodo models.zip or bundled in Docker image)
- NVIDIA GPU with CUDA compute capability >= 8.x and driver >= 525

## Outputs

- Simulated nanopore signal POD5 file
- Basecalled FASTQ (or BAM with --emit-bam) of simulated reads when --basecall is enabled
- Sampled reference reads FASTA (--sample-output)
- Per-read accuracy/Q-score metrics when used for benchmarking

## How to apply

Run NanoSimFormer via the pre-built Docker image chobits323/nano-sim:v1.1 (or conda/pip with Python 3.10 and flash-attn, plus Zenodo checkpoints 10.5281/zenodo.20364515). Use `python -m nano_signal_simulator simulate` with `--mode Reference` and the `ont_r1041_dna_5khz` preset for DNA FASTA references (e.g., HG002 chr22, HG001-HG005, bacterial genomes) or `ont_rna004_4khz` with a 3-column transcript-profile TSV for direct-RNA transcriptomes; supply `--coverage` or `--sample-reads` to control output volume and `--seed 42` for reproducibility. Add `--basecall` so Dorado converts simulated POD5 signals into a FASTQ (optionally BAM with `--emit-bam`). Tune noise and read-length behaviour via `--noise-stdv`, `--duration-stdv` (DNA range ~2.45-8.49; RNA004 ~3.87-13.42), and `--length-dist-mode {stat,expon}`. Validate fidelity by Dorado-basecalling the simulated reads, aligning with minimap2, and checking median read accuracy (~98.74-99.65% for DNA, matching real 99.39-99.72%) and median Q-scores (23.3-24.7); keep sigma_amp <= 2.0 to preserve SNP F1 ~0.997. Note that fidelity depends critically on the basecaller-guided CTC training objective — an ablated model without it drops to 93.18-97.69% median accuracy.

## Related tools

- **NanoSimFormer** (End-to-end Transformer-based simulator generating synthetic POD5 signals from references or reads; presets ont_r1041_dna_5khz and ont_rna004_4khz; --basecall triggers Dorado basecalling) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls simulated POD5 signals into FASTQ/BAM so accuracy and variant-calling fidelity can be evaluated) — github.com/nanoporetech/dorado
- **seq2squiggle** (Competing deep-learning simulator used as a baseline for accuracy comparison (93.34-97.25% median accuracy)) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **Squigulator** (Signal-based competing simulator baseline (89.98-96.70% median accuracy))
- **minimap2** (Aligns basecalled simulated reads to references to compute per-read accuracy, Q-scores, and error profiles)

## Examples

```
docker run --rm -it --gpus=all -v ${EXAMPLE_DIR}:${EXAMPLE_DIR} --ipc=host chobits323/nano-sim:v1.1 python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --sample-reads 1000 --preset ont_r1041_dna_5khz --basecall --seed 42
```

## Evaluation signals

- Median basecalled read accuracy for DNA R10.4.1 simulations falls within 98.74-99.65%, close to real experimental 99.39-99.72%
- Median Q-scores fall within 23.3-24.7 (abstract reports >22.8); per-read error rates match mismatch 0.57%, insertion 0.28%, deletion 0.42%
- DTW distances between simulated and real signal chunks (mean 3.98-6.75 on HG001-HG005) remain in range
- Variant calling on simulated reads yields SNP F1 0.9953-0.9973 and indel F1 0.7862-0.8612; SNP F1 degrades to ~0.7031 only when --noise-stdv is raised to 4.0
- POD5 and basecalled FASTQ are produced for every reference and the simulation is reproducible with --seed 42

## Limitations

- Requires an NVIDIA GPU with CUDA compute capability >= 8.x (Ampere or newer) and driver >= 525; Linux/Ubuntu 22.04 tested
- Non-default noise settings degrade fidelity: SNP F1 drops from ~0.997 to 0.7031 at sigma_amp=4.0
- Pre-trained fidelity is tied to the basecaller-guided CTC training objective; removing it degrades median accuracy to 93.18-97.69%
- Pip/conda installs must separately download model checkpoints from Zenodo (10.5281/zenodo.20364515); the Docker image bundles them
- Only two chemistry presets are provided (DNA R10.4.1 5 kHz, RNA004 4 kHz); other chemistries require custom adaptation

## Evidence

- [abstract] Basecaller guidance during training: "an end-to-end Transformer-based signal simulator that integrates basecaller guidance during training to generate high-fidelity nanopore signals"
- [methods] Basecall option: [evidence span withheld — anchor status: not_found]
- [intro] Presets and input modes: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads."
- [readme] Docker image: "docker pull chobits323/nano-sim:v1.1"
- [readme] Install and checkpoints: "The Docker image already contains all required pre-trained model files. For pip/conda installations, download the model checkpoints from [Zenodo](https://doi.org/10.5281/zenodo.20364515)"
- [other] Noise sensitivity: [evidence span withheld — anchor status: not_found]

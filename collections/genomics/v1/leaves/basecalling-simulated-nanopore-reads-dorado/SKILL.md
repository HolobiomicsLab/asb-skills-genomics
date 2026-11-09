---
name: basecalling-simulated-nanopore-reads-dorado
description: Use when simulated POD5 signals from NanoSimFormer (R10.4.1 DNA or RNA004
  direct-RNA presets) must be converted to basecalled reads for alignment and variant-calling
  benchmarking.
when_to_use_negative:
- Input is real experimental POD5/FAST5 data already basecalled by Dorado — re-simulation
  and re-basecalling are unnecessary
- Benchmarking the simulator itself on raw-signal fidelity metrics (e.g. dwell time
  or level distributions), which requires signals only, not basecalls
- Platforms other than ONT R10.4.1 DNA or RNA004 direct-RNA — NanoSimFormer presets
  do not cover other chemistries without model retraining/adaptation
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Simulator whose --basecall option automatically invokes Dorado on simulated
    signals; invoked via 'python -m nano_signal_simulator simulate'
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecaller executed on simulated POD5 signals to produce FASTQ/BAM basecalls
  repo: https://github.com/nanoporetech/dorado
- name: Bonito
  role: Used internally by NanoSimFormer for loading official ONT basecalling models
  repo: https://github.com/nanoporetech/bonito
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:123e679dfa61ede6694b5df317ee4043af48474a17b3a64171f2ea5c615a8f94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Bonito
  - Dorado
  - NanoSimFormer
  - RTG tools
  - Squigulator
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# basecalling-simulated-nanopore-reads-dorado

## Summary

Basecall NanoSimFormer-simulated nanopore signals into FASTQ/BAM using Dorado, triggered via the simulator's --basecall option, so that simulated reads can be aligned and downstream variant-calling fidelity (SNP/indel F1 vs. GIAB truth sets) can be benchmarked against real experimental baselines.

## When to use

Use when simulated POD5 signals from NanoSimFormer (R10.4.1 DNA or RNA004 direct-RNA presets) must be converted to basecalled reads for alignment and variant-calling benchmarking.

## When NOT to use

- Input is real experimental POD5/FAST5 data already basecalled by Dorado — re-simulation and re-basecalling are unnecessary
- Benchmarking the simulator itself on raw-signal fidelity metrics (e.g. dwell time or level distributions), which requires signals only, not basecalls
- Platforms other than ONT R10.4.1 DNA or RNA004 direct-RNA — NanoSimFormer presets do not cover other chemistries without model retraining/adaptation

## Inputs

- NanoSimFormer-simulated POD5 signal files (ont_r1041_dna_5khz or ont_rna004_4khz preset)
- Reference genome FASTA or basecalled-read FASTQ used as simulation input
- 3-column TSV transcript profile (--trans-profile) for RNA004 transcript abundance and truncation

## Outputs

- Basecalled FASTQ file of simulated reads
- Basecalled BAM file (when --emit-bam is set)
- Simulated POD5 signals alongside the basecalls

## How to apply

Enable basecalling directly during simulation by passing --basecall (and --emit-bam to store basecalled reads as BAM) to 'python -m nano_signal_simulator simulate', which automatically runs Dorado on the simulated signals after generation. Use the ont_r1041_dna_5khz preset for HG001–HG005 DNA simulations and ont_rna004_4khz with --trans-profile for HG002 direct-RNA simulations, keeping --seed fixed (e.g. 42) for reproducibility. The resulting FASTQ/BAM are then aligned to the reference and fed to Clair3 (DNA), Clair3-RNA (RNA004), or Sniffles2 (SVs) for benchmarking against GIAB v3.3.2 truth sets with RTG vcfeval and Truvari. Correct application is judged by whether basecalled reads reach the reported fidelity: median accuracy >99% and Q-scores above 22.8, with downstream DNA SNP F1 0.9953–0.9973 and small indel F1 0.7862–0.8612.

## Related tools

- **NanoSimFormer** (Simulator whose --basecall option automatically invokes Dorado on simulated signals; invoked via 'python -m nano_signal_simulator simulate') — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecaller executed on simulated POD5 signals to produce FASTQ/BAM basecalls) — https://github.com/nanoporetech/dorado
- **Bonito** (Used internally by NanoSimFormer for loading official ONT basecalling models) — https://github.com/nanoporetech/bonito

## Examples

```
docker run --rm -it --gpus=all -v ${EXAMPLE_DIR}:${EXAMPLE_DIR} --ipc=host chobits323/nano-sim:v1.1 python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --basecall --emit-bam --preset ont_r1041_dna_5khz --seed 42
```

## Evaluation signals

- Median basecalling accuracy of simulated reads exceeds 99% with Q-scores above 22.8 for both DNA R10.4.1 and direct RNA
- Downstream Clair3 SNP F1-scores on simulated DNA BAMs fall within 0.9953–0.9973 across the five HG001–HG005 samples
- Small indel F1-scores fall within 0.7862–0.8612, comparable to experimental baselines of 0.8209–0.8963
- Basecalled reads are emitted as FASTQ, or as BAM when --emit-bam is passed, and align cleanly to the reference for Clair3/Clair3-RNA/Sniffles2 input
- RNA004 sample benchmarks yield SNP F1 ≈ 0.92 and indel F1 ≈ 0.57

## Limitations

- Requires an NVIDIA GPU with CUDA compute capability >= 8.x (Ampere or newer) and driver >= 525; tested on Ubuntu 22.04
- Pre-trained presets cover only ONT R10.4.1 DNA and RNA004 direct-RNA; other chemistries require the documented model adaptation workflow
- Pip/conda installations must additionally download model checkpoints from Zenodo (10.5281/zenodo.20364515); the Docker image includes them

## Evidence

- [methods] The `--basecall` option automatically runs Dorado to basecall the simulated reads into a FASTQ file after signal simulation.: [evidence span withheld — anchor status: not_found]
- [abstract] NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies’ latest DNA R10.4.1 and direct RNA sequencing, closely mirroring real: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8"
- [abstract] It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612 for single-nucleotide polymorphisms and: "achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612 for single-nucleotide polymorphisms"
- [readme] --basecall            enable basecalling simulated reads (default: False): "--basecall            enable basecalling simulated reads (default: False)"

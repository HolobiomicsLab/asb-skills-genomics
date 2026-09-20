---
name: variant-calling-f1-evaluation
description: Use when assessing how faithfully a simulated ONT read set recapitulates
  real variant-calling performance, or when quantifying sensitivity of variant detection
  to simulator noise parameters (e.g. sigma_amp, duration SD) or genomic context (homopolymer/STR).
when_to_use_negative:
- Input reads are already real experimental reads and the goal is raw variant calling
  rather than simulator fidelity assessment
- The analysis targets methylation/modified-base status rather than small-variant
  SNP/indel detection performance
- Downstream analysis is de novo bacterial assembly or abundance correlation rather
  than variant-level benchmarking
edam_operation: http://edamontology.org/operation_0329
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0199
tools:
- name: NanoSimFormer
  role: Generates the simulated ONT DNA R10.4.1 / RNA004 read sets whose variant-calling
    F1 is being evaluated, via its simulate subcommand
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated POD5 signals into FASTQ/BAM when the --basecall option
    is enabled
  repo: github.com/nanoporetech/dorado
- name: Docker
  role: Runs NanoSimFormer via the pre-built image chobits323/nano-sim:v1.1 bundling
    checkpoints and Dorado
- name: Python
  role: Bins false-positive SNP/indel calls and deletion errors by homopolymer/STR
    context length and writes the sensitivity tables
- name: seq2squiggle
  role: Comparator simulator whose context-stratified false positives are contrasted
    with NanoSimFormer's
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:b4fc1636742899b183abd80e975ce4da5e0f550aacefbe307ad43a332d3f7586
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Python
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_0329
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# variant-calling-f1-evaluation

## Summary

Evaluate small-variant calling fidelity of simulated nanopore reads by aligning Dorado-basecalled FASTQ/BAM to a reference, calling variants, and computing SNP/indel F1-scores — optionally swept across noise parameters or stratified by homopolymer/STR context.

## When to use

Use when assessing how faithfully a simulated ONT read set recapitulates real variant-calling performance, or when quantifying sensitivity of variant detection to simulator noise parameters (e.g. sigma_amp, duration SD) or genomic context (homopolymer/STR).

## When NOT to use

- Input reads are already real experimental reads and the goal is raw variant calling rather than simulator fidelity assessment
- The analysis targets methylation/modified-base status rather than small-variant SNP/indel detection performance
- Downstream analysis is de novo bacterial assembly or abundance correlation rather than variant-level benchmarking

## Inputs

- Reference genome FASTA (e.g. HG002 reference) for simulation and variant calling truth
- Simulated POD5 + FASTQ/BAM read sets (one per noise-parameter setting)
- Real HG002 R10.4.1 read variant calls as baseline comparator
- Competitor simulator variant sets (seq2squiggle, Squigulator)
- Homopolymer-run and short tandem repeat genomic context tracks derived from the reference

## Outputs

- Per-condition SNP and indel F1-score table (e.g. variant_f1_noise_sensitivity.csv)
- Context-stratified false-positive SNP/indel and deletion-error counts binned by homopolymer/STR length
- Comparative plots of error rates per simulator vs. real-read baseline

## How to apply

Simulate reads from the reference FASTA with NanoSimFormer (--mode Reference, --preset ont_r1041_dna_5khz, --basecall, --seed 42, --batch-size 64), optionally sweeping --noise-stdv (up to 4.0) or --duration-stdv (DNA ~2.45–8.49; RNA004 ~3.87–13.42) and generating one POD5+FASTQ set per setting. The --basecall option runs Dorado to produce FASTQ (or BAM with --emit-bam). Align basecalled reads with minimap2 using the --MD tag on an indexed BAM, applying NanoSimFormer data-preparation filters (min_mapq >= 1, min_quality 20.0), then call small variants against the reference with a standard pipeline. Compute SNP and indel F1-scores per condition; for context stratification, annotate each variant position against homopolymer-run and STR tracks derived from the reference and bin false positives and deletion errors by repeat length. Compare against real-read baselines and against seq2squiggle/Squigulator outputs to detect excess homopolymer/STR false positives. Tabulate results (e.g. variant_f1_noise_sensitivity.csv) and check against expected degradation/invariance patterns.

## Related tools

- **NanoSimFormer** (Generates the simulated ONT DNA R10.4.1 / RNA004 read sets whose variant-calling F1 is being evaluated, via its simulate subcommand) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated POD5 signals into FASTQ/BAM when the --basecall option is enabled) — github.com/nanoporetech/dorado
- **Docker** (Runs NanoSimFormer via the pre-built image chobits323/nano-sim:v1.1 bundling checkpoints and Dorado)
- **Python** (Bins false-positive SNP/indel calls and deletion errors by homopolymer/STR context length and writes the sensitivity tables)
- **seq2squiggle** (Comparator simulator whose context-stratified false positives are contrasted with NanoSimFormer's) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
docker run --rm -it --gpus=all chobits323/nano-sim:v1.1 python -m nano_signal_simulator simulate --mode Reference --basecall --seed 42 --batch-size 64 --preset ont_r1041_dna_5khz --noise-stdv 2.0 --input ref.fasta --output out
```

## Evaluation signals

- Sensitivity table must show SNP F1 >= 0.99 for sigma_amp <= 2.0, ~0.992 at sigma_amp 3.0, and ~0.7031 at sigma_amp 4.0
- Duration-stdv sweep must stay flat: SNP F1 within 0.9972–0.9975 and indel F1 within 0.8084–0.8244 (variation < ~0.002/~0.016)
- Baseline HG002 run should reproduce SNP F1 in the 0.9953–0.9973 range and small indel F1 in the 0.7862–0.8612 range
- Homopolymer/STR stratification must show NanoSimFormer FP rates comparable to real data while seq2squiggle/Squigulator deletion errors increase with homopolymer length
- Alignment setup must use minimap2 --MD with an indexed BAM and filters min_mapq >= 1, min_quality 20.0 before calling

## Limitations

- SNP F1 degrades sharply at sigma_amp=4.0 (to ~0.7031), so amplifying noise beyond ~3.0 invalidates comparisons against real-data baselines
- Context stratification depends on accurately derived homopolymer/STR tracks from the reference; mis-annotation shifts bins
- Pre-trained models are chemistry-specific (ont_r1041_dna_5khz, ont_rna004_4khz); other chemistries require the Zenodo checkpoints or retraining

## Evidence

- [abstract] Faithful recapitulation of SNP/indel F1 ranges across five human samples: "It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612"
- [abstract] Homopolymer/STR false-positive reduction claim motivating context stratification: "Compared with previous simulators, NanoSimFormer also substantially reduces false positives in homopolymer and short tandem repeat regions."
- [methods] Alignment filter thresholds used before variant calling: "--min_mapq MIN_MAPQ Minimum mapping quality (MAPQ). (default: 1)"
- [readme] Dorado basecalling step producing FASTQ from simulated signals: [evidence span withheld — anchor status: not_found]

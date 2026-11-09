---
name: duration-variance-sensitivity-sweep
description: Use when you need to quantify how robust downstream small-variant calling
  is to perturbing only the duration-sampler variance of simulated ONT R10.4.1 DNA
  or RNA004 reads, while holding amplitude noise and seed fixed.
when_to_use_negative:
- The question concerns amplitude-noise sensitivity (--noise-stdv / sigma_amp), where
  SNP F1 degrades sharply to ~0.7031 at sigma_amp=4.0 — use the amplitude-noise sweep
  instead of the duration sweep.
- You are simulating with a tool lacking a duration-sampler standard-deviation parameter
  (e.g. seq2squiggle) or analyzing real (non-simulated) reads — the sweep design requires
  the --duration-stdv control.
- You need general read-length or coverage tuning (--mean-read-length, --coverage,
  --sample-reads) rather than a duration-variance sensitivity analysis.
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: NanoSimFormer
  role: Signal simulator whose `simulate` subcommand accepts --duration-stdv; used
    to generate one POD5+FASTQ set per duration-variance value from the reference
    FASTA
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls each simulated read set into FASTQ/BAM via the --basecall option
    so variant calling can be run per sweep point
  repo: github.com/nanoporetech/dorado
- name: Docker (chobits323/nano-sim:v1.1)
  role: Recommended installation path that bundles pretrained checkpoints and Dorado
    for reproducible GPU simulation runs
  repo: https://github.com/BioinfoSZU/NanoSimFormer
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:cdb108a8d57285dbbda0a04fe2be47fd8b61440c67cd3f14c088006f0e096c1b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Docker (chobits323/nano-sim:v1.1)
  - Dorado
  - NanoSimFormer
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# duration-variance-sensitivity-sweep

## Summary

Sweeps NanoSimFormer's duration-sampler standard-deviation parameter (--duration-stdv) across its supported range for the ont_r1041_dna_5khz (~2.45-8.49) or ont_rna004_4khz (~3.87-13.42) presets, basecalls each simulated POD5+FASTQ set with Dorado, and measures how duration-variance perturbation affects SNP and small-indel F1 scores.

## When to use

Use when you need to quantify how robust downstream small-variant calling is to perturbing only the duration-sampler variance of simulated ONT R10.4.1 DNA or RNA004 reads, while holding amplitude noise and seed fixed.

## When NOT to use

- The question concerns amplitude-noise sensitivity (--noise-stdv / sigma_amp), where SNP F1 degrades sharply to ~0.7031 at sigma_amp=4.0 — use the amplitude-noise sweep instead of the duration sweep.
- You are simulating with a tool lacking a duration-sampler standard-deviation parameter (e.g. seq2squiggle) or analyzing real (non-simulated) reads — the sweep design requires the --duration-stdv control.
- You need general read-length or coverage tuning (--mean-read-length, --coverage, --sample-reads) rather than a duration-variance sensitivity analysis.

## Inputs

- Reference genome FASTA (e.g. chr22.fasta) for ont_r1041_dna_5khz or transcriptome for ont_rana004_4khz simulation
- NanoSimFormer Docker image chobits323/nano-sim:v1.1 with bundled pretrained checkpoints and Dorado
- Duration-sampler standard-deviation sweep values (DNA ~2.45-8.49; RNA004 preset ~3.87-13.42)

## Outputs

- One POD5 + FASTQ (optionally BAM with --emit-bam) simulated read set per --duration-stdv value
- Basecalled FASTQ/BAM per setting from Dorado
- Per-setting SNP and indel F1 scores tabulated in a sensitivity CSV (e.g. variant_f1_noise_sensitivity.csv)

## How to apply

Pull the pre-built Docker image chobits323/nano-sim:v1.1 (bundles pretrained checkpoints and Dorado), then run NanoSimFormer's `simulate` subcommand in Reference mode with the ont_r1041_dna_5khz preset (--basecall, --seed 42, --batch-size 64), sweeping --duration-stdv over the DNA range ~2.45-8.49 (or ~3.87-13.42 for the RNA004 preset) in fixed increments to generate one POD5+FASTQ set per value. Critically, keep --noise-stdv at its default (untouched) so that amplitude noise is not confounded with duration variance; the fixed --seed 42 makes replicate sets comparable across the sweep. Call small variants from each basecalled FASTQ/BAM against the reference genome FASTA with the standard pipeline, compute SNP and indel F1 per duration-stdv setting, and tabulate results (e.g. variant_f1_noise_sensitivity.csv). The expected outcome is a flat response: SNP F1 stays at 0.9972-0.9975 and indel F1 at 0.8084-0.8244 across the whole duration-variance sweep, indicating variant calling is largely insensitive to duration-sampler variance.

## Related tools

- **NanoSimFormer** (Signal simulator whose `simulate` subcommand accepts --duration-stdv; used to generate one POD5+FASTQ set per duration-variance value from the reference FASTA) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls each simulated read set into FASTQ/BAM via the --basecall option so variant calling can be run per sweep point) — github.com/nanoporetech/dorado
- **Docker (chobits323/nano-sim:v1.1)** (Recommended installation path that bundles pretrained checkpoints and Dorado for reproducible GPU simulation runs) — https://github.com/BioinfoSZU/NanoSimFormer

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --preset ont_r1041_dna_5khz --basecall --seed 42 --duration-stdv 5.0
```

## Evaluation signals

- SNP F1 remains flat across the sweep: 0.9972-0.9975 for every duration-stdv value (range < ~0.002).
- Indel F1 remains flat across the sweep: 0.8084-0.8244 for every duration-stdv value (range < ~0.016).
- The flat duration-variance result contrasts with the amplitude-noise control, where SNP F1 drops to ~0.7031 at sigma_amp=4.0 — if your duration sweep also degrades sharply, --noise-stdv was likely not held at default.
- Per-setting POD5+FASTQ outputs exist for every swept --duration-stdv value, all generated with --seed 42 so read sets are comparable.
- Results are tabulated per (sigma_amp, duration SD) setting in the sensitivity CSV.

## Limitations

- The paper only validates the flat-response expectation for the tested DNA (~2.45-8.49) and RNA004 (~3.87-13.42) ranges; extrapolating duration-stdv beyond these bounds is untested.
- Results depend on holding --noise-stdv at its default; a joint noise+duration sweep would confound the two sensitivity axes.
- Docker image requires an NVIDIA GPU with CUDA compute capability >= 8.x and NVIDIA driver >= 525; simulation is Linux-tested (Ubuntu 22.04).

## Evidence

- [readme] duration-stdv is an explicit simulator parameter: "--duration-stdv DURATION_STDV
                        duration sampler standard deviation (default: None)"
- [results] duration sweep yields flat SNP/indel F1 while amplitude noise degrades SNP F1 sharply: [evidence span withheld — anchor status: not_found]
- [readme] random seed fixed for comparability across the sweep: "--seed SEED           random seed (default: 42)"
- [abstract] baseline variant-calling F1 range the sweep should reproduce at nominal settings: "It faithfully recapitulates experimental variant calling performance across the five human samples, achieving F1-scores of 0.9953–0.9973 and 0.7862–0.8612 for single-nucleotide polymorphisms and"

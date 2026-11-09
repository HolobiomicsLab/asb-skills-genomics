---
name: match-rate-delta-tabulation
description: 'Use when ablating a learned simulation module: you need to replace seq2squiggle''s
  learned duration/noise samplers with static normal distributions and score the resulting
  signals against real reads via match-rate deltas.'
when_to_use_negative:
- The goal is a full seq2squiggle vs. other-simulator benchmark rather than an isolated
  ablation of the duration/noise sampler modules
- Real nanopore signals are unavailable — match-rate deltas require scoring simulated
  reads against real Drosophila signals
- You need absolute simulation-quality metrics rather than per-module deltas relative
  to the default learned configuration
tools:
- name: seq2squiggle
  role: Simulates nanopore signals from FASTA/FASTQ; its --noise-sampling/--noise-std
    and --duration-sampling/--dwell-mean/--dwell-std flags implement the static-normal
    ablations being compared
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: mamba
  role: Creates the seq2squiggle-dev environment used for the editable development
    install
- name: poetry
  role: Installs seq2squiggle in editable mode (poetry install) so the sampler modules
    can be modified and tested
- name: squigulator
  role: Reference simulator whose static parameterization (length mean=9.0, SD=4.0;
    noise mean=0.0, SD=1.0) the ablation distributions are matched to
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:96e75b69e90704a930d927d5b3af6bdd42829c8ac4453ae567af5ec5d868cd90
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Git
  - Python
  - conda
  - mamba
  - nox
  - poetry
  - pre-commit
  - seq2squiggle
  - squigulator
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# match-rate-delta-tabulation

## Summary

Quantifies the per-module effect of seq2squiggle's learned duration and noise samplers by regenerating evaluation signals with squigulator-style static normal distributions (duration mean=9.0, SD=4.0; noise mean=0.0, SD=1.0) and tabulating match-rate deltas against a real Drosophila nanopore signal set, reproducing the pattern of Supplementary Table S4.

## When to use

Use when ablating a learned simulation module: you need to replace seq2squiggle's learned duration/noise samplers with static normal distributions and score the resulting signals against real reads via match-rate deltas.

## When NOT to use

- The goal is a full seq2squiggle vs. other-simulator benchmark rather than an isolated ablation of the duration/noise sampler modules
- Real nanopore signals are unavailable — match-rate deltas require scoring simulated reads against real Drosophila signals
- You need absolute simulation-quality metrics rather than per-module deltas relative to the default learned configuration

## Inputs

- Drosophila nanopore reference (FASTA) for synthetic read generation
- Real Drosophila nanopore raw signals (BLOW5/SLOW5 or POD5) as the evaluation reference
- seq2squiggle source with duration-sampler and noise modules identified for ablation
- seq2squiggle pretrained model weights (--model)

## Outputs

- Simulated Drosophila read set under the static-normal ablation configuration (BLOW5/POD5)
- Simulated Drosophila read set under the default learned duration/noise samplers (BLOW5/POD5)
- match_rate_deltas.csv with per-module match-rate deltas

## How to apply

Set up the seq2squiggle-dev mamba environment (python 3, poetry, fortran-compiler, nox, pre-commit) and install the package editable with poetry install. Locate the duration-sampler and noise modules in the source and, via the CLI flags --noise-sampling False / --noise-std 1.0 and --duration-sampling False / --dwell-std 4.0 (with dwell-mean=9.0), replace both learned samplers with static normal distributions parameterized to match squigulator. Regenerate the Drosophila evaluation read set once with the static-normal configuration and once with the default learned samplers, then score both simulated BLOW5/POD5 read sets against real Drosophila nanopore signals to compute per-module match-rate deltas. Write the deltas as match_rate_deltas.csv; the expected validation pattern is a slightly positive delta for the noise module comparison and a slightly negative delta for the duration sampler comparison, mirroring Supplementary Table S4.

## Related tools

- **seq2squiggle** (Simulates nanopore signals from FASTA/FASTQ; its --noise-sampling/--noise-std and --duration-sampling/--dwell-mean/--dwell-std flags implement the static-normal ablations being compared) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **mamba** (Creates the seq2squiggle-dev environment used for the editable development install)
- **poetry** (Installs seq2squiggle in editable mode (poetry install) so the sampler modules can be modified and tested)
- **squigulator** (Reference simulator whose static parameterization (length mean=9.0, SD=4.0; noise mean=0.0, SD=1.0) the ablation distributions are matched to)

## Examples

```
seq2squiggle predict example.fasta -o example.blow5 --noise-std 1.0 --noise-sampling False --duration-sampling False --dwell-mean 9.0 --dwell-std 4.0
```

## Evaluation signals

- match_rate_deltas.csv shows a slightly positive delta for the noise-module comparison (learned noise beats static normal mean=0.0, SD=1.0)
- match_rate_deltas.csv shows a slightly negative delta for the duration-sampler comparison (learned sampler scores lower than static length mean=9.0, SD=4.0)
- The delta table pattern reproduces Supplementary Table S4
- Both simulated read sets are generated from the same Drosophila reference/reads so the only varying factor is the sampler configuration
- Environment check: `seq2squiggle version` resolves after `poetry install` in the seq2squiggle-dev environment

## Limitations

- seq2squiggle requires Python >= 3.10 and compatible pretrained model weights (auto-download via --model if not supplied)
- GPU inference is recommended; CPU-only runs are substantially slower for high sample counts
- The deltas are only 'slight' per the reported finding — do not expect large separation between learned and static configurations

## Evidence

- [supplementary] The learned noise module in seq2squiggle generated signals with a slightly higher match rate than static distributions (Supplementary Table S4): [evidence span withheld — anchor status: not_found]
- [readme] Noise sampler and duration sampler are togglable: [evidence span withheld — anchor status: external_doc]
- [readme] Static noise/duration CLI flags: [evidence span withheld — anchor status: external_doc]
- [readme] Duration-sampler static flag: [evidence span withheld — anchor status: external_doc]
- [methods] Editable dev install: "poetry install
seq2squiggle version"

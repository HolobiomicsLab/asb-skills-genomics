---
name: duration-distribution-parameterization
description: Use when you need to quantify how much a learned duration sampler or
  noise module in a nanopore signal simulator contributes to signal fidelity versus
  static distribution baselines.
when_to_use_negative:
- When the goal is production simulation quality rather than ablation of sampler components
  — keep the default learned samplers enabled
- When the source material is already a feature table, eventalign table, or basecalled
  output and no signal simulation is required
- When evaluating a different simulator (e.g. squigulator itself) whose CLI lacks
  the --noise-sampling/--duration-sampling static-distribution switches
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: seq2squiggle
  role: Transformer-based simulator whose noise and duration samplers are toggled
    off (--noise-sampling False, --duration-sampling False) to run static-normal ablations
    and default configurations
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- name: mamba
  role: Creates the seq2squiggle-dev conda environment for the editable dev install
- name: poetry
  role: Installs the modified seq2squiggle package in editable mode (poetry install)
    inside the dev environment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae744
    title: End-to-end simulation of nanopore sequencing signals with feed-forward
      transformers
  source_content_hash: sha256:63f238a3c7a4cd95805273eb5d3eb084fbfbef36dd3cf2a4af5b81f544f2835a
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
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# duration-distribution-parameterization

## Summary

Benchmark seq2squiggle's learned duration and noise samplers against static normal distributions (squigulator-style: length mean=9.0, SD=4.0; noise mean=0.0, SD=1.0) by regenerating simulated Drosophila nanopore signals under both configurations and comparing per-module match rates against real signals.

## When to use

Use when you need to quantify how much a learned duration sampler or noise module in a nanopore signal simulator contributes to signal fidelity versus static distribution baselines.

## When NOT to use

- When the goal is production simulation quality rather than ablation of sampler components — keep the default learned samplers enabled
- When the source material is already a feature table, eventalign table, or basecalled output and no signal simulation is required
- When evaluating a different simulator (e.g. squigulator itself) whose CLI lacks the --noise-sampling/--duration-sampling static-distribution switches

## Inputs

- Reference FASTA file of the Drosophila genome (evaluation reads sampled from it)
- Real Drosophila nanopore raw signals (SLOW5/BLOW5 or POD5) as ground truth
- Pretrained seq2squiggle model weights (--model, auto-downloaded if omitted)

## Outputs

- Two simulated read sets in BLOW5/POD5 format: one static-normal configuration, one default learned-sampler configuration
- match_rate_deltas.csv tabulating per-module match-rate deltas
- Validation pattern matching Supplementary Table S4: slightly positive delta for the noise module, slightly negative delta for the duration sampler

## How to apply

Set up the seq2squiggle-dev environment (mamba create -n seq2squiggle-dev python=3 poetry fortran-compiler nox pre-commit) and install the package in editable mode with poetry install. Locate the duration-sampler and noise modules in the source and, instead of modifying code, use the CLI switches --noise-sampling False and --duration-sampling False with static parameters: --noise-std 1.0 (noise mean 0.0, SD 1.0, matching squigulator) and a gamma-replacing static duration normal with mean 9.0 and SD 4.0 (--dwell-mean / --dwell-std). Regenerate the Drosophila evaluation read set under the static-normal configuration and again with the default learned samplers, then score both simulated read sets against real Drosophila nanopore signals to compute per-module match-rate deltas. The rationale is that the learned noise module better captures noise patterns resembling real experimental data, so its delta should be slightly positive while the learned duration sampler's delta is slightly negative relative to the static distributions.

## Related tools

- **seq2squiggle** (Transformer-based simulator whose noise and duration samplers are toggled off (--noise-sampling False, --duration-sampling False) to run static-normal ablations and default configurations) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle
- **mamba** (Creates the seq2squiggle-dev conda environment for the editable dev install)
- **poetry** (Installs the modified seq2squiggle package in editable mode (poetry install) inside the dev environment)

## Examples

```
seq2squiggle predict example.fasta -o static.blow5 --noise-sampling False --noise-std 1.0 --duration-sampling False --dwell-mean 9.0 --dwell-std 4.0
```

## Evaluation signals

- The learned noise module produces a slightly higher match rate than the static normal noise distribution (noise mean=0.0, SD=1.0) — a slightly positive delta
- The learned duration sampler produces a slightly lower match rate delta in the tabulated comparison relative to the static normal (mean=9.0, SD=4.0) duration distribution
- match_rate_deltas.csv reproduces the directionality pattern of Supplementary Table S4
- Both simulated read sets are generated for the same Drosophila reference/read input, differing only in sampler configuration
- Environment check: seq2squiggle version runs successfully after poetry install in the seq2squiggle-dev environment

## Limitations

- The expected deltas are only slight — the learned noise module 'slightly higher' match rate — so differences are small and sensitive to evaluation noise
- Static distributions replace the gamma duration sampler and learned noise module only for this ablation; results do not reflect the simulators' best configurations
- seq2squiggle requires pretrained model weights compatible with the chosen chemistry; testing workflow was not implemented in the source repo

## Evidence

- [readme] README noise options: [evidence span withheld — anchor status: external_doc]
- [readme] README static noise example: [evidence span withheld — anchor status: external_doc]
- [readme] README static duration example: [evidence span withheld — anchor status: external_doc]
- [methods] article dev environment setup: "mamba create -n seq2squiggle-dev python=3 poetry fortran-compiler nox pre-commit"
- [supplementary] card finding on learned noise: [evidence span withheld — anchor status: not_found]
- [abstract] article scope: "Unlike existing simulators that rely on static k-mer models, our approach learns sequential contextual information from segmented signal data"

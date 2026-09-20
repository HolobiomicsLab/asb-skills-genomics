---
name: per-read-score-consensus-modeling
description: Use when per-read methylation score files (ID, Chr, Pos, Strand, Score)
  from multiple nanopore methylation callers are available and single-tool predictions
  show poor agreement with expected methylation frequencies.
when_to_use_negative:
- When only a single tool's per-read scores are available — the consensus requires
  predictions from two or more tools
- When time is constrained and no GPU is available, since combining tools is 'impractical
  for routine analyses, due to the running times for some methods without GPU support'
edam_operation: http://edamontology.org/operation_3208
tools:
- name: METEORE
  role: Provides the random-forest (combination_model_prediction.py) and multiple
    linear regression (meteore_reg.py) consensus models and Snakemake pipelines that
    standardize per-read outputs across tools
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Constituent tool whose per-read scores feed the consensus; best individual
    performer but GPU-dependent
  repo: https://github.com/nanoporetech/megalodon
- name: DeepSignal
  role: Constituent tool whose per-read scores are combined with Megalodon's for the
    best balance of accuracy and running time
  repo: https://github.com/bioinfomaticsCSU/deepsignal
- name: Snakemake
  role: Workflow engine running the per-tool methylation-calling pipelines that produce
    the per-read score files
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:a161f684e4032fa4ae9e76accb07547b91cf2e0a9628819af9d6d974a8142dc3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - DeepMod
  - DeepSignal
  - Guppy
  - METEORE
  - Megalodon
  - Nanopolish
  - Snakemake
  - Tombo
  edam_operation: http://edamontology.org/operation_3208
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# Per-read score consensus modeling

## Summary

Combines per-read 5mC methylation predictions from multiple nanopore tools (e.g. Megalodon and DeepSignal) into a consensus score using METEORE's random-forest or regression models, improving accuracy over any single tool for CpG methylation detection.

## When to use

Use when per-read methylation score files (ID, Chr, Pos, Strand, Score) from multiple nanopore methylation callers are available and single-tool predictions show poor agreement with expected methylation frequencies.

## When NOT to use

- When only a single tool's per-read scores are available — the consensus requires predictions from two or more tools
- When time is constrained and no GPU is available, since combining tools is 'impractical for routine analyses, due to the running times for some methods without GPU support'

## Inputs

- Per-read methylation score files (ID, Chr, Pos, Strand, Score) from each constituent tool (e.g. Megalodon, DeepSignal)
- Nanopore single-read fast5 files converted via multi_to_single_fast5
- Reference genome FASTA (e.g. data/ecoli_k12_mg1655.fasta for the example dataset)
- For regression consensus: WGBS-anchored training datasets of per-read scores from controlled methylation mixtures (figshare)

## Outputs

- Consensus per-read methylation score file from METEORE
- Per-site methylation frequency table (augmented BED: chromosome, start, end, coverage, methylation frequency) at 0.5 threshold
- Strand-combined per-site table averaging methylation frequencies and summing coverage per CpG

## How to apply

First generate per-read score files from each selected tool via METEORE's Snakemake pipelines (Nanopolish, DeepSignal, Tombo, Guppy) or single commands (Megalodon, DeepMod), ensuring a consistent ID/Chr/Pos/Strand/Score format. For the random-forest consensus, run combination_model_prediction.py with either the default (-m default) or optimized (-m optimized) model. For the regression consensus, run meteore_reg.py on WGBS-anchored training data, setting per-tool trunc_min/trunc_max log-difference score ranges (e.g. -5/5 and -50/50) and keeping the default --filter and MinMax score scaling. Convert consensus per-read predictions to per-site methylation frequencies with prediction_to_mod_frequency.py at the default 0.5 threshold, then optionally merge both strands by averaging frequencies and summing coverage per CpG. The rationale, per the benchmarking, is that individual tools trade off false positives vs false negatives, so combining Megalodon with DeepSignal struck a good balance of accuracy and running time.

## Related tools

- **METEORE** (Provides the random-forest (combination_model_prediction.py) and multiple linear regression (meteore_reg.py) consensus models and Snakemake pipelines that standardize per-read outputs across tools) — https://github.com/comprna/METEORE
- **Megalodon** (Constituent tool whose per-read scores feed the consensus; best individual performer but GPU-dependent) — https://github.com/nanoporetech/megalodon
- **DeepSignal** (Constituent tool whose per-read scores are combined with Megalodon's for the best balance of accuracy and running time) — https://github.com/bioinfomaticsCSU/deepsignal
- **Snakemake** (Workflow engine running the per-tool methylation-calling pipelines that produce the per-read score files)

## Examples

```
python combination_model_prediction.py -i score_files -o consensus_scores -m default; python prediction_to_mod_frequency.py -i consensus_scores -o sites.bed
```

## Evaluation signals

- The consensus output per-read file retains the ID, Chr, Pos, Strand, Score column schema expected by prediction_to_mod_frequency.py
- Per-site frequencies produced at the default 0.5 threshold fall in [0,1] and coverage values are non-negative integers
- In the WGBS comparison on nCATS NA12878 data, METEORE REG should attain the highest Pearson correlation (r = 0.8885) and lowest RMSE (0.1607) among all methods
- Correlation and RMSE of the consensus against WGBS CpG frequencies should exceed those of every individual constituent tool
- Running with --no_scaling should degrade results relative to default MinMax scaling, confirming scaling was active

## Limitations

- M.SssI methylated control has only ~95% efficiency, lowering agreement with expected methylation frequency
- Discrepancies may stem from detection differences between nanopore and WGBS or disparate model types and training regimes of the tools
- Combining multiple tools may be impractical for routine analyses due to running times of methods without GPU support
- DeepMod per-site augmented BED output was not yet supported in the Mar-2021 METEORE update

## Evidence

- [abstract] including a consensus approach, METEORE ( https://github.com/comprna/METEORE ), based on the combination of the predictions from two or more tools: "including a consensus approach, METEORE ( https://github.com/comprna/METEORE ), based on the combination of the predictions from two or more tools"
- [readme] filter parameter quote: "--filter FLOAT       Percentage of total reads to remove around mean score, default -1 (disabled)"
- [readme] truncation parameters quote: "The trunc_min and trunc_max parameters can be used to restrict the useful range of log difference scores on a per-tool basis"
- [discussion] best combination quote: "The combination of Megalodon and DeepSignal struck a good balance for accuracy and running times but would require a GPU for efficiency."
- [readme] strand merging quote: "we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage"

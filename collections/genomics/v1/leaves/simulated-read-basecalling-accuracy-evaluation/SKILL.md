---
name: simulated-read-basecalling-accuracy-evaluation
description: Use when you have simulated ONT nanopore signals (e.g. POD5 from NanoSimFormer's
  simulate with preset ont_r1041_dna_5khz) and need a quantitative readout of signal
  fidelity via basecalling accuracy.
when_to_use_negative:
- You only need raw signal-level fidelity (e.g. per-base dwell or level distributions)
  without a sequence-level accuracy readout — basecalling adds GPU cost without adding
  information
- Your input is real experimental reads rather than simulated signals — Dorado basecalling
  is then a routine re-basecall, not a simulator-fidelity evaluation
- The simulated reads were generated without a known source reference/read sequence,
  making per-read accuracy against ground truth impossible to compute
edam_operation: http://edamontology.org/operation_3184
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoSimFormer
  role: Generates the simulated nanopore signals under evaluation; its simulate subcommand
    with --basecall triggers Dorado basecalling of simulated reads
  repo: https://github.com/BioinfoSZU/NanoSimFormer
- name: Dorado
  role: Basecalls the simulated reads into FASTQ/BAM so per-read accuracy can be computed
    against the source sequence
  repo: github.com/nanoporetech/dorado
- name: seq2squiggle
  role: Baseline simulator whose accuracy range (comparable to the 93.18–97.69% ablation
    result) serves as the comparison anchor
  repo: github.com/ZKI-PH-ImageAnalysis/seq2squiggle
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btag402
    title: 'NanoSimFormer: an end-to-end transformer-based nanopore signal simulator
      with basecaller guidance'
  source_content_hash: sha256:2d4cb40169c03a2a5eb42bf01d3c3ccd1a9a2df84cb54367aba43823bb4a4418
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BioinfoSZU/NanoSimFormer
  tools:
  - Docker
  - Dorado
  - NanoSimFormer
  - Python
  - conda
  - flash-attn
  - pip
  - seq2squiggle
  edam_operation: http://edamontology.org/operation_3184
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# simulated-read-basecalling-accuracy-evaluation

## Summary

Evaluate the fidelity of simulated nanopore signals by basecalling simulated reads with Dorado and computing median read basecalling accuracy, the primary metric used by NanoSimFormer (full basecaller-guided model >99%; ablated/seq2squiggle-comparable models 93.18–97.69%).

## When to use

Use when you have simulated ONT nanopore signals (e.g. POD5 from NanoSimFormer's simulate with preset ont_r1041_dna_5khz) and need a quantitative readout of signal fidelity via basecalling accuracy.

## When NOT to use

- You only need raw signal-level fidelity (e.g. per-base dwell or level distributions) without a sequence-level accuracy readout — basecalling adds GPU cost without adding information
- Your input is real experimental reads rather than simulated signals — Dorado basecalling is then a routine re-basecall, not a simulator-fidelity evaluation
- The simulated reads were generated without a known source reference/read sequence, making per-read accuracy against ground truth impossible to compute

## Inputs

- Simulated nanopore signal reads (POD5) produced by `python -m nano_signal_simulator simulate` with preset ont_r1041_dna_5khz or ont_rna004_4khz
- Reference genome FASTA or source basecalled FASTQ used as simulation input for accuracy computation
- NanoSimFormer checkpoint (e.g. ablated model from training_output/weights) to be evaluated

## Outputs

- FASTQ file of basecalled simulated reads (produced by Dorado via --basecall)
- Optionally BAM files of basecalled reads via --emit-bam
- Median read basecalling accuracy (e.g. >99% for the full model, 93.18–97.69% for the basecaller-guidance-ablated model)

## How to apply

After generating simulated reads with NanoSimFormer's simulate subcommand, enable the --basecall option so Dorado automatically basecalls the simulated signals into a FASTQ file (use --emit-bam if BAM output is desired). Compare each basecalled read to its source reference or read sequence to compute per-read basecalling accuracy, then summarize as the median. Interpret the median against established anchors: a full basecaller-guided NanoSimFormer model should exceed 99% median accuracy for DNA R10.4.1, whereas an ablated model trained without the basecaller-guided CTC loss should fall to 93.18–97.69%, comparable to seq2squiggle. This metric is the validation signal for ablation studies: it isolates whether high fidelity is driven by the basecaller-guided training objective rather than the Transformer architecture itself.

## Related tools

- **NanoSimFormer** (Generates the simulated nanopore signals under evaluation; its simulate subcommand with --basecall triggers Dorado basecalling of simulated reads) — https://github.com/BioinfoSZU/NanoSimFormer
- **Dorado** (Basecalls the simulated reads into FASTQ/BAM so per-read accuracy can be computed against the source sequence) — github.com/nanoporetech/dorado
- **seq2squiggle** (Baseline simulator whose accuracy range (comparable to the 93.18–97.69% ablation result) serves as the comparison anchor) — github.com/ZKI-PH-ImageAnalysis/seq2squiggle

## Examples

```
python -m nano_signal_simulator simulate --input chr22.fasta --output out --mode Reference --sample-reads 1000 --preset ont_r1041_dna_5khz --basecall --seed 42
```

## Evaluation signals

- Full basecaller-guided NanoSimFormer model yields median basecalling accuracy exceeding 99% for ONT DNA R10.4.1
- Basecaller-guidance-ablated model (identical Transformer, event-level mean and noise losses only) yields median accuracy within the reported 93.18–97.69% range, comparable to seq2squiggle
- Dorado produces a FASTQ file for every simulated read, and per-read accuracy is computable by aligning each basecall to its source reference/read sequence
- Accuracy is summarized as the median across reads (not the mean), matching the article's reported metric

## Limitations

- Median basecalling accuracy depends on the simulator checkpoint quality — an ablated or under-trained model will fall far below the >99% anchor and this is an expected outcome of the ablation, not a pipeline failure
- Requires an NVIDIA GPU with CUDA compute capability >= 8.x and NVIDIA driver >= 525 for both simulation and Dorado basecalling
- Evaluation is anchored to DNA R10.4.1 (and RNA004) presets; other chemistries require the model-extension workflow before accuracy anchors apply

## Evidence

- [methods] Dorado automatically basecalls simulated reads into a FASTQ file after signal simulation: [evidence span withheld — anchor status: not_found]
- [abstract] Median basecalling accuracy exceeding 99% for ONT DNA R10.4.1: "NanoSimFormer achieves a median basecalling accuracy exceeding 99% and Q-scores above 22.8 for Oxford Nanopore Technologies’ latest DNA R10.4.1 and direct RNA sequencing"
- [results] Ablated model degrades to median read accuracies of 93.18–97.69%, comparable to seq2squiggle: [evidence span withheld — anchor status: not_found]
- [readme] Supports ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation: "It supports both ONT DNA (R10.4.1) and direct-RNA (RNA004) signal simulation, enabling users to generate synthetic POD5 files from references or existing basecalled reads."

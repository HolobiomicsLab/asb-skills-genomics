---
name: classification-auc-benchmarking
description: Use when benchmarking a binary methylation/site classifier (e.g. CHEUI-solo
  Model 2) on held-out IVT signals and you need transcript-site-level AUC and FPR
  under known ground truth.
when_to_use_negative:
- Data is from cell transcriptomes without known ground-truth methylation status —
  the article notes these require very strict cut-offs to maintain low expected FDR,
  and sensitivity there cannot be validated against true labels
- You need differential methylation between two conditions — that is CHEUI-diff's
  task, not the Model 2 site-classification benchmark
- Read-level (single-molecule) accuracy is the target — that is assessed separately
  from the transcript-site-level Model 2 evaluation
tools:
- name: CHEUI
  role: Two-stage deep learning tool whose Model 2 (CNN site-level classifier on Model
    1 per-read probability distributions) is the benchmarked classifier
  repo: https://github.com/comprna/CHEUI
- name: nanopore direct RNA sequencing
  role: Source modality whose observed and expected ionic-current signals feed CHEUI's
    single-molecule, transcript-site, and stoichiometry predictions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:7fb2cf561609d56541be8df0a663a93c706f2c60d8359ef26de8534b76cbc8d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - nanopore direct RNA sequencing
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# classification-auc-benchmarking

## Summary

Benchmark a site-level RNA-modification classifier by simulating controlled modified/unmodified mixtures from in-vitro transcript (IVT) read signals and measuring ROC-AUC and per-site false positive rate, as done for CHEUI-solo Model 2 (AUC 0.92 for m6A, 0.953 for m5C at P>0.99).

## When to use

Use when benchmarking a binary methylation/site classifier (e.g. CHEUI-solo Model 2) on held-out IVT signals and you need transcript-site-level AUC and FPR under known ground truth.

## When NOT to use

- Data is from cell transcriptomes without known ground-truth methylation status — the article notes these require very strict cut-offs to maintain low expected FDR, and sensitivity there cannot be validated against true labels
- You need differential methylation between two conditions — that is CHEUI-diff's task, not the Model 2 site-classification benchmark
- Read-level (single-molecule) accuracy is the target — that is assessed separately from the transcript-site-level Model 2 evaluation

## Inputs

- Held-out modified and unmodified IVT read signals (Jenjaroenpun et al. direct RNA sequencing set, disjoint from Model 1 training/validation/test)
- CHEUI-solo Model 1 per-read probability distributions for each IVT read
- Simulation settings: 50% modified/unmodified prior, coverage 0–100 with linear-decay weights (10 − coverage) × 0.01 + 0.9, ~1.5 million synthetic sites per modification
- CHEUI-solo Model 2 trained model files (e.g. CHEUI_m6A_model2.h5, CHEUI_m5C_model2.h5)

## Outputs

- Site-level ROC-AUC per modification (target ≈ 0.92 for m6A, ≈ 0.953 for m5C)
- Per-site false positive rate on fully unmodified sites at P>0.99 (target ≈ 0.00074 for m6A, ≈ 0.00034 for m5C)
- CHEUI-solo Model 2 site-level methylation probability calls over the 10% held-out test split

## How to apply

Load held-out modified and unmodified IVT read signals (Jenjaroenpun et al. set, disjoint from Model 1 training/validation/test) plus their CHEUI-solo Model 1 per-read probability distributions. Simulate ~1.5 million synthetic sites per modification, choosing modified vs unmodified with 50% probability, drawing coverage 0–100 and stoichiometry with linear-decay weights (10 − coverage) × 0.01 + 0.9 via python random.choices, and assigning per-read Model 1 probabilities from the matching modified/unmodified pool. Split sites 9:1 into training and testing sets, then run CHEUI-solo Model 2, the CNN binary classifier that consumes per-site distributions of Model 1 read probabilities. Call a site positive when the Model 2 probability exceeds 0.99 and negative otherwise, then compute site-level ROC-AUC per modification and per-site FPR on fully unmodified sites. Accept the run when AUC ≈ 0.92 (m6A) and 0.953 (m5C) and FPR at P>0.99 ≈ 0.00074 (m6A) and 0.00034 (m5C).

## Related tools

- **CHEUI** (Two-stage deep learning tool whose Model 2 (CNN site-level classifier on Model 1 per-read probability distributions) is the benchmarked classifier) — https://github.com/comprna/CHEUI
- **nanopore direct RNA sequencing** (Source modality whose observed and expected ionic-current signals feed CHEUI's single-molecule, transcript-site, and stoichiometry predictions)

## Examples

```
nanopolish eventalign -t 48 --reads <read fastq> --bam <sorted-bam-file> --genome <transcriptome fasta> --scale-events --signal-index --samples --print-read-names > nanopolish_out.txt
```

## Evaluation signals

- Reproduced site-level ROC-AUC ≈ 0.92 for m6A and ≈ 0.953 for m5C on the held-out 10% split of synthetic IVT sites
- Per-site FPR on fully unmodified sites at P>0.99 ≈ 0.00074 (m6A) and ≈ 0.00034 (m5C)
- Class balance of simulated sites is 50% modified / 50% unmodified, with coverage 0–100 drawn using the linear-decay weights (10 − coverage) × 0.01 + 0.9
- Synthetic site count is ~1.5 million per modification, split 9:1 into training/testing
- Held-out IVT reads are disjoint from the CHEUI-solo Model 1 training/validation/test sets (Jenjaroenpun et al. data)

## Limitations

- Very strict cut-offs used on cell transcriptome sites to maintain low expected FDR reduce CHEUI sensitivity
- Many sites identified by orthogonal methods had high CHEUI probabilities below the set thresholds; relaxing thresholds would recover more sites but at the cost of potential false positives
- Nanopore signal variability can make modified vs non-modified read differences comparable to variation among non-modified reads, limiting detection

## Evidence

- [methods] CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads: "CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads"
- [intro] CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs): "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [abstract] CHEUI achieves high single-molecule, transcript-site, and stoichiometry accuracies in tests using synthetic RNA standards: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"
- [discussion] given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR): "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)"
- [readme] CHEUI uses a two-stage deep learning method to detect m6A and m5C transcriptome-wide at single-read and single-site resolution: [evidence span withheld — anchor status: external_doc]

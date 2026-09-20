---
name: nanopore-probability-distribution-modeling
description: Use when you have CHEUI-solo Model 1 per-read probability distributions
  grouped by transcript site and need site-level m6A/m5C modification probability
  and stoichiometry in a single condition without a knock-out/control sample.
when_to_use_negative:
- When the goal is single-read (single-molecule) modification calls rather than transcript-site-level
  prediction — that is CHEUI-solo Model 1's role
- When detecting differential methylation between two conditions — use CHEUI-diff
  instead
- When the signal variability between modified and non-modified reads is expected
  to be comparable to variation among non-modified reads (e.g. densely modified rRNA
  with signal interference at distances <5 nt), site-level accuracy will be limited
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CHEUI
  role: Provides CHEUI-solo Model 1 read-level probabilities and the Model 2 CNN classifier
    for site-level m6A/m5C prediction
  repo: https://github.com/comprna/CHEUI-public
- name: Nanopolish
  role: Resquiggles nanopore signals to aligned transcripts (eventalign with --scale-events
    --signal-index --samples) to produce the per-read signals Model 1 consumes upstream
  repo: https://nanopolish.readthedocs.io/en/latest/
- name: Minimap2
  role: Aligns basecalled reads to the reference transcriptome (-ax map-ont -k14)
    before signal-to-transcript alignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:665da49b9e8a35c2015db7648d0ae00d2d246c49234323a18a17da4f4ec228ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI-public
  tools:
  - CHEUI
  - Minimap2
  - Nanopolish
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# nanopore-probability-distribution-modeling

## Summary

Second-stage modeling in CHEUI that classifies transcript-site methylation (m6A, m5C) by feeding a CNN binary classifier with the per-site distribution of Model 1 per-read modification probabilities from nanopore direct RNA sequencing, yielding site-level probabilities, stoichiometry, and controlled false-positive rates.

## When to use

Use when you have CHEUI-solo Model 1 per-read probability distributions grouped by transcript site and need site-level m6A/m5C modification probability and stoichiometry in a single condition without a knock-out/control sample.

## When NOT to use

- When the goal is single-read (single-molecule) modification calls rather than transcript-site-level prediction — that is CHEUI-solo Model 1's role
- When detecting differential methylation between two conditions — use CHEUI-diff instead
- When the signal variability between modified and non-modified reads is expected to be comparable to variation among non-modified reads (e.g. densely modified rRNA with signal interference at distances <5 nt), site-level accuracy will be limited

## Inputs

- Sorted CHEUI-solo Model 1 per-read probability file grouped by transcript site (m6A or m5C)
- Trained CHEUI Model 2 .h5 weights (CHEUI_m6A_model2.h5 / CHEUI_m5C_model2.h5)
- Held-out modified and unmodified IVT read signals (Jenjaroenpun et al.) with Model 1 probability distributions for training/benchmarking

## Outputs

- Per-transcript-site methylation probabilities from CHEUI-solo Model 2
- Site-level stoichiometry estimates at transcript reference sites
- Site-level ROC-AUC (0.92 for m6A, 0.953 for m5C) and per-site FPR at P>0.99 (0.00074 m6A, 0.00034 m5C) on synthetic test sites

## How to apply

Train CHEUI-solo Model 2, a CNN binary classifier, on controlled mixtures of modified and unmodified reads: sample held-out IVT reads from Jenjaroenpun et al. (disjoint from Model 1 training sets), simulate ~1.5 million synthetic sites per modification with 50% modified/unmodified draws, coverage 0–100 and stoichiometry drawn with linear-decay weights ((10 - coverage) x 0.01 + 0.9), and assign per-read Model 1 probabilities from the matching modified or unmodified pool. Split synthetic sites 9:1 into training and testing sets and fit separate Model 2 networks for m6A (CHEUI_m6A_model2.h5) and m5C (CHEUI_m5C_model2.h5). At prediction time, aggregate Model 1 read-level probabilities per transcript site, feed the resulting distribution to Model 2, and call a site positive when the site-level probability exceeds 0.99. Benchmark against held-out mixture test sets by computing site-level ROC-AUC and the per-site false positive rate on fully unmodified sites at P>0.99.

## Related tools

- **CHEUI** (Provides CHEUI-solo Model 1 read-level probabilities and the Model 2 CNN classifier for site-level m6A/m5C prediction) — https://github.com/comprna/CHEUI-public
- **Nanopolish** (Resquiggles nanopore signals to aligned transcripts (eventalign with --scale-events --signal-index --samples) to produce the per-read signals Model 1 consumes upstream) — https://nanopolish.readthedocs.io/en/latest/
- **Minimap2** (Aligns basecalled reads to the reference transcriptome (-ax map-ont -k14) before signal-to-transcript alignment)

## Examples

```
python3 ../scripts/CHEUI_predict_model2.py --probabilities results_m6A_model1.sorted.txt --out results_m6A_model2 --DL_model ../CHEUI_trained_models/CHEUI_m6A_model2.h5
```

## Evaluation signals

- Reproduced site-level ROC-AUC on the IVT mixture test set is approximately 0.92 for m6A and 0.953 for m5C
- Per-site false positive rate on fully unmodified sites at probability threshold P>0.99 is approximately 0.00074 (m6A) and 0.00034 (m5C)
- Synthetic site simulation matches the paper's parameters: ~1.5 million sites per modification, 50% modified/unmodified, coverage 0–100 with linear-decay weights, 9:1 train/test split
- Output files contain one entry per transcript reference site with a Model 2 probability, and positive calls all exceed the 0.99 threshold

## Limitations

- Very strict cut-offs (P>0.99) are required on cell transcriptome sites to maintain low expected FDR given the large number of sites tested, which reduces CHEUI sensitivity
- Many sites identified by orthogonal methods (e.g. m6ACE-seq/miCLIP union) have high CHEUI probabilities below the set thresholds; relaxing thresholds would recover more sites but at the cost of potential false positives
- Nanopore signal variability makes modified vs non-modified read differences often comparable to differences observed among non-modified reads, limiting sensitivity
- Model 2 benchmarks derive from simulated synthetic mixtures, so real-transcriptome behavior depends on the strict cutoffs and independent validation

## Evidence

- [methods] CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads: "CHEUI-solo Model 2 was trained using controlled mixtures of modified and unmodified reads"
- [intro] Two-stage neural network trained on IVT signals: "CHEUI is based on a two-stage neural network and was trained using read signals generated from in-vitro transcripts (IVTs)"
- [readme] Model 1 and Model 2 use different m6A/m5C model files: [evidence span withheld — anchor status: external_doc]
- [discussion] Strict cutoffs to maintain low FDR: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)"
- [abstract] High transcript-site accuracy claims: "CHEUI processes observed and expected nanopore direct RNA sequencing signals to achieve high single-molecule, transcript-site, and stoichiometry accuracies"

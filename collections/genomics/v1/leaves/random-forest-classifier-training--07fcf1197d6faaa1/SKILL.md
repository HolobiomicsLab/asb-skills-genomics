---
name: random-forest-classifier-training
description: Use when you have aligned PacBio CCS reads with kinetic tags and need
  a baseline supervised classifier to test whether kinetic signals discriminate two
  base states and to rank feature contributions.
when_to_use_negative:
- The goal is single-nucleotide Z calling on production reads — use Z-Calling's MLP
  base-calling module (k11-mixed-AZ model) instead of the RFC baseline
- Context k-mers contain adjacent A/Z bases or fewer than 10 observations in one condition,
  violating the one-target-base and minimum-observation filters
- Comparing architectures (MLP, CNN, BiLSTM, GCN) rather than establishing the RFC
  kinetic baseline
edam_operation: http://edamontology.org/operation_3948
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0080
tools:
- name: Z-Calling
  role: Provides kinetic extraction from filtered CCS reads and the ZA_signal_feature_importance.py
    pipeline used to train the Random Forest classifiers and rank feature importances
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Generates HiFi BAMs with --keep-kinetics so pulse width (PW) and inter-pulse
    duration (IPD) signals are preserved for feature extraction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:3a65c06d994ff7ae4e5ffbb4f341f61b63860819b79321fa851b5e1bf4046392
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  edam_operation: http://edamontology.org/operation_3948
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# random-forest-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains Random Forest classifiers on PacBio kinetic features (PW and IPD) extracted from 21-mer context k-mers to discriminate Z from A bases, and exports a feature-importance matrix that quantifies which kinetic signals drive A/Z discrimination.

## When to use

Use when you have aligned PacBio CCS reads with kinetic tags and need a baseline supervised classifier to test whether kinetic signals discriminate two base states and to rank feature contributions.

## When NOT to use

- The goal is single-nucleotide Z calling on production reads — use Z-Calling's MLP base-calling module (k11-mixed-AZ model) instead of the RFC baseline
- Context k-mers contain adjacent A/Z bases or fewer than 10 observations in one condition, violating the one-target-base and minimum-observation filters
- Comparing architectures (MLP, CNN, BiLSTM, GCN) rather than establishing the RFC kinetic baseline

## Inputs

- Filtered PacBio CCS BAM with kinetic tags (fi, ri, fp, rp) preserving PW and IPD
- Context k-mer feature matrix of PW/IPD values for 21-mers centered on A/Z target bases (≥10 observations per k-mer in both dZ-DNA and canonical-DNA conditions)
- Per-model fixed central 3-mer motif labels (Z-context vs A-context)

## Outputs

- Mean cross-validated RFC accuracy (expected ≈0.842 ± 0.018)
- Random Forest feature-importance matrix ranked by contribution (PW features vs IPD features)
- Trained per-motif Random Forest models for Z vs A discrimination

## How to apply

Extract PW and IPD values from filtered PacBio CCS reads using Z-Calling's kinetic extraction on 21 bp context k-mers centered on the target base. Restrict to context k-mers containing exactly one A/Z target base and at least 10 recorded observations in both dZ-DNA and canonical-DNA conditions, holding the central 3-mer motif fixed per model to control for sequence context. Train Random Forest classifiers with the ZA_signal_feature_importance.py pipeline to separate Z vs A contexts on the PW/IPD k-mer feature matrix, and record mean cross-validated accuracy. Export the feature-importance matrix ranked by contribution; the article's baseline achieved 0.842 ± 0.018 accuracy with pulse width (PW) features dominating over IPD, consistent with Z incorporation primarily affecting PW.

## Related tools

- **Z-Calling** (Provides kinetic extraction from filtered CCS reads and the ZA_signal_feature_importance.py pipeline used to train the Random Forest classifiers and rank feature importances) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Generates HiFi BAMs with --keep-kinetics so pulse width (PW) and inter-pulse duration (IPD) signals are preserved for feature extraction)

## Examples

```
python ZA_signal_feature_importance.py  # train RF on PW/IPD 21-mer k-mer matrix, output CV accuracy and ranked feature-importance matrix
```

## Evaluation signals

- Mean cross-validated accuracy falls within the reported benchmark range 0.842 ± 0.018
- PW-derived features dominate the feature-importance ranking over IPD-derived features
- Feature-importance matrix shows the downstream nucleotide exerting the strongest context influence
- Every k-mer in the training matrix has ≥10 observations in both dZ-DNA and canonical-DNA conditions and exactly one A/Z target base
- Central 3-mer motif is fixed within each trained model

## Limitations

- Z-induced kinetic shift (0.5–1.5-fold) overlaps substantially with canonical Adenine variance due to inherent PacBio kinetic measurement noise, capping achievable accuracy
- Analysis restricted to k-mers with one A/Z target base to avoid interference among adjacent A/Zs, so poly-A/Z contexts are not modeled
- Random Forest is the baseline only; it is outperformed by deep architectures (MLP, CNN, BiLSTM, GCN) benchmarked in the study

## Evidence

---
name: kmer-context-feature-engineering
description: Use when PacBio CCS reads with kinetic tags (fi, ri, fp, rp) must be
  converted into per-k-mer PW/IPD feature matrices to train classifiers that discriminate
  modified vs canonical bases in defined sequence contexts.
when_to_use_negative:
- Input reads lack kinetic tags or fail the Median_PW/Median_IPD 0.3-99th-percentile
  filter, so PW/IPD context features cannot be reliably extracted
- The analysis target is single-nucleotide Z/A probability calling on reads rather
  than controlled-context feature-importance modeling (use Z-Calling's k11-mixed-AZ
  MLP module instead)
- Context k-mers contain multiple adjacent A/Z bases, which the paper explicitly excludes
  to avoid interference among adjacent A/Zs
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3474
tools:
- name: Z-Calling
  role: Provides kinetic extraction from filtered CCS reads and the ZA_signal_feature_importance.py
    pipeline used to build k-mer feature matrices and train Random Forest classifiers
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source of reads carrying pulse width (PW) and inter-pulse duration (IPD) kinetic
    signals preserved via --keep-kinetics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:88d19f3860708372d7e60f6f2e55dd3f29f2bfc4cf1cc931995b9c1c33099916
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kmer-context-feature-engineering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds 21-mer sequence-context feature matrices centered on a target A/Z base from PacBio CCS kinetic signals (PW, IPD), restricted to k-mers with exactly one A/Z and ≥10 observations per condition, for machine-learning discrimination of dZ-DNA from canonical DNA.

## When to use

Use when PacBio CCS reads with kinetic tags (fi, ri, fp, rp) must be converted into per-k-mer PW/IPD feature matrices to train classifiers that discriminate modified vs canonical bases in defined sequence contexts.

## When NOT to use

- Input reads lack kinetic tags or fail the Median_PW/Median_IPD 0.3-99th-percentile filter, so PW/IPD context features cannot be reliably extracted
- The analysis target is single-nucleotide Z/A probability calling on reads rather than controlled-context feature-importance modeling (use Z-Calling's k11-mixed-AZ MLP module instead)
- Context k-mers contain multiple adjacent A/Z bases, which the paper explicitly excludes to avoid interference among adjacent A/Zs

## Inputs

- Filtered PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp; np >= 3, Median_PW/Median_IPD >= 0.3 and <= 99th percentile)
- Matched dZ-DNA and canonical-DNA sequencing datasets (e.g. amplicons with dZTP replacing dATP)
- Reference genome/fasta for alignment context

## Outputs

- 21-mer context k-mer feature matrix of PW and IPD values, restricted to k-mers with exactly one A/Z and >=10 observations per condition
- Random Forest feature-importance matrix ranked by contribution (e.g. from ZA_signal_feature_importance.py)
- Cross-validated classification accuracy per central 3-mer motif (benchmark: 0.842 +/- 0.018)

## How to apply

Extract pulse width (PW) and inter-pulse duration (IPD) values from filtered PacBio CCS reads (BAM np ≥3, ≥95% reference similarity, Median_PW/Median_IPD between 0.3 and the 99th percentile) over 21 bp context k-mers centered on the target base. Restrict to k-mers containing exactly one A/Z target base to avoid interference among adjacent A/Zs, and keep only k-mers with at least 10 recorded observations in both dZ-DNA and canonical-DNA conditions. Hold the central 3-mer motif fixed per model to control sequence context when training discriminative models. Feed the resulting PW/IPD k-mer feature matrix into Random Forest classifiers (e.g. ZA_signal_feature_importance.py) and export feature-importance rankings; the paper found PW features dominate over IPD, consistent with Z incorporation primarily affecting pulse width.

## Related tools

- **Z-Calling** (Provides kinetic extraction from filtered CCS reads and the ZA_signal_feature_importance.py pipeline used to build k-mer feature matrices and train Random Forest classifiers) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source of reads carrying pulse width (PW) and inter-pulse duration (IPD) kinetic signals preserved via --keep-kinetics)

## Examples

```
python ZA_signal_feature_importance.py --kmers 21 --conditions dZ,canonical --min_obs 10
```

## Evaluation signals

- Mean cross-validated Random Forest accuracy falls near the reported benchmark of 0.842 +/- 0.018 for A/Z kinetic discrimination
- PW-derived features dominate the feature-importance ranking over IPD-derived features, matching the finding that Z incorporation primarily affects pulse width
- Every retained context k-mer contains exactly one A/Z target base and has >=10 observations in both dZ-DNA and canonical-DNA conditions
- Downstream nucleotide exerts the strongest influence among positional context features in the importance matrix

## Limitations

- PacBio kinetic measurements are inherently noisy: the subtle Z-induced shift (0.5-1.5-fold) overlaps substantially with canonical Adenine variance, limiting hybrid sample quantification
- Only context k-mers with a single A/Z can be used; adjacent A/Z interference forces exclusion of multi-Z contexts
- Reads shorter than 500 bp or with fewer than 100 A+Z bases are excluded, constraining usable input
- Sequence-context dependence means fixed central 3-mer motifs must be modeled separately per model

## Evidence

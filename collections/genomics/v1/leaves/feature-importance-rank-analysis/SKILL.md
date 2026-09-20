---
name: feature-importance-rank-analysis
description: Use when analyzing PacBio HiFi/CCS kinetic signals (PW/IPD) to quantify
  which features distinguish a modified base (e.g., Z) from a canonical base (A) within
  fixed sequence contexts, and to rank feature importance.
when_to_use_negative:
- Data are BAM reads still needing preprocessing/kinetic-tag filtering — run Z-Calling's
  filter_bam.py first (requires fi/ri/fp/rp tags, Median_PW/Median_IPD ≥ 0.3 and ≤
  99th percentile, fn/rn ≥ 3)
- Goal is per-base Z calling or read-level dZ-DNA classification rather than feature-importance
  analysis — use Z-Calling's MLP and SVM modules instead
- Context k-mers contain multiple A/Z bases or fewer than 10 observations in one condition
  — adjacent A/Zs interfere and signal is insufficient
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: Z-Calling
  role: Provides kinetic extraction on 21 bp k-mers and the ZA_signal_feature_importance.py
    pipeline used for RFC training and feature-importance ranking
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Source of HiFi reads with preserved PW/IPD kinetics (--keep-kinetics) that
    form the feature matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:62e60ac2cb2a2b0e7e061c6f85fd3985962d4a4bd941b1cd0b9a1ef3542cf948
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# feature-importance-rank-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains Random Forest classifiers on PacBio CCS kinetic features (pulse width, IPD) to discriminate A vs Z (2,6-diaminopurine) bases in 21-mer sequence contexts, then ranks per-k-mer feature contributions to identify which nucleotide positions and signal types dominate Z discrimination.

## When to use

Use when analyzing PacBio HiFi/CCS kinetic signals (PW/IPD) to quantify which features distinguish a modified base (e.g., Z) from a canonical base (A) within fixed sequence contexts, and to rank feature importance.

## When NOT to use

- Data are BAM reads still needing preprocessing/kinetic-tag filtering — run Z-Calling's filter_bam.py first (requires fi/ri/fp/rp tags, Median_PW/Median_IPD ≥ 0.3 and ≤ 99th percentile, fn/rn ≥ 3)
- Goal is per-base Z calling or read-level dZ-DNA classification rather than feature-importance analysis — use Z-Calling's MLP and SVM modules instead
- Context k-mers contain multiple A/Z bases or fewer than 10 observations in one condition — adjacent A/Zs interfere and signal is insufficient

## Inputs

- Filtered PacBio CCS BAM files with kinetic tags (fi, ri, fp, rp) preserved via --keep-kinetics
- 21-mer context k-mer feature matrix of PW and IPD values centered on A/Z target bases
- Matched dZ-DNA and canonical-DNA condition datasets with per-k-mer observation counts

## Outputs

- Mean cross-validated RFC accuracy (baseline target: 0.842 ± 0.018)
- Ranked feature-importance matrix from the Random Forest model
- Identification of dominant signal type (PW vs IPD) and dominant nucleotide position

## How to apply

Extract PW and IPD values from filtered PacBio CCS reads using Z-Calling's kinetic extraction on 21 bp context k-mers centered on the target base. Restrict to context k-mers containing exactly one A/Z target base and at least 10 recorded observations in both dZ-DNA and canonical-DNA conditions, holding the central 3-mer motif fixed per model to control sequence context. Train Random Forest classifiers (via Z-Calling's ZA_signal_feature_importance.py) on the PW/IPD k-mer feature matrix to discriminate Z vs A contexts. Record mean cross-validated accuracy and export the feature-importance matrix ranked by contribution; the baseline expectation is ~0.842 ± 0.018 accuracy with PW features dominating over IPD features and the downstream nucleotide exerting the strongest influence.

## Related tools

- **Z-Calling** (Provides kinetic extraction on 21 bp k-mers and the ZA_signal_feature_importance.py pipeline used for RFC training and feature-importance ranking) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Source of HiFi reads with preserved PW/IPD kinetics (--keep-kinetics) that form the feature matrix)

## Examples

```
python py filter_bam.py -b RawBAM -o FilteredBAM; python ZA_signal_feature_importance.py  # RFC feature-importance pipeline (detailed flags in Supplementary Note 1)
```

## Evaluation signals

- Mean cross-validated accuracy falls in the reported baseline range (0.842 ± 0.018)
- PW-derived features outrank IPD-derived features in the importance matrix, consistent with Z incorporation primarily affecting pulse width
- Feature-importance matrix is exported and sortable by contribution for each fixed central 3-mer motif model
- Every analyzed context k-mer contains exactly one A/Z target base and ≥ 10 observations in both dZ-DNA and canonical-DNA conditions
- Downstream nucleotide shows the strongest positional influence among ranked features

## Limitations

- Z-induced kinetic shifts (0.5–1.5-fold) overlap substantially with canonical Adenine variance due to inherent PacBio kinetic noise
- Models are optimized for fully substituted dZ-genomes; hybrid reads with diluted kinetic signatures are harder to classify
- Detailed command lines and custom scripts are deferred to Supplementary Notes rather than the main text
- The molecular mechanism of Z-induced kinetic changes remains uncharacterized

## Evidence

- [full_text] RFC baseline models trained on 21-mer contexts with fixed central A/Z 3-mer motifs achieved an average accuracy of 0.842 ± 0.018, with pulse width (PW) emerging as the dominant discriminatory feature: [evidence span withheld — anchor status: not_found]
- [discussion] CCS reads must preserve kinetic signals before feature extraction: [evidence span withheld — anchor status: not_found]

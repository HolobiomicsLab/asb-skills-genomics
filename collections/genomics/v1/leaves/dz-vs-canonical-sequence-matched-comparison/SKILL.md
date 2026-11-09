---
name: dz-vs-canonical-sequence-matched-comparison
description: Use when quantifying how Z (2,6-diaminopurine) incorporation perturbs
  PacBio HiFi PW/IPD kinetics relative to canonical A, using sequence-matched k-mer
  contexts with exactly one A/Z target base per window.
when_to_use_negative:
- Goal is production Z/A base calling or dZ-DNA read classification with the pre-trained
  models (z-calling-base with k21/k11 models, or the SVM read classifier) rather than
  feature-importance analysis
- Context k-mer windows contain multiple adjacent A/Z bases — the paper explicitly
  excludes these to avoid interference among adjacent A/Zs
- Input BAMs lack kinetic tags (fi, ri, fp, rp) because CCS was run without --keep-kinetics,
  or reads fail the Median_PW/Median_IPD >=0.3 quality filter
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: Z-Calling
  role: Read filtering (filter_bam.py) and kinetic extraction of PW/IPD values from
    21 bp context k-mers in filtered PacBio CCS reads; provides the ZA_signal_feature_importance.py
    script for RFC training and feature-importance export
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Sequencing source of kinetic signals; BAMs must be produced with the --keep-kinetics
    option to preserve PW and IPD values
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:7c19a91d4687fd10eeaf591e0f614a74e55dd1f9d0d6bb4eb0afadd9b08b2cb8
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# dz-vs-canonical-sequence-matched-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares PacBio kinetic signals (pulse width, inter-pulse duration) between matched dZ-DNA and canonical-DNA reads on 21-mer contexts centered on a single A/Z target base, controlling sequence context (fixed central 3-mer) to isolate Z-induced kinetic shifts and quantify them via Random Forest feature importance.

## When to use

Use when quantifying how Z (2,6-diaminopurine) incorporation perturbs PacBio HiFi PW/IPD kinetics relative to canonical A, using sequence-matched k-mer contexts with exactly one A/Z target base per window.

## When NOT to use

- Goal is production Z/A base calling or dZ-DNA read classification with the pre-trained models (z-calling-base with k21/k11 models, or the SVM read classifier) rather than feature-importance analysis
- Context k-mer windows contain multiple adjacent A/Z bases — the paper explicitly excludes these to avoid interference among adjacent A/Zs
- Input BAMs lack kinetic tags (fi, ri, fp, rp) because CCS was run without --keep-kinetics, or reads fail the Median_PW/Median_IPD >=0.3 quality filter

## Inputs

- PacBio HiFi/CCS BAM files generated with --keep-kinetics, for matched dZ-DNA and canonical-DNA samples, containing kinetic tags (fi, ri, fp, rp)
- Reference genome FASTA for alignment/similarity filtering (e.g., E. coli BL21 or other matched host)
- Optional: example_data.tar.gz from the Z-Calling repo (100 dZ-DNA + 100 native E. coli amplicon reads) for smoke-testing the filter/call pipeline

## Outputs

- PW/IPD feature matrix over 21-mer contexts centered on a single A/Z target base (>=10 observations per k-mer in both conditions)
- Mean cross-validated RFC accuracy for Z vs A discrimination per fixed central 3-mer motif (benchmark: 0.842 +/- 0.018)
- Ranked Random Forest feature-importance matrix (PW vs IPD, by position) from ZA_signal_feature_importance.py

## How to apply

Filter PacBio CCS BAMs first: require kinetic tags (fi, ri, fp, rp), at least 3 passes on both strands (fn>=3, rn>=3), >=95% reference similarity, median PW/median IPD ratio >=0.3 and below the 99th percentile, length >=500 bp with >=100 A+Z bases, and >=60,000 reads per dataset. Extract PW and IPD values for 21 bp context k-mers centered on the target base using Z-Calling's kinetic extraction, keeping only k-mer contexts with exactly one A/Z (to avoid interference among adjacent A/Zs) and with at least 10 recorded observations in both dZ-DNA and canonical-DNA conditions. Hold the central 3-mer motif fixed per model so classification reflects kinetics rather than sequence composition. Train Random Forest classifiers via ZA_signal_feature_importance.py to discriminate Z vs A contexts, then export the feature-importance matrix ranked by contribution. The comparison is considered correct if mean cross-validated accuracy lands near the 0.842 +/- 0.018 benchmark and PW-derived features dominate IPD-derived ones, consistent with Z incorporation primarily affecting pulse width.

## Related tools

- **Z-Calling** (Read filtering (filter_bam.py) and kinetic extraction of PW/IPD values from 21 bp context k-mers in filtered PacBio CCS reads; provides the ZA_signal_feature_importance.py script for RFC training and feature-importance export) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Sequencing source of kinetic signals; BAMs must be produced with the --keep-kinetics option to preserve PW and IPD values)

## Examples

```
python py/filter_bam.py -b <RawBAM> -o <FilteredBAM>
```

## Evaluation signals

- Mean cross-validated RFC accuracy falls within the reported benchmark range 0.842 +/- 0.018
- PW-derived features dominate the feature-importance ranking over IPD-derived features, consistent with Z incorporation primarily affecting pulse width
- Every retained context k-mer contains exactly one A/Z target base and has >=10 observations in both dZ-DNA and canonical-DNA conditions
- All analyzed reads satisfy the filters: kinetic tags present, fn>=3 and rn>=3, Median_PW/Median_IPD >=0.3 and <99th percentile, length >=500 bp with >=100 A+Z bases
- Each model's dataset is restricted to a single fixed central 3-mer motif, so accuracy differences across models reflect sequence-context effects

## Limitations

- The subtle Z-induced kinetic shift (0.5-1.5-fold) overlaps substantially with canonical adenine variance, limiting quantification in hybrid samples
- The downstream nucleotide exerts the strongest influence within the 21-mer context, so results are sequence-dependent and must be reported per fixed central motif
- Detailed command lines and custom scripts (including ZA_signal_feature_importance.py usage) are deferred to Supplementary Note 1 rather than the main text
- The companion SVM read classifier is optimized for fully substituted dZ-genomes and misses hybrid reads with diluted kinetic signatures — not directly usable for mixed-sample ground truth here

## Evidence

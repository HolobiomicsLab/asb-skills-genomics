---
name: basecalled-feature-aggregation-across-reads
description: Use when per-read base-calling features (mismatch/deletion frequencies,
  base quality scores) from ONT direct RNA reads must be pooled into per-site summaries
  before scoring candidate m6A sites, especially across partial-methylation mixtures.
when_to_use_negative:
- Goal is single-read/single-molecule modification calling — EpiNano explicitly predicts
  per site, not per read
- Need to distinguish between different modification types (e.g. m1A vs m6A) — the
  algorithm cannot discriminate modification types
- Input data was base-called with a different basecaller version than the pretrained
  SVM model (e.g. Guppy data against Albacore 2.1.7-trained models)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: EpiNano
  role: Extracts per-k-mer base-calling 'error' features (mismatch, deletion, per-base
    quality, current intensity) from direct RNA reads; used in per-k-mer feature-table
    mode before per-site aggregation
  repo: https://github.com/novoalab/EpiNano
- name: scikit-learn
  role: Hosts the trained combined-feature linear SVM used to score the aggregated
    per-site feature vectors and compute ROC/AUC
- name: Oxford Nanopore Technologies (ONT)
  role: Source platform for direct RNA sequencing reads and their base-called FASTQ/FAST5
    features
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:c48be141cd02ce8c852e2642d8ba56c2cc7088908a9ad037ab153505a1da59c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - ONT platform
  - Oxford Nanopore Technologies (ONT)
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# Basecalled-feature aggregation across reads for per-site m6A scoring

## Summary

Aggregates per-read base-calling 'error' features (mismatch frequency, deletion frequency, per-base quality) extracted by EpiNano from ONT direct RNA reads into per-site summaries for every candidate m6A motif site, so that a trained combined-feature SVM can score sites rather than individual molecules.

## When to use

Use when per-read base-calling features (mismatch/deletion frequencies, base quality scores) from ONT direct RNA reads must be pooled into per-site summaries before scoring candidate m6A sites, especially across partial-methylation mixtures.

## When NOT to use

- Goal is single-read/single-molecule modification calling — EpiNano explicitly predicts per site, not per read
- Need to distinguish between different modification types (e.g. m1A vs m6A) — the algorithm cannot discriminate modification types
- Input data was base-called with a different basecaller version than the pretrained SVM model (e.g. Guppy data against Albacore 2.1.7-trained models)

## Inputs

- Per-read EpiNano feature table with mismatch frequency, deletion frequency, and base quality scores per RRACH k-mer position
- Read-to-site assignment for candidate m6A motif sites (RRACH 5-mers)
- Read-level labels distinguishing m6A-modified vs unmodified read pools (synthetic or WT/KO control)
- Trained combined-features linear SVM model (scikit-learn)

## Outputs

- Per-site aggregated feature table (site-level summaries of mismatch, deletion, base quality)
- Methylation-proportion vs AUC table (e.g. mixture_auc_results.csv) with ROC curves per ratio
- Per-site m6A modification scores/predictions from the combined-feature SVM

## How to apply

Extract per-k-mer base-calling features with EpiNano (v1.x) from direct RNA FASTQ/FAST5 base-called with the same basecaller as the training data (e.g. Albacore 2.1.7 or Guppy 3.1.5 pretrained models). Because the algorithm does not predict modifications in individual molecules, aggregate read-level mismatch/deletion frequencies and base quality scores into per-site summaries (e.g. per-site means via groupby) for every RRACH candidate motif site, pooling information from all reads mapping to the site. For partial-methylation settings, generate in-silico mixtures by subsampling modified and unmodified reads at 0, 5, 10, 25, 50, 75, 90, 95 and 100% methylation with a fixed random seed per ratio, then aggregate before scoring. Feed the per-site feature vectors into the trained combined-features linear SVM (scikit-learn) and compute ROC/AUC for modified-site detection; the expected signature is a methylation-proportion-dependent AUC, e.g. AUC = 0.72 at 25% methylation.

## Related tools

- **EpiNano** (Extracts per-k-mer base-calling 'error' features (mismatch, deletion, per-base quality, current intensity) from direct RNA reads; used in per-k-mer feature-table mode before per-site aggregation) — https://github.com/novoalab/EpiNano
- **scikit-learn** (Hosts the trained combined-feature linear SVM used to score the aggregated per-site feature vectors and compute ROC/AUC)
- **Oxford Nanopore Technologies (ONT)** (Source platform for direct RNA sequencing reads and their base-called FASTQ/FAST5 features)

## Examples

```
site_feats = reads.groupby(['site','ratio'])[['mismatch_freq','deletion_freq','base_q']].mean(); aucs[ratio] = roc_auc_score(labels, svm.decision_function(site_feats))
```

## Evaluation signals

- AUC of the SVM site scores at the 25% methylation ratio should fall near the reported 0.72
- Aggregated table must cover every candidate RRACH motif site, with one row per site (not per read) and pooled statistics over all reads mapping to that site
- Combined-feature SVM on the same aggregated features should reach ~88–91% in vitro classification accuracy, consistent with ~90% overall accuracy
- Methylation-proportion curve must be monotone-ish across 0–100% ratios, demonstrating dependence of performance on methylation proportion
- Feature schema check: mismatch frequency, deletion frequency, and per-base quality columns present; current intensity excluded from final models (not used to train EpiNano 1.1+ models)

## Limitations

- EpiNano has no per-read resolution: predictions aggregate information from all reads mapping to a site, so single-read modification detection is not achieved
- Performance depends on the methylation stoichiometry of the sample; low modification fractions reduce detection power (AUC 0.72 already at 25%)
- Direct RNA base-calling is 'error'-rich even for unmodified sequences; a matched control (WT vs KO/KD or demethylase-treated) is highly recommended for confident calls
- Only a fraction of known sites are recovered (~32% overall recovery), and feature contributions vary across k-mers so k-mer-specific training is needed to reduce false positives

## Evidence

- [discussion] Site-level aggregation, not per-read prediction: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [discussion] Error features used for detection: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] No per-read resolution (README): [evidence span withheld — anchor status: external_doc]
- [readme] Error-rich basecalling motivates control pairing (README): [evidence span withheld — anchor status: external_doc]
- [readme] Feature extraction modes (README): [evidence span withheld — anchor status: not_found]

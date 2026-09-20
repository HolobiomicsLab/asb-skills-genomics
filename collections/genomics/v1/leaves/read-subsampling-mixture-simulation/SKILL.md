---
name: read-subsampling-mixture-simulation
description: Use when you need to characterize how site-level m6A detection performance
  degrades as the fraction of modified reads per site drops — e.g., benchmarking EpiNano-SVM
  on heterogeneous or partial-methylation samples.
when_to_use_negative:
- You need per-read (single-molecule) modification calls — the algorithm explicitly
  aggregates information from all reads mapping to a site and cannot resolve individual
  molecules
- You are trying to distinguish modification types (e.g., m1A vs m6A) — the trained
  m6A models do not discriminate between different modification species
- The sample is fully modified or fully unmodified with no stoichiometry variation
  — a simple pairwise (WT vs KO) EpiNano-Error analysis is more appropriate than mixture
  simulation
tools:
- name: EpiNano
  role: Feature extraction (mismatch/deletion frequency, per-base quality) from base-called
    reads and SVM scoring of aggregated per-site features across mixture levels
  repo: github.com/enovoa/EpiNano
- name: ONT platform
  role: Source of native direct RNA sequencing reads (modified and unmodified pools)
    that are subsampled to construct the mixtures
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:6e9f6aba5f116fabb25a6475c530a04dfe36e2a4b2de94eaefcf98f9c5bf31c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiNano
  - ONT platform
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Read-subsampling mixture simulation for methylation-proportion sensitivity analysis

## Summary

Generates in-silico partial-methylation read pools by subsampling m6A-modified and unmodified direct RNA sequencing reads at defined proportions, then scores sites with the trained EpiNano combined-features SVM to quantify how detection performance (ROC AUC) depends on methylation stoichiometry.

## When to use

Use when you need to characterize how site-level m6A detection performance degrades as the fraction of modified reads per site drops — e.g., benchmarking EpiNano-SVM on heterogeneous or partial-methylation samples.

## When NOT to use

- You need per-read (single-molecule) modification calls — the algorithm explicitly aggregates information from all reads mapping to a site and cannot resolve individual molecules
- You are trying to distinguish modification types (e.g., m1A vs m6A) — the trained m6A models do not discriminate between different modification species
- The sample is fully modified or fully unmodified with no stoichiometry variation — a simple pairwise (WT vs KO) EpiNano-Error analysis is more appropriate than mixture simulation

## Inputs

- Trained EpiNano combined-features linear SVM model (base quality, mismatch frequency, deletion frequency features)
- Base-called per-read error-rate feature tables (mismatch/deletion frequency) for synthetic m6A-modified and unmodified direct RNA sequencing read pools
- Per-read base quality-score feature tables for both read pools
- List of candidate m6A motif sites for aggregation

## Outputs

- Per-site SVM scores at each methylation proportion (0–100%)
- ROC curves and AUC values for modified-site detection per mixture level
- Methylation-proportion vs AUC table (e.g., mixture_auc_results.csv)

## How to apply

Load the trained combined-features linear SVM (base quality, mismatch frequency, deletion frequency) plus the base-called per-read error-rate and per-base quality-score feature tables for the synthetic m6A-modified and unmodified read pools. Create mixtures by random subsampling of modified and unmodified reads at methylation proportions of 0, 5, 10, 25, 50, 75, 90, 95, and 100%, using a fixed seed per ratio for reproducibility. Because EpiNano does not score individual molecules, aggregate read-level features into per-site summaries for every candidate m6A motif site, then score each site with the SVM at each mixture level. Compute ROC curves and AUC values for modified-site detection at each proportion, tabulate AUC versus methylation proportion, and verify the reported dependence including AUC = 0.72 at the 25% mixture. Write the proportion-vs-AUC table (e.g., mixture_auc_results.csv) as the final deliverable.

## Related tools

- **EpiNano** (Feature extraction (mismatch/deletion frequency, per-base quality) from base-called reads and SVM scoring of aggregated per-site features across mixture levels) — github.com/enovoa/EpiNano
- **ONT platform** (Source of native direct RNA sequencing reads (modified and unmodified pools) that are subsampled to construct the mixtures)

## Evaluation signals

- Monotonic relationship between methylation proportion and AUC across the 0–100% titration series
- Reproduces the reported reference point: AUC ≈ 0.72 at the 25% methylation mixture
- AUC near chance level at 0% methylation (all-unmodified control)
- Subsampling is reproducible under a fixed seed per ratio — re-running yields identical mixture composition
- Feature tables are aggregated per site (not per read) before SVM scoring, matching the algorithm's site-level design

## Limitations

- EpiNano has no per-read resolution; predictions use information pooled from all reads mapping to a site, so single-read modification detection is not achievable
- The algorithm cannot distinguish between different RNA modification types (e.g., m1A vs m6A)
- Feature contributions vary across k-mers, so k-mer-specific training is needed to reduce false positives; general models may underperform at specific sequence contexts
- Only ~32% of known modification sites are recovered overall, so low-stoichiometry sites may be missed entirely
- SVM predictions are base-caller dependent (e.g., Albacore 2.1.7 / Guppy 3.1.5 models); mixing base-caller versions invalidates pretrained models

## Evidence

- [discussion] Site-level aggregation rationale: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] Stoichiometry dependence (README): [evidence span withheld — anchor status: external_doc]
- [abstract] SVM training on synthetic modified/unmodified sequences: "our algorithm, trained with m 6 A-modified and unmodified synthetic sequences, can predict m 6 A RNA modifications with ~90% accuracy"
- [readme] Per-read resolution limitation (README): [evidence span withheld — anchor status: external_doc]

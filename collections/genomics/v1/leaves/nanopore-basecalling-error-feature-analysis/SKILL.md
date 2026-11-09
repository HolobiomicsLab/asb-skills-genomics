---
name: nanopore-basecalling-error-feature-analysis
description: Use when quantifying per-site m6A stoichiometry from ONT direct RNA reads
  that have wt, fully modified, and unmodified (knockout-derived) counterparts, using
  base-called error features rather than raw current intensity.
when_to_use_negative:
- When single-read (per-molecule) modification detection is required — the algorithm
  aggregates information from all reads mapping to a site and cannot call modifications
  per read
- When you need to distinguish between modification types (e.g., m1A vs m6A) — the
  base-calling error signature does not discriminate modification identity
- When no matched unmodified control (e.g., ime4Δ/KO or demethylase-treated sample)
  exists — direct RNA base-calling is error-rich in unmodified sequences and error
  patterns alone are noisy
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EpiNano
  role: Feature extractor (mismatch, insertion, deletion frequencies and per-base
    quality per base/k-mer) and linear-model framework used to map wt mismatch frequencies
    onto modified/unmodified reference endpoints for stoichiometry estimation
  repo: github.com/enovoa/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform whose base-called error features supply the
    mismatch frequencies used for m6A detection and stoichiometry estimation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:9f5ca7e4cf07c5f719371535b0d3fa833171a676356cd158d60882112619d860
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nanopore-basecalling-error-feature-analysis

## Summary

Estimate per-site m6A modification stoichiometry from Oxford Nanopore direct RNA sequencing by regressing wild-type base-calling mismatch frequencies (extracted with EpiNano) against 100%-modified in vitro and 0%-modified ime4Δ reference endpoints, as done for yeast (median stoichiometry 12–30%).

## When to use

Use when quantifying per-site m6A stoichiometry from ONT direct RNA reads that have wt, fully modified, and unmodified (knockout-derived) counterparts, using base-called error features rather than raw current intensity.

## When NOT to use

- When single-read (per-molecule) modification detection is required — the algorithm aggregates information from all reads mapping to a site and cannot call modifications per read
- When you need to distinguish between modification types (e.g., m1A vs m6A) — the base-calling error signature does not discriminate modification identity
- When no matched unmodified control (e.g., ime4Δ/KO or demethylase-treated sample) exists — direct RNA base-calling is error-rich in unmodified sequences and error patterns alone are noisy

## Inputs

- Base-called FASTQ/FAST5 files from yeast wt direct RNA sequencing (ONT)
- EpiNano-extracted mismatch-frequency feature tables (per-base and per-k-mer) for wt reads at known m6A sites
- Mismatch-frequency feature tables from a 100%-modified in vitro RNA sample
- Mismatch-frequency feature tables from an unmodified 0% methylation (ime4Δ-derived) in vitro reference
- List of known m6A site positions shared across all three samples

## Outputs

- Per-site m6A stoichiometry table clipped to 0–100% (stoichiometry_estimates.csv)
- Summary statistics including the median wt m6A stoichiometry across known sites

## How to apply

First run EpiNano feature extraction on base-called ONT reads to obtain per-base/per-k-mer mismatch-frequency tables for three samples: the wild-type direct RNA dataset, a 100%-modified in vitro RNA sample, and an unmodified (ime4Δ-derived, 0% methylation) in vitro reference. Restrict all three tables to the same set of known m6A sites so the regression is computed on matched positions. For each site, fit a linear regression in the EpiNano linear-model framework that maps the wt mismatch frequency onto the two reference endpoints (100% modified and 0% modified); the fitted coefficients give the estimated modification fraction, which is clipped to the 0–100% range. Aggregate per-site stoichiometry values across all recovered known m6A sites and compute the median, then write the per-site table and summary statistics to stoichiometry_estimates.csv. The rationale is that m6A appears in base-called reads as systematic, reproducible 'errors' (altered mismatch, insertion, deletion frequencies and per-base qualities) that are typically absent in ime4-knockout strains.

## Related tools

- **EpiNano** (Feature extractor (mismatch, insertion, deletion frequencies and per-base quality per base/k-mer) and linear-model framework used to map wt mismatch frequencies onto modified/unmodified reference endpoints for stoichiometry estimation) — github.com/enovoa/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform whose base-called error features supply the mismatch frequencies used for m6A detection and stoichiometry estimation)

## Evaluation signals

- Median wt yeast m6A stoichiometry estimate falls within the reported 12–30% range
- Every per-site stoichiometry estimate lies between 0% and 100%
- wt, 100%-modified, and 0%-modified (ime4Δ) feature tables are restricted to exactly the same known m6A sites before regression
- Only sites recovered by the error signature are aggregated (the method has an overall recovery of 32% of known sites, so the output covers a subset, not all known sites)
- Mismatch frequencies come from EpiNano per-k-mer feature tables, consistent with the error-based (not SVM/current-intensity) strategy

## Limitations

- The algorithm does not predict modifications in individual RNA molecules; it employs information from all reads mapping to a specific site (no per-read resolution)
- The algorithm does not distinguish between different types of RNA modifications (e.g., m1A vs m6A)
- Only ~32% of known modification sites are recovered; contribution of each feature varies across k-mers, so k-mer-specific handling affects accuracy and false positives
- Performance is dependent on modification stoichiometry, and direct RNA base-calling produces substantial errors even in unmodified sequences, so a matched control (KO/KD or demethylase-treated) is highly recommended

## Evidence

- [abstract] using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [discussion] These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site.: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [discussion] our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] *EpiNano* will extract a set of 'features' from direct RNA sequencing reads, which will be in turn used to predict whether the 'error' is caused by the presence of an RNA modification or not.: [evidence span withheld — anchor status: external_doc]
- [readme] direct RNA sequencing base-calling produces a significant amount of 'errors' in unmodified sequences. Therefore, to obtain higher confidence m6A-modified sites, we recommend to sequence both modified and unmodified datasets: [evidence span withheld — anchor status: external_doc]

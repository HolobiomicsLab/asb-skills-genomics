---
name: mismatch-frequency-feature-extraction
description: Use when analyzing ONT direct RNA sequencing FASTQ/BAM data to detect
  m6A (or other RNA modifications such as pseudouridine) from base-called 'error'
  signatures, or to build feature tables for stoichiometry estimation.
when_to_use_negative:
- When per-read, single-molecule modification resolution is required — EpiNano aggregates
  information from all reads mapping to a site and does not predict modifications
  in individual RNA molecules
- When a specific modification type must be distinguished (e.g. m1A vs m6A) — the
  algorithm does not distinguish between different modification types
- When data was base-called with a base-caller version for which no pre-trained SVM
  model exists (e.g. non-Guppy 3.1.5 data with EpiNano-SVM); in that case use EpiNano-Error
  pairwise mode or retrain
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: EpiNano
  role: Extracts per-base and per-k-mer base-calling error features (mismatch, insertion,
    deletion, quality) from direct RNA reads and uses them to detect m6A sites and
    estimate modification stoichiometry
  repo: https://github.com/enovoa/EpiNano
- name: EpiNano 1.2 (novoalab)
  role: Current EpiNano version providing EpiNano-Error and EpiNano-SVM modes, delta-features,
    sumErr, and Guppy 3.1.5 pre-trained m6A models plus EpiNano_Plot visualization
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT) direct RNA sequencing
  role: Sequencing platform whose base-called reads supply the error and quality signals
    that EpiNano converts into modification-detecting features
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:a8e86e5fcc6e37aee3a246a66996ed0c0b14594e0b875f9146864e9ad7cba3e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/enovoa/EpiNano
  tools:
  - EpiNano
  - EpiNano 1.2 (novoalab)
  - Oxford Nanopore Technologies (ONT) direct RNA sequencing
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# mismatch-frequency-feature-extraction

## Summary

Extract per-site (per-base and per-k-mer) base-calling error features — mismatch frequency, deletion frequency, insertion frequency, and per-base quality — from ONT direct RNA sequencing reads with EpiNano, as the basis for detecting and quantifying m6A RNA modifications.

## When to use

Use when analyzing ONT direct RNA sequencing FASTQ/BAM data to detect m6A (or other RNA modifications such as pseudouridine) from base-called 'error' signatures, or to build feature tables for stoichiometry estimation.

## When NOT to use

- When per-read, single-molecule modification resolution is required — EpiNano aggregates information from all reads mapping to a site and does not predict modifications in individual RNA molecules
- When a specific modification type must be distinguished (e.g. m1A vs m6A) — the algorithm does not distinguish between different modification types
- When data was base-called with a base-caller version for which no pre-trained SVM model exists (e.g. non-Guppy 3.1.5 data with EpiNano-SVM); in that case use EpiNano-Error pairwise mode or retrain

## Inputs

- Base-called ONT direct RNA sequencing reads (FASTQ, from Guppy 3.1.5 or Albacore 2.1.7 depending on the EpiNano version)
- Albacore FAST5 files (required only by EpiNano 1.0 for current intensity extraction)
- Known m6A site annotations (e.g. from yeast wt transcriptome)
- Matched control sample (e.g. ime4Δ knockout or demethylase-treated unmodified reference, 0% methylation)
- Fully modified in vitro reference sample (100% m6A methylation)

## Outputs

- Per-base and per-k-mer feature tables (mismatch frequency, deletion frequency, insertion frequency, quality, sumErr)
- Per-site predicted modified-site calls with scores
- Per-site m6A stoichiometry estimates (e.g. written to stoichiometry_estimates.csv)
- EpiNano_Plot visualizations of RNA modification predictions (EpiNano 1.2)

## How to apply

Run EpiNano's feature extraction on base-called direct RNA reads (e.g. from Guppy or Albacore) to produce per-site tables of current intensity/duration, read quality, per-base quality, and mismatch/insertion/deletion frequencies, organized in per-base and per-k-mer formats. Restrict the resulting feature tables to candidate or known m6A sites and compute per-site mismatch frequencies for each sample (e.g. wild-type yeast, in vitro 100% modified, and ime4Δ-derived unmodified in vitro reference). For modification calling, compare error patterns between matched modified/unmodified samples (EpiNano-Error pairwise mode) or feed features into pre-trained SVM models (EpiNano-SVM, base-caller version dependent). For stoichiometry, fit a linear regression relating wild-type mismatch frequencies at known m6A sites to the 100%-modified and 0%-modified reference endpoints, converting fitted coefficients into per-site m6A modification fractions clipped to the 0-100% range. Because feature contributions vary across k-mers, analysis should be k-mer-specific; mismatch frequencies differ systematically at modified sites and are largely absent in the unmodified ime4Δ control.

## Related tools

- **EpiNano** (Extracts per-base and per-k-mer base-calling error features (mismatch, insertion, deletion, quality) from direct RNA reads and uses them to detect m6A sites and estimate modification stoichiometry) — https://github.com/enovoa/EpiNano
- **EpiNano 1.2 (novoalab)** (Current EpiNano version providing EpiNano-Error and EpiNano-SVM modes, delta-features, sumErr, and Guppy 3.1.5 pre-trained m6A models plus EpiNano_Plot visualization) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT) direct RNA sequencing** (Sequencing platform whose base-called reads supply the error and quality signals that EpiNano converts into modification-detecting features)

## Examples

```
python Scripts/epinano.py -f WT.bam -t 8 && python Scripts/epinano.py -f KO.bam -t 8; Rscript Scripts/plot_epinano_results.r
```

## Evaluation signals

- Median estimated m6A stoichiometry in wild-type yeast falls within the reported 12–30% range and every per-site estimate lies between 0% and 100%
- Systematic mismatch/deletion error signals are detected at m6A sites in vitro with ~90% accuracy and in vivo in yeast with 87% accuracy
- Feature 'errors' are absent (or strongly reduced) at matched positions in the unmodified ime4Δ yeast knockout dataset, confirming modification-specific signal
- All per-site feature values are properly formatted (per-base and per-k-mer) and restricted consistently to the same set of known m6A sites across wt, 100%-modified, and 0%-modified samples
- Approximately 32% of known modification sites are recovered overall; recovery rate should be in this ballpark, not near 0% or 100%

## Limitations

- The current algorithm does not predict RNA modifications in individual RNA molecules, but employs information from all reads mapping to a specific site (no per-read resolution)
- The algorithm does not distinguish between different types of RNA modifications (e.g. m1A vs m6A)
- The contribution of each feature varies across different k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives
- Direct RNA sequencing base-calling produces a significant amount of 'errors' in unmodified sequences, so a matched control (KO/KD or demethylase-treated) is highly recommended
- Only ~32% of known modification sites are recovered, and SVM predictions are only accurate if data was base-called with the matching base-caller version (e.g. Guppy 3.1.5 or Albacore 2.1.7)

## Evidence

- [readme] Feature extraction produces per-base and per-k-mer error and quality metrics: [evidence span withheld — anchor status: not_found]
- [abstract] m6A detection manifests as systematic errors in direct RNA sequencing: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [intro] EpiNano algorithm purpose and accuracy: "propose a novel algorithm, EpiNano (github.com/enovoa/EpiNano), which can be used to identify m6A RNA modifications from RNA reads with an overall accuracy of ~90%"
- [discussion] Errors detected as altered per-base qualities and frequencies at the modified site: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [discussion] Algorithm aggregates per site rather than per read: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [abstract] Error features absent in ime4 knockout control: [evidence span withheld — anchor status: not_found]

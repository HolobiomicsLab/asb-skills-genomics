---
name: linear-regression-stoichiometry-estimation
description: Use when ONT direct RNA sequencing feature tables (mismatch/insertion/deletion
  frequencies, per-base quality) are available for a wild-type sample plus matched
  fully-modified and unmodified references, and per-site modification fraction estimates
  are needed.
when_to_use_negative:
- Only a single sample is available with no matched modified/unmodified references
  — EpiNano error-based approaches are recommended in paired mode (WT vs KO/KD)
- Single-read (per-molecule) modification calling is required — EpiNano aggregates
  information across all reads at a site and lacks per-read resolution
- The modification type is unknown — EpiNano does not distinguish between different
  modification types (e.g., m1A vs m6A)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: EpiNano
  role: Extracts base-calling error features (mismatch, insertion, deletion frequencies,
    per-base quality) from ONT direct RNA reads and provides the linear-model framework
    for relating wt mismatch frequencies to modified/unmodified reference endpoints
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the base-called reads from which
    modification-associated error features are derived
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:68b8860b3da6999951a1310456ad8d50879e35b0b0053a9f10e225f415f1486f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# linear-regression-stoichiometry-estimation

## Summary

Estimates per-site m6A modification stoichiometry by linearly regressing wild-type mismatch frequencies at known m6A sites onto two reference endpoints: a 100%-modified in vitro sample and a 0%-modified (ime4Δ-derived) unmodified reference, using EpiNano-extracted base-calling error features from ONT direct RNA sequencing.

## When to use

Use when ONT direct RNA sequencing feature tables (mismatch/insertion/deletion frequencies, per-base quality) are available for a wild-type sample plus matched fully-modified and unmodified references, and per-site modification fraction estimates are needed.

## When NOT to use

- Only a single sample is available with no matched modified/unmodified references — EpiNano error-based approaches are recommended in paired mode (WT vs KO/KD)
- Single-read (per-molecule) modification calling is required — EpiNano aggregates information across all reads at a site and lacks per-read resolution
- The modification type is unknown — EpiNano does not distinguish between different modification types (e.g., m1A vs m6A)

## Inputs

- Mismatch-frequency feature tables (EpiNano per-base/per-kmer features) for known m6A sites from yeast wt ONT direct RNA sequencing reads
- Mismatch-frequency feature tables from a 100%-modified in vitro RNA sample
- Mismatch-frequency feature tables from an unmodified (0% methylation, ime4Δ-derived) in vitro reference
- List of known m6A site positions shared across all three samples

## Outputs

- Per-site m6A stoichiometry table (stoichiometry_estimates.csv) with clipped 0–100% estimates
- Summary statistics across sites (e.g. median stoichiometry)

## How to apply

Use EpiNano feature extraction on base-called ONT direct RNA reads to compute per-kmer/per-base mismatch frequencies, and restrict all tables to the same set of known m6A sites across three samples: wild-type yeast, a 100%-modified in vitro RNA sample, and an unmodified (ime4Δ-derived) 0%-methylation in vitro reference. For each site, fit a linear regression (EpiNano linear-model framework) mapping the wt mismatch frequency onto the two reference endpoints (100% modified and 0% modified); the fitted coefficients yield an estimated modification fraction. Clip per-site estimates to the 0–100% range, then aggregate across all recovered known m6A sites and compute summary statistics (e.g. median). The regression rationale is that base-calling error rates scale approximately linearly with the fraction of modified molecules at a site, since EpiNano aggregates features over all reads mapping to a site rather than calling single molecules.

## Related tools

- **EpiNano** (Extracts base-calling error features (mismatch, insertion, deletion frequencies, per-base quality) from ONT direct RNA reads and provides the linear-model framework for relating wt mismatch frequencies to modified/unmodified reference endpoints) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the base-called reads from which modification-associated error features are derived)

## Examples

```
python regression_stoichiometry.py --wt wt_mismatch.csv --modified_100 in_vitro_modified.csv --unmodified_0 ime4delta.csv --sites known_m6A_sites.bed --output stoichiometry_estimates.csv
```

## Evaluation signals

- Median wt yeast m6A stoichiometry estimate falls within the reported 12–30% range
- Every per-site stoichiometry estimate lies between 0% and 100% after clipping
- Per-site feature tables are restricted to the same known m6A sites across wt, 100%-modified, and 0%-modified samples before regression
- Only known m6A sites recovered in the wt data (article reports ~32% recovery of known sites) contribute estimates

## Limitations

- EpiNano does not predict modifications in individual RNA molecules; it aggregates information from all reads mapping to a site, so stoichiometry is a site-level aggregate
- Only ~32% of known modification sites are recovered, limiting the sites available for stoichiometry estimation
- Direct RNA sequencing is error-rich even in unmodified sequences, so matched unmodified controls are highly recommended; performance depends on the stoichiometry of the modification
- The algorithm does not distinguish between different types of RNA modifications (e.g., m1A vs m6A)
- SVM/linear model performance is base-caller dependent (e.g., Albacore 2.1.7 or Guppy 3.1.5 pretrained models)

## Evidence

- [discussion] Our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] EpiNano relies on base-calling errors; direct RNA base-calling produces many errors in unmodified sequences, so modified/unmodified datasets should be sequenced: [evidence span withheld — anchor status: external_doc]
- [abstract] m6A modifications detected via direct RNA sequencing as systematic errors and decreased base-calling qualities: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [discussion] Method accuracy: 90% in vitro, 87% in vivo, 32% recovery of known sites: "the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites"
- [discussion] These errors take the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."

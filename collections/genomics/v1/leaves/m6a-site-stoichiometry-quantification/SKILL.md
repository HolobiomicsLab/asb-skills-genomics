---
name: m6a-site-stoichiometry-quantification
description: Use when ONT direct RNA sequencing base-called data contains known m6A
  sites and you need per-site fractional modification levels (0–100%), not just binary
  modified/unmodified calls.
when_to_use_negative:
- You need single-read (per-molecule) modification calls — EpiNano aggregates information
  from all reads mapping to a site and has no per-read resolution
- You only have one sample with no matched modified/unmodified references — the regression
  requires both 100%-modified and 0%-modified endpoints (a matched control such as
  KO/KD is highly recommended)
- Your goal is binary modified-site detection rather than fractional quantification
  — EpiNano-SVM with pre-trained m6A models is the more direct choice
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0621
tools:
- name: EpiNano
  role: 'Feature extractor and linear-model framework: extracts per-kmer base-calling
    error features (mismatch, deletion, insertion, quality) from ONT base-called reads
    and fits the regression mapping wt mismatch frequency onto modified/unmodified
    reference endpoints'
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the base-called reads from which
    error features are extracted for stoichiometry estimation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:025d908787993f09e549416635cc59a228a6563d3132cfb3e827d77fe36b9ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# m6a-site-stoichiometry-quantification

## Summary

Estimates per-site m6A modification stoichiometry in direct RNA sequencing data by fitting a linear regression that maps wild-type base-calling mismatch frequencies at known m6A sites onto fully modified (100%) and unmodified (0%, e.g. ime4Δ) reference endpoints, as demonstrated for yeast where the median wt stoichiometry was 12–30%.

## When to use

Use when ONT direct RNA sequencing base-called data contains known m6A sites and you need per-site fractional modification levels (0–100%), not just binary modified/unmodified calls.

## When NOT to use

- You need single-read (per-molecule) modification calls — EpiNano aggregates information from all reads mapping to a site and has no per-read resolution
- You only have one sample with no matched modified/unmodified references — the regression requires both 100%-modified and 0%-modified endpoints (a matched control such as KO/KD is highly recommended)
- Your goal is binary modified-site detection rather than fractional quantification — EpiNano-SVM with pre-trained m6A models is the more direct choice

## Inputs

- Mismatch-frequency feature tables (EpiNano per-base/per-kmer format) computed from ONT base-called wt direct RNA reads at known m6A sites
- Mismatch-frequency feature table from a 100%-modified in vitro RNA reference
- Mismatch-frequency feature table from an unmodified (0% methylation, ime4Δ-derived) in vitro reference
- A curated list of known m6A sites common to all three samples

## Outputs

- Per-site m6A stoichiometry table (clipped to 0–100%) written as stoichiometry_estimates.csv
- Summary statistics including the median stoichiometry across recovered sites

## How to apply

Extract EpiNano base-calling 'error' features (mismatch frequency, plus deletion/insertion frequency and per-base quality) at known m6A sites from three matched samples: wild-type direct RNA reads, a fully modified reference (e.g. in vitro transcribed RNA with 100% m6A), and an unmodified reference (0% methylation, e.g. ime4Δ-derived in vitro RNA). Restrict all feature tables to the identical set of known m6A sites so the regression is fitted per-site on comparable k-mers. For each site, regress the wt mismatch frequency onto the two reference endpoints (100% modified and 0% modified) using the EpiNano linear-model framework; the fitted coefficients yield an estimated modification fraction. Clip per-site estimates to the 0–100% range, then aggregate across sites and report the median (the yeast wt estimate was 12–30%). Rationale: m6A appears in direct RNA data as systematic base-calling 'errors' whose magnitude scales with modification fraction, and the article stresses that algorithm performance depends on stoichiometry, making quantification essential for interpreting calls.

## Related tools

- **EpiNano** (Feature extractor and linear-model framework: extracts per-kmer base-calling error features (mismatch, deletion, insertion, quality) from ONT base-called reads and fits the regression mapping wt mismatch frequency onto modified/unmodified reference endpoints) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the base-called reads from which error features are extracted for stoichiometry estimation)

## Evaluation signals

- Median wt yeast stoichiometry estimate falls within the reported 12–30% range
- Every per-site stoichiometry estimate lies between 0% and 100% after clipping
- Unmodified (ime4Δ) reference sites show systematically lower mismatch frequencies than fully modified references at the same known m6A sites
- All feature tables resolve to the same set of known m6A sites before regression (per-site pairing is exact, no missing endpoints)
- Per-site stoichiometry table matches the schema of stoichiometry_estimates.csv (site ID, estimated fraction, summary stats)

## Limitations

- The EpiNano algorithm aggregates information from all reads mapping to a site and does not predict modifications in individual RNA molecules
- Performance of the algorithm is dependent on the stoichiometry of the modification, so low-stoichiometry sites may fall below reliable detection thresholds
- Only ~32% of known modification sites are recovered by the method, so stoichiometry estimates cover a subset of sites
- Error-based features do not distinguish between different types of RNA modifications (e.g. m1A vs m6A), so quantified signal could include non-m6A error sources

## Evidence

- [results] Using linear regression relating wt mismatch frequencies to the 100%-modified and 0%-modified (ime4Δ) references, the median m6A stoichiometry in wt yeast strains was estimated at 12–30%.: [evidence span withheld — anchor status: not_found]
- [abstract] m6A appears as systematic base-calling errors whose detection underpins stoichiometry estimation: "N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [discussion] The algorithm aggregates all reads per site rather than calling individual molecules: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] README: algorithm performance depends on modification stoichiometry: [evidence span withheld — anchor status: external_doc]
- [discussion] Only a subset of known sites is recovered by the method: "with an overall recovery of 32% of known sites"

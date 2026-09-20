---
name: multiple-testing-correction-thresholding
description: Use when calling modified transcriptomic sites (m6A/m5C k-mers) from
  nanopore direct RNA predictions across a large number of sites, and per-site significance
  or probability must be corrected at alpha = 0.05, 0.01, or 0.001.
when_to_use_negative:
- Only a handful of pre-specified candidate sites are being tested, so the multiple-testing
  burden is negligible and strict transcriptome-wide cut-offs would unnecessarily
  reduce sensitivity
- The goal is single-read (per-molecule) modification prediction rather than site-level
  significance testing across many transcriptomic sites
- Input is raw FAST5/nanopolish eventalign signal that has not yet been scored by
  a modification caller — thresholding applies downstream of prediction
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CHEUI
  role: Source of Model 1 (per-read) and Model 2 (site/stoichiometry) probabilities
    thresholded at > 0.99 for positivity and > 0.9999 / > 0.7 / < 0.3 for stoichiometry
    estimation
  repo: https://github.com/comprna/R2Dtool
- name: Xpore
  role: Compared tool; mod_rate_WT-rep1 values serve as per-site stoichiometry estimates
    evaluated under corrected alpha thresholds
- name: Nanocompore
  role: Compared tool; WT modified-cluster counts over total WT reads used as stoichiometry
    estimates under corrected alpha thresholds
- name: Epinano
  role: Compared tool run in sumErr mode on WT/KO pairs at corrected alpha levels
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:ba68b949c4c04cbdaec45f32d3ed0aed031d397f3bd44573850b198e8df41625
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/R2Dtool
  tools:
  - CHEUI
  - Epinano
  - NanoRMS
  - Nanocompore
  - Tombo
  - Xpore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# multiple-testing-correction-thresholding

## Summary

Applies multiple-testing correction and probability/score thresholds to control the expected FDR when calling RNA modifications (m6A, m5C) from nanopore direct RNA data across thousands of transcriptomic sites, as used to benchmark CHEUI against Xpore, Nanocompore, Epinano, NanoRMS and Tombo.

## When to use

Use when calling modified transcriptomic sites (m6A/m5C k-mers) from nanopore direct RNA predictions across a large number of sites, and per-site significance or probability must be corrected at alpha = 0.05, 0.01, or 0.001.

## When NOT to use

- Only a handful of pre-specified candidate sites are being tested, so the multiple-testing burden is negligible and strict transcriptome-wide cut-offs would unnecessarily reduce sensitivity
- The goal is single-read (per-molecule) modification prediction rather than site-level significance testing across many transcriptomic sites
- Input is raw FAST5/nanopolish eventalign signal that has not yet been scored by a modification caller — thresholding applies downstream of prediction

## Inputs

- Caller output tables per site (CHEUI Model 1 per-read probabilities and Model 2 site probabilities; Xpore mod_rate values; Nanocompore WT modified-cluster counts; Epinano sumErr results)
- Reference k-mer sets (k = 9) with one centered m6A or m5C and no additional modifiable bases
- Alpha levels for testing (0.05, 0.01, 0.001) with multiple-testing correction
- Unmodified-only read sets for false-positive rate estimation

## Outputs

- Per-site modified/unmodified calls with corrected significance at each alpha level
- Per-tool true-positive and false-positive counts
- Per-site stoichiometry estimates (e.g., CHEUI Model 2 probability > 0.9999 sites with per-read Model 1 probability > 0.7 / < 0.3 calls) for Pearson correlation against expected stoichiometry
- benchmark_results.csv with per-tool TP counts, FP counts, and Pearson correlation coefficients

## How to apply

Run each modification caller on its designated inputs (e.g., CHEUI-solo on the WT sample alone with Model 2 positivity at probability > 0.99; Epinano, Nanocompore, and Xpore on WT/KO pairs) and apply caller outputs at alpha = 0.05, 0.01, and 0.001 with multiple-testing correction. For CHEUI, define site positivity via CHEUI-solo Model 2 probability > 0.99, and estimate per-site stoichiometry using Model 2 probability > 0.9999 sites combined with per-read Model 1 probability > 0.7 (modified) / < 0.3 (unmodified) calls. The rationale is that, given the large number of transcriptomic sites tested, very strict cut-offs are required to maintain a low expected false discovery rate, which trades sensitivity for precision. Evaluate true positives only on k-mers (k = 9) containing one centered modified base (m6A or m5C) with no additional modifiable As/Cs, and estimate false-positive rates by counting sites called modified when comparing unmodified-only read sets.

## Related tools

- **CHEUI** (Source of Model 1 (per-read) and Model 2 (site/stoichiometry) probabilities thresholded at > 0.99 for positivity and > 0.9999 / > 0.7 / < 0.3 for stoichiometry estimation) — https://github.com/comprna/R2Dtool
- **Xpore** (Compared tool; mod_rate_WT-rep1 values serve as per-site stoichiometry estimates evaluated under corrected alpha thresholds)
- **Nanocompore** (Compared tool; WT modified-cluster counts over total WT reads used as stoichiometry estimates under corrected alpha thresholds)
- **Epinano** (Compared tool run in sumErr mode on WT/KO pairs at corrected alpha levels)

## Evaluation signals

- CHEUI stoichiometry Pearson correlation against expected stoichiometry reproduces r = 0.839 for both m6A and m5C on the independent IVT benchmark
- Per-tool TP/FP counts match the trends shown in Fig. 2f–2h at each corrected alpha (0.05, 0.01, 0.001)
- False-positive rates are estimated against unmodified-only read sets (single unmodified set for CHEUI) and remain low
- Positive sites are restricted to k = 9-mers with exactly one centered m6A (no additional As) or one centered m5C (no additional Cs) per stoichiometry level
- CHEUI site positivity uses Model 2 probability > 0.99; stoichiometry computation uses Model 2 probability > 0.9999 sites with per-read Model 1 > 0.7 / < 0.3 calls

## Limitations

- Very strict cut-offs used on cell transcriptome sites to maintain a low expected FDR reduce CHEUI sensitivity; many sites found by orthogonal methods had high CHEUI probabilities below the set thresholds
- Relaxing thresholds would recover more sites found by orthogonal techniques but at the cost of potential false positives
- This limitation may stem from nanopore signal variability, where differences between modified and non-modified reads are often comparable to differences among non-modified reads

## Evidence

- [other] alpha thresholds in benchmark: [evidence span withheld — anchor status: not_found]
- [discussion] strict cutoffs rationale: "given the large number of sites tested, we used very strict cut-offs to maintain a low expected false discovery rate (FDR)"
- [discussion] sensitivity cost of strict cutoffs: "While these strict cutoffs resulted in a reduction of CHEUI sensitivity"
- [other] stoichiometry probability gates: [evidence span withheld — anchor status: not_found]
- [readme] two model roles in README: [evidence span withheld — anchor status: external_doc]

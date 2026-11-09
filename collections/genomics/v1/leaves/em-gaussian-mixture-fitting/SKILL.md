---
name: em-gaussian-mixture-fitting
description: Use when you have per-site SMsn scores for a methylated motif (e.g. 5'-CTGCAG
  in E. coli C227) and need to estimate the methylated fraction of sites — especially
  when no trustworthy score cutoff exists.
when_to_use_negative:
- You already have a validated subjective score threshold and only need to classify
  sites — use the SMsn protocol's per-site calling directly rather than mixture estimation.
- Your library is long (10Kb+) and you care about molecule-wide methylation status
  — the SMp motif-pooled protocol, not per-site EM fitting, is designed for that regime.
- The modification's kinetic signature extends into surrounding bases (e.g. some non-adenine
  modifications) — the article states detection of such types requires more sophisticated
  algorithms than this framework provides.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: PyMix
  role: Fits the two-component Gaussian mixture to SMsn score distributions via its
    EM algorithm, separating methylated and non-methylated site populations without
    subjective thresholds
- name: SMALR
  role: Produces the SMsn per-site methylation scores (native vs matching WGA kinetics
    comparison) that serve as the input distribution for the EM mixture fit
  repo: https://github.com/fanglab/smalr
- name: Python
  role: Hosts the simulation of in silico native/WGA mixtures, the PyMix EM fitting
    loop, coverage downsampling, and CSV output
- name: R
  role: Used as a SMALR/analysis dependency (>= 3.0) in the pipeline producing the
    SMsn scores
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:b47cc3f0ebd70941dca57e8759b1a5eeb53c6548baec72a0d256824cb0b8206f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - PyMix
  - Python
  - R
  - SMALR
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# em-gaussian-mixture-fitting

## Summary

Fits a two-component Gaussian mixture (via EM, e.g. PyMix) to per-site SMsn score distributions (native minus WGA SMRT kinetics scores) to estimate the fraction of methylated motif sites without applying subjective score thresholds, and tests how the estimate holds under downsampled genomic coverage.

## When to use

Use when you have per-site SMsn scores for a methylated motif (e.g. 5'-CTGCAG in E. coli C227) and need to estimate the methylated fraction of sites — especially when no trustworthy score cutoff exists.

## When NOT to use

- You already have a validated subjective score threshold and only need to classify sites — use the SMsn protocol's per-site calling directly rather than mixture estimation.
- Your library is long (10Kb+) and you care about molecule-wide methylation status — the SMp motif-pooled protocol, not per-site EM fitting, is designed for that regime.
- The modification's kinetic signature extends into surrounding bases (e.g. some non-adenine modifications) — the article states detection of such types requires more sophisticated algorithms than this framework provides.

## Inputs

- SMsn.out results file with per-site SMsn scores (native minus WGA) for the motif of interest
- Known or hypothesized native methylated fractions used to build in silico native/WGA mixtures (5–100%)
- Coverage levels for downsampling (e.g. progressively lower genomic sequence coverage)

## Outputs

- Per-mixture, per-coverage EM-estimated native methylated fractions (native_fraction_estimates.csv)
- Two-component Gaussian mixture parameters separating methylated and non-methylated site populations

## How to apply

Draw the per-site SMsn score values (native score minus WGA score) for the motif from the SMsn.out results file and construct in silico mixtures spanning 5–100% native fraction with ~100,000 molecules per mixture. Fit a two-component Gaussian mixture to each simulated mixture's SMsn score distribution using PyMix's EM algorithm, letting the EM assign sites to methylated vs non-methylated populations instead of imposing a subjective threshold. Downsample each mixture to progressively lower genomic coverage and re-run the EM fit at each level to characterize the coverage-versus-accuracy tradeoff of single-nucleotide SMsn detection. Compare EM-estimated native fractions against known simulated fractions, expecting a slight but consistent underestimation of the native fraction attributable to stably non-methylated sites.

## Related tools

- **PyMix** (Fits the two-component Gaussian mixture to SMsn score distributions via its EM algorithm, separating methylated and non-methylated site populations without subjective thresholds)
- **SMALR** (Produces the SMsn per-site methylation scores (native vs matching WGA kinetics comparison) that serve as the input distribution for the EM mixture fit) — https://github.com/fanglab/smalr
- **Python** (Hosts the simulation of in silico native/WGA mixtures, the PyMix EM fitting loop, coverage downsampling, and CSV output)
- **R** (Used as a SMALR/analysis dependency (>= 3.0) in the pipeline producing the SMsn scores)

## Evaluation signals

- Every EM-estimated native fraction falls in [0, 1]
- Estimated native fractions track the known simulated fractions across all mixture/coverage combinations
- Any deviation from ground truth appears as the reported slight, consistent underestimation of the native fraction (attributable to stably non-methylated sites), not random error
- Accuracy persists at low genomic sequence coverage — the coverage-vs-accuracy re-runs should show stable separation, not breakdown at low coverage
- The fit classifies methylated vs non-methylated sites without invoking any hand-tuned SMsn score threshold

## Limitations

- Accuracy degrades with few repeated observations per molecule; single-molecule detection depends on number of passes per site, with a tradeoff between library size and single-molecule, single-nucleotide accuracy
- The EM fit underestimates the true native fraction slightly but consistently, attributable to stably non-methylated sites
- Demonstrated on short (~250bp) library preps of single bacterial strains; extension to mixed bacterial populations was future work in the study

## Evidence

- [discussion] Short-library SMsn threshold-free estimation: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds"
- [intro] SMsn per-molecule per-site scoring context: "Each motif site on each sequencing molecule is assessed for methylation status. This is designed for use with short (~250bp) sequencing library preps"
- [readme] Reliability depends on single-molecule coverage: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."
- [discussion] Low-coverage robust separation enables de novo motif discovery: "the robust separation between methylated and non-methylated sites based on SMSN scores at low levels of genomic sequence coverage suggests the possibility of reference-free, de novo discovery of"

---
name: in-silico-methylome-mixture-simulation
description: Use when you need to quantify methylated vs non-methylated motif-site
  fractions from SMALR SMsn scores without applying thresholds — e.g., estimating
  native fraction in native/WGA mixtures or probing the coverage-versus-accuracy tradeoff.
when_to_use_negative:
- Input is long (10Kb+) library prep data where molecule-wide methylation status should
  be assessed with the SMp motif-pooled protocol instead of per-site SMsn scoring
- The goal is reference-free, de novo discovery of methylation motifs rather than
  estimating the methylated fraction of a known motif
- A population-level consensus methylation call suffices and single-molecule or mixture-level
  fraction estimation is not required
tools:
- name: SMALR
  role: Generates the per-site SMsn scores (native minus WGA kinetics) from aligned_reads.cmph5
    files that serve as the simulated mixture input
  repo: https://github.com/fanglab/smalr
- name: PyMix
  role: Provides the EM algorithm for fitting the two-component Gaussian mixture to
    the simulated SMsn score distribution
- name: Python
  role: Implements the in-silico mixture construction, downsampling, and comparison
    of estimated vs true native fractions
- name: R
  role: Downstream analysis/visualization of native-fraction estimation results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:1555ef693d0fa31a075d58ae2cbda9b99502f05abe398a24fb9055f9f3112478
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - PyMix
  - Python
  - R
  - SMALR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# in-silico-methylome-mixture-simulation

## Summary

Simulates in silico mixtures of native and WGA molecules from per-site SMSn scores and fits a two-component Gaussian mixture with PyMix's EM algorithm to estimate the native (methylated) motif-site fraction without subjective score thresholds, testing accuracy across genomic sequence coverages.

## When to use

Use when you need to quantify methylated vs non-methylated motif-site fractions from SMALR SMsn scores without applying thresholds — e.g., estimating native fraction in native/WGA mixtures or probing the coverage-versus-accuracy tradeoff.

## When NOT to use

- Input is long (10Kb+) library prep data where molecule-wide methylation status should be assessed with the SMp motif-pooled protocol instead of per-site SMsn scoring
- The goal is reference-free, de novo discovery of methylation motifs rather than estimating the methylated fraction of a known motif
- A population-level consensus methylation call suffices and single-molecule or mixture-level fraction estimation is not required

## Inputs

- SMsn.out results file from SMALR containing per-site SMsn score values (native score minus WGA score) for the 5'-CTGCAG motif of E. coli C227
- SMsn score distributions from native and matching WGA aligned_reads.cmph5 comparisons
- Target native fraction grid (5–100%) and target genomic sequence coverage levels for downsampling

## Outputs

- native_fraction_estimates.csv with per-mixture, per-coverage estimated and true native fractions
- Fitted two-component Gaussian mixture parameters separating methylated and non-methylated SMsn score populations

## How to apply

Draw per-site SMsn score values (native score minus WGA score) for the target motif (here 5'-CTGCAG of E. coli C227) from the SMsn.out results file produced by the SMALR pipeline, then construct in silico mixtures spanning 5–100% native fraction with 100,000 molecules per mixture. Fit a two-component Gaussian mixture to each simulated mixture's SMsn score distribution using PyMix's EM algorithm, letting the two components separate the methylated and non-methylated site populations rather than applying subjective score cutoffs. Downsample each simulated mixture to progressively lower levels of genomic sequence coverage and re-run the PyMix EM fit at each coverage level to probe the coverage-versus-accuracy tradeoff, which the paper ties to the number of repeated observations per single molecule. At every mixture/coverage combination, compare the EM-estimated native fraction against the known simulated fraction and quantify the slight, consistent underestimation of the native fraction attributable to stably non-methylated sites. Upstream SMALR filtering (e.g., --nativeCovThresh 10, --wgaCovThresh 10, --minSubreadLen 100, --minAcc 0.8) should already have been applied when generating the SMsn scores.

## Related tools

- **SMALR** (Generates the per-site SMsn scores (native minus WGA kinetics) from aligned_reads.cmph5 files that serve as the simulated mixture input) — https://github.com/fanglab/smalr
- **PyMix** (Provides the EM algorithm for fitting the two-component Gaussian mixture to the simulated SMsn score distribution)
- **Python** (Implements the in-silico mixture construction, downsampling, and comparison of estimated vs true native fractions)
- **R** (Downstream analysis/visualization of native-fraction estimation results)

## Examples

```
import numpy as np; mix = np.random.permutation(np.concatenate((native_smsn_scores, wga_smsn_scores)))[:100000]  # 5-100% native in-silico mixture, then PyMix EM two-component Gaussian fit
```

## Evaluation signals

- Every estimated native fraction must lie in [0,1] for all mixture/coverage combinations
- Estimated native fractions must track the known simulated fractions across all coverage levels, showing only the slight consistent underestimation of the native fraction reported in the paper
- The methylated and non-methylated components of the EM Gaussian mixture must be robustly separated in the SMsn score distribution without any subjective score threshold being applied
- Estimates must remain accurate after downsampling to progressively lower levels of genomic sequence coverage, reflecting the reliability-vs-passes tradeoff of SMsn scoring

## Limitations

- There is a slight but consistent underestimation of the native fraction attributable to stably non-methylated sites
- Detection of DNA modifications from SMRT reads fundamentally depends on the number of repeated observations per single molecule; for a given read length there is a tradeoff between library size and accuracy of single-molecule, single-nucleotide methylation detection
- SMsn scoring is designed for short (~250bp) sequencing library preps where SMRT reads give multiple passes over each motif site; reliability increases with more passes
- The SMsn score separation was demonstrated on single bacterial strain cultures; extension to mixed bacterial populations remained future work

## Evidence

- [discussion] The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds"
- [other] Fit a two-component Gaussian mixture to each simulated mixture's SMSn score distribution using PyMix's EM algorithm: [evidence span withheld — anchor status: not_found]
- [other] construct in silico mixtures spanning 5–100% native fraction with 100,000 molecules per mixture: [evidence span withheld — anchor status: not_found]
- [discussion] detection of DNA modifications from SMRT reads fundamentally depends on the number of repeated observations for each single molecule: "detection of DNA modifications from SMRT reads fundamentally depends on the number of repeated observations for each single molecule. For a given read length, there is a resulting tradeoff between"
- [readme] Each motif site on each sequencing molecule is assessed for methylation status.: "Each motif site on each sequencing molecule is assessed for methylation status. This is designed for use with short (~250bp) sequencing library preps"

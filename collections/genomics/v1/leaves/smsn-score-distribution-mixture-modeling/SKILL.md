---
name: smsn-score-distribution-mixture-modeling
description: Use when you have single-molecule SMsn score distributions (native vs
  WGA kinetic comparison for a motif such as 5'-CTGCAG in E. coli C227) and need to
  estimate the methylated/native site fraction threshold-free, including at low coverage.
when_to_use_negative:
- You have long (10Kb+) library preps where molecule-wide methylation status is assessed
  — use the SMp motif-pooled protocol instead of per-site SMsn score distributions
- You need per-molecule methylation calls or phased epigenetic variation rather than
  population-level estimation of the methylated motif-site fraction
- You lack a matching WGA control to compute native-minus-WGA SMsn scores, since the
  score distribution being modeled depends on native/WGA kinetics comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: SMALR
  role: Produces the SMsn per-site methylation scores (native minus WGA kinetics comparison)
    whose distribution is modeled
  repo: https://github.com/fanglab/smalr
- name: PyMix
  role: Provides the EM algorithm used to fit the two-component Gaussian mixture to
    SMsn score distributions
- name: Python 2.7
  role: Environment in which SMALR and the mixture-fitting/coverage-downsampling simulation
    are implemented
- name: R
  role: Listed dependency in the SMALR methods environment (R >= 3.0) used alongside
    the analysis pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:a100911774a3a6ffe7bb50e81359845556247701d7e14fc2c71093972569afac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - PyMix
  - Python
  - Python 2.7
  - R
  - SMALR
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# smsn-score-distribution-mixture-modeling

## Summary

Fits a two-component Gaussian mixture (via PyMix's EM algorithm) to per-site SMsn score distributions (native minus WGA kinetic scores) to estimate the fraction of methylated motif sites without subjective thresholds, and tests robustness by downsampling to low genomic coverage.

## When to use

Use when you have single-molecule SMsn score distributions (native vs WGA kinetic comparison for a motif such as 5'-CTGCAG in E. coli C227) and need to estimate the methylated/native site fraction threshold-free, including at low coverage.

## When NOT to use

- You have long (10Kb+) library preps where molecule-wide methylation status is assessed — use the SMp motif-pooled protocol instead of per-site SMsn score distributions
- You need per-molecule methylation calls or phased epigenetic variation rather than population-level estimation of the methylated motif-site fraction
- You lack a matching WGA control to compute native-minus-WGA SMsn scores, since the score distribution being modeled depends on native/WGA kinetics comparison

## Inputs

- SMsn.out results file with per-site SMsn scores (native minus WGA) for a motif (e.g., 5'-CTGCAG of E. coli C227)
- In silico native/WGA molecule mixtures (e.g., 5-100% native fraction, 100,000 molecules each)
- Downsampled coverage variants of each simulated mixture

## Outputs

- EM-estimated native (methylated) fraction per mixture and coverage level
- native_fraction_estimates.csv with per-mixture, per-coverage estimated and true native fractions
- Estimated two-component Gaussian mixture parameters separating methylated vs non-methylated site populations

## How to apply

Draw per-site SMsn score values (native score minus WGA score) from the SMsn.out results file for the motif of interest, and construct in silico mixtures spanning 5-100% native fraction with 100,000 molecules per mixture. Fit a two-component Gaussian mixture to each mixture's score distribution using PyMix's EM algorithm, letting the data separate methylated and non-methylated site populations instead of applying subjective score thresholds. Downsample each simulated mixture to progressively lower genomic coverage and re-run the EM fit at each level to probe the coverage-versus-accuracy tradeoff, which is grounded in the fact that SMsn reliability increases with more passes per molecule. At every mixture/coverage combination, compare the EM-estimated native fraction to the known simulated fraction; expect slight but consistent underestimation of the native fraction attributable to stably non-methylated sites. Write per-mixture, per-coverage estimated and true native fractions to a CSV such as native_fraction_estimates.csv.

## Related tools

- **SMALR** (Produces the SMsn per-site methylation scores (native minus WGA kinetics comparison) whose distribution is modeled) — https://github.com/fanglab/smalr
- **PyMix** (Provides the EM algorithm used to fit the two-component Gaussian mixture to SMsn score distributions)
- **Python 2.7** (Environment in which SMALR and the mixture-fitting/coverage-downsampling simulation are implemented)
- **R** (Listed dependency in the SMALR methods environment (R >= 3.0) used alongside the analysis pipeline)

## Examples

```
from pyfmit import MixtureModel, GaussianDistribution; mm = MixtureModel([GaussianDistribution(0,1), GaussianDistribution(1,1)]); mm.EM(scores, 10, 10)
```

## Evaluation signals

- Every EM-estimated native fraction lies within [0,1]
- Estimated native fractions track the known simulated fractions across all mixture (5-100%) and coverage combinations
- Only slight, consistent underestimation of the native fraction is observed, matching the paper's reported bias from stably non-methylated sites
- The fitted two-component mixture cleanly separates methylated and non-methylated site populations without imposing any hand-tuned score threshold
- Accuracy at low genomic coverage remains consistent with the high-coverage estimates, reflecting the coverage-vs-accuracy tradeoff of SMsn detection

## Limitations

- Accuracy of single-molecule, single-nucleotide detection depends on the number of repeated observations per molecule; for a given read length there is a tradeoff between library size and detection accuracy
- SMsn reliability increases with more passes (higher single-molecule coverage), so very low single-molecule coverage degrades score reliability
- A slight, consistent underestimation of the native fraction occurs, attributable to stably non-methylated sites

## Evidence

- [discussion] The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic: "The short library-based SMSN method enables accurate estimation of the fractions of methylated and non-methylated motif sites without the use of subjective thresholds, even at low levels of genomic"
- [methods] Each motif site on each sequencing molecule is assessed for methylation status.: "Each motif site on each sequencing molecule is assessed for methylation status."
- [readme] There are two protocols available for use within the pipeline, SMsn and SMp.: "There are two protocols available for use within the pipeline, SMsn and SMp."
- [readme] The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage).: "The reliability of the SMsn scores increases with more passes (i.e. higher single-molecule coverage)."

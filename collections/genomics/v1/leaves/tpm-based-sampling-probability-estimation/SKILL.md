---
name: tpm-based-sampling-probability-estimation
description: 'Use when planning sequencing depth for transcript discovery: given transcript
  abundances (TPM) from LRTS/PacBio data and a two-copy IsoSeq discovery threshold,
  estimate observation probability as a function of depth.'
when_to_use_negative:
- Input is short-read RNA-seq data rather than aligned full-length transcript (LRTS
  bam) data, since the model is built for IsoTools' LRTS saturation analysis
- The goal is alternative splicing event detection or differential splicing (beta-binomial
  differential analysis), which are separate IsoTools capabilities
- No TPM abundance estimates are available and only read counts without transcript-level
  quantification exist
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python module used for the saturation analysis workflow, transcript quantification
    (TPM), and data visualization of the fitted observation-probability curves
  repo: github.com/MatthiasLienhard/isotools
- name: Python
  role: Language used to define the negative binomial cumulative distribution function,
    fit parameters, evaluate the depth grid, and plot saturation curves
- name: pip
  role: Installs the isotools package (python3 -m pip install isotools)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:e72192a349ae7622a436acb7ecbe3d725c1190f05919a8c4d6ae35558e2c0699
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IsoTools
  - Python
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# TPM-based sampling probability estimation

## Summary

A model-based saturation analysis for Long Read Transcriptome Sequencing (LRTS) that fits a negative binomial observation-probability model — with parameters n (sequencing depth), r (minimum required coverage), and p (per-transcript sampling probability proportional to RNA concentration/TPM) — to estimate how much depth is needed to detect transcripts, applied with IsoTools' saturation analysis workflow (notebooks/04_saturation_analysis.ipynb).

## When to use

Use when planning sequencing depth for transcript discovery: given transcript abundances (TPM) from LRTS/PacBio data and a two-copy IsoSeq discovery threshold, estimate observation probability as a function of depth.

## When NOT to use

- Input is short-read RNA-seq data rather than aligned full-length transcript (LRTS bam) data, since the model is built for IsoTools' LRTS saturation analysis
- The goal is alternative splicing event detection or differential splicing (beta-binomial differential analysis), which are separate IsoTools capabilities
- No TPM abundance estimates are available and only read counts without transcript-level quantification exist

## Inputs

- Transcriptome data prepared with the IsoTools saturation analysis workflow (notebooks/04_saturation_analysis.ipynb, e.g. an isotools .pkl file such as example_isotools.pkl)
- Transcript abundances in TPM for each transcript
- Measured depth-detection relationship from the saturation analysis data

## Outputs

- Fitted negative-binomial observation-probability curve over a sequencing-depth grid
- Predicted observation probabilities for transcripts at 1 TPM and >=2 TPM at the given depths
- Saturation curve figure (saturation_curve.png)

## How to apply

Load the transcriptome prepared with the IsoTools saturation analysis workflow (notebooks/04_saturation_analysis.ipynb) and define the observation-probability model as the cumulative distribution function of a negative binomial with parameters n (depth), r (minimum required coverage), and p, where the per-transcript sampling probability is proportional to the transcript's RNA concentration (TPM). Apply the IsoSeq two-copy discovery threshold so a transcript counts as observed once at least two reads are sampled. Fit or set the model parameters against the measured depth-detection relationship from the saturation analysis data, then evaluate the fitted curve across a depth grid. Plot the fitted observation-probability curves versus sequencing depth with IsoTools/Python and save the figure (e.g. saturation_curve.png). The rationale is that transcripts with low RNA concentration (e.g. one TPM) are the bottleneck for transcript discovery, so their probability of being sampled at sufficient depth determines required sequencing depth.

## Related tools

- **IsoTools** (Python module used for the saturation analysis workflow, transcript quantification (TPM), and data visualization of the fitted observation-probability curves) — github.com/MatthiasLienhard/isotools
- **Python** (Language used to define the negative binomial cumulative distribution function, fit parameters, evaluate the depth grid, and plot saturation curves)
- **pip** (Installs the isotools package (python3 -m pip install isotools))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl')
```

## Evaluation signals

- The fitted curve predicts that at the reported hepatocyte depths, one-TPM transcripts have over 70% observation probability
- Transcripts expressed at two TPM or more are detected with at least two reads with close to 100% probability at the reported depths
- The negative binomial CDF reproduces the measured depth-detection relationship from the saturation analysis data
- Observation probability is monotone increasing in sequencing depth n for all TPM levels
- saturation_curve.png is produced showing observation-probability curves versus sequencing depth

## Limitations

- Predicted probabilities are qualitative saturation results; validation is based on the qualitative thresholds (>70% for 1 TPM, ~100% for >=2 TPM), not exact curve agreement
- The model assumes per-transcript sampling probability is proportional to RNA concentration (TPM), which may not hold if capture bias distorts abundance
- Calibration to the reported hepatocyte depths means predictions may not transfer to other platforms or samples without refitting

## Evidence

- [abstract] IsoTools Python-based LRTS framework: "We introduce IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts."
- [abstract] IsoTools applied to PacBio hepatocyte data: "we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid"
- [readme] IsoTools is a python module for LRTS analysis: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [readme] Save transcriptome as pkl for faster import: [evidence span withheld — anchor status: external_doc]

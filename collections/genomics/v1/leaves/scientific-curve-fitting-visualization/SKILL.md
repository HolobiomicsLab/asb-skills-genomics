---
name: scientific-curve-fitting-visualization
description: Use when PacBio LRTS saturation-analysis data relate observed transcript
  detection to sequencing depth and you must estimate how deep to sequence before
  transcripts at a given expression level (e.g. 1 TPM) become reliably detectable.
when_to_use_negative:
- The input is short-read RNA-seq quantification rather than long-read (PacBio IsoSeq)
  saturation data with per-transcript read sampling
- The goal is detection of differential alternative splicing (use the beta-binomial
  differential-event workflow instead of depth-planning curve fitting)
- Expression values are not available in TPM, so the per-transcript sampling probability
  cannot be made proportional to RNA concentration
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: IsoTools
  role: Python framework whose saturation analysis workflow provides the depth-detection
    data that the negative binomial curve is fitted to
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: 'Implementation language: define the negative binomial CDF model, fit parameters,
    evaluate on a depth grid, and render saturation_curve.png'
- name: pip
  role: Installation channel for the isotools package used in the workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:03732c5e2513f5e23bcba4f319e7565e4711eeda7b05c5154e9e5f4c2d60df44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# scientific-curve-fitting-visualization

## Summary

Fit a parametric detection-probability model (negative binomial cumulative distribution with depth n, coverage r, and TPM-proportional sampling probability p) to long-read transcriptome saturation data and plot the fitted curves to support sequencing-depth planning for transcript discovery.

## When to use

Use when PacBio LRTS saturation-analysis data relate observed transcript detection to sequencing depth and you must estimate how deep to sequence before transcripts at a given expression level (e.g. 1 TPM) become reliably detectable.

## When NOT to use

- The input is short-read RNA-seq quantification rather than long-read (PacBio IsoSeq) saturation data with per-transcript read sampling
- The goal is detection of differential alternative splicing (use the beta-binomial differential-event workflow instead of depth-planning curve fitting)
- Expression values are not available in TPM, so the per-transcript sampling probability cannot be made proportional to RNA concentration

## Inputs

- IsoTools saturation-analysis transcriptome object (e.g. from notebooks/04_saturation_analysis.ipynb, imported from LRTS bam files plus gff3/gtf reference annotation)
- Measured depth-detection relationship (observation counts per transcript at increasing sequencing depths)
- Transcript expression levels in TPM

## Outputs

- Fitted negative-binomial observation-probability curve over a sequencing-depth grid
- Predicted observation probabilities for 1-TPM and >=2-TPM transcripts at the reported depths
- Figure file saturation_curve.png

## How to apply

Load the transcriptome prepared with the IsoTools saturation analysis workflow (notebooks/04_saturation_analysis.ipynb in the isotools repository). In Python, define the observation-probability model as the cumulative distribution function of a negative binomial with parameters n (depth), r (minimum required coverage), and p (per-transcript sampling probability proportional to its RNA concentration in TPM). Apply the IsoSeq two-copy discovery threshold so a transcript counts as observed once at least two reads are sampled. Fit the model against the measured depth-detection relationship from the saturation data, evaluate the fitted curve over a depth grid, and record predicted observation probabilities for transcripts at 1 TPM and at >=2 TPM. Finally plot the fitted observation-probability curves versus sequencing depth with IsoTools/Python (e.g. matplotlib) and save the figure. The rationale is that the fitted curve predicts, at the reported hepatocyte depths, even rare transcripts (one TPM) have over 70% observation probability while transcripts at two TPM or more are detected with at least two reads with close to 100% probability.

## Related tools

- **IsoTools** (Python framework whose saturation analysis workflow provides the depth-detection data that the negative binomial curve is fitted to) — https://github.com/MatthiasLienhard/isotools
- **Python** (Implementation language: define the negative binomial CDF model, fit parameters, evaluate on a depth grid, and render saturation_curve.png)
- **pip** (Installation channel for the isotools package used in the workflow)

## Examples

```
python3 -m pip install isotools; # in notebooks/04_saturation_analysis.ipynb: fit negative binomial CDF (n, r, p~TPM) to depth-detection data and save saturation_curve.png
```

## Evaluation signals

- The fitted curve predicts one-TPM transcripts are observed with >70% probability at the reported hepatocyte depths
- The fitted curve predicts >=2-TPM transcripts are detected with at least two reads with close to 100% probability
- Fitted curve reproduces the qualitative depth-detection relationship measured in the saturation analysis data
- Figure saturation_curve.png is produced showing observation probability versus sequencing depth for both expression classes
- Model counts a transcript as observed only when at least two reads are sampled (two-copy discovery threshold)

## Limitations

- The model assumes per-transcript sampling probability is proportional to RNA concentration (TPM); deviations from this assumption bias the fitted curve
- Validation of the fit is qualitative (qualitative saturation result), not a quantitative goodness-of-fit criterion
- Predictions are anchored to the reported PacBio SequelII human hepatocyte depths and may not transfer to other platforms or tissues

## Evidence

- [other] Negative-binomial saturation model prediction: [evidence span withheld — anchor status: not_found]
- [results] Observed saturation result: [evidence span withheld — anchor status: not_found]
- [intro] Tool identity: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [readme] Repository installation: [evidence span withheld — anchor status: external_doc]

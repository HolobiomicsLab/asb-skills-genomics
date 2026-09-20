---
name: transcript-detection-saturation-analysis
description: Use when planning long-read transcriptome sequencing (LRTS, e.g. PacBio
  Iso-Seq) depth and you must predict the observation probability of transcripts at
  given expression levels (e.g. 1 TPM vs >=2 TPM) as a function of sequencing depth.
when_to_use_negative:
- Input is raw Iso-Seq reads that have not yet been imported and quantified with IsoTools
  (bam import, QC metrics, transcript reconstruction) — run the standard IsoTools
  workflow first
- The goal is alternative splicing event detection or differential splicing between
  sample groups (beta-binomial differential analysis), not sequencing-depth planning
- Data are short-read RNA-seq rather than long-read transcriptome sequencing, where
  the two-copy full-length-transcript discovery threshold does not apply
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0092
tools:
- name: IsoTools
  role: Python LRTS analysis framework that imports aligned full-length transcripts,
    computes QC metrics, and supplies the saturation analysis workflow used to build
    the depth-detection relationship
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Language in which the negative-binomial CDF observation model (parameters
    n, r, p) is implemented and the saturation curves are plotted
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:dae1276d41e03cb0d3fb7f7670f503863a0edf2115ed6ef1fd353b505d39a261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# transcript-detection-saturation-analysis

## Summary

A model-based saturation analysis that fits a negative-binomial observation-probability model (parameters n, r, p, with per-transcript sampling probability proportional to RNA concentration/TPM) to LRTS depth-detection data in order to estimate the sequencing depth needed for transcript discovery with IsoTools.

## When to use

Use when planning long-read transcriptome sequencing (LRTS, e.g. PacBio Iso-Seq) depth and you must predict the observation probability of transcripts at given expression levels (e.g. 1 TPM vs >=2 TPM) as a function of sequencing depth.

## When NOT to use

- Input is raw Iso-Seq reads that have not yet been imported and quantified with IsoTools (bam import, QC metrics, transcript reconstruction) — run the standard IsoTools workflow first
- The goal is alternative splicing event detection or differential splicing between sample groups (beta-binomial differential analysis), not sequencing-depth planning
- Data are short-read RNA-seq rather than long-read transcriptome sequencing, where the two-copy full-length-transcript discovery threshold does not apply

## Inputs

- Transcriptome dataset prepared with the IsoTools saturation analysis workflow (notebooks/04_saturation_analysis.ipynb, isotools repository)
- IsoTools pkl file of imported LRTS bam files with reference annotation (gff3/gtf) and QC metrics
- Per-transcript abundance estimates (TPM) from the IsoTools quantification

## Outputs

- Fitted negative-binomial observation-probability curves over a sequencing-depth grid
- Predicted observation probabilities for 1-TPM and >=2-TPM transcripts at reported depths
- Figure saturation_curve.png

## How to apply

Load transcriptome data prepared with the IsoTools saturation analysis workflow (notebooks/04_saturation_analysis.ipynb in the MatthiasLienhard/isotools repository). In Python, define the observation-probability model as the cumulative distribution function of a negative binomial with parameters n (sequencing depth), r (minimum required coverage), and p (sampling probability proportional to the transcript's RNA concentration in TPM). Apply the IsoSeq two-copy discovery threshold so a transcript counts as observed once at least two reads are sampled. Fit or set n, r, p against the measured depth-detection relationship from the saturation analysis data, evaluate the fitted curve over a depth grid, and record predicted observation probabilities for transcripts at 1 TPM and >=2 TPM. Plot the fitted curves versus depth and save as saturation_curve.png.

## Related tools

- **IsoTools** (Python LRTS analysis framework that imports aligned full-length transcripts, computes QC metrics, and supplies the saturation analysis workflow used to build the depth-detection relationship) — https://github.com/MatthiasLienhard/isotools
- **Python** (Language in which the negative-binomial CDF observation model (parameters n, r, p) is implemented and the saturation curves are plotted)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl')  # then run notebooks/04_saturation_analysis.ipynb to fit the nbinom(n, r, p) saturation model
```

## Evaluation signals

- At the reported human hepatocyte depths, the fitted curve must predict >70% observation probability for one-TPM transcripts (qualitative acceptance criterion from the source workflow)
- At the same depths, transcripts expressed at two TPM or more must be detected with at least two reads with close to 100% probability
- The fitted negative-binomial curve should reproduce the measured depth-detection relationship from the IsoTools saturation analysis data across the depth grid
- The observation model must enforce the IsoSeq two-copy discovery threshold (a transcript is observed only when at least two reads are sampled)
- Output figure saturation_curve.png is produced and its curves increase monotonically with sequencing depth toward a probability of 1

## Limitations

- Model assumes per-transcript sampling probability is proportional to RNA concentration (TPM); departures from this assumption (e.g. length- or GC-dependent capture bias) are not modeled
- Results are tied to the reported hepatocyte depths; predictions for untested depth ranges are extrapolations of the fitted curve
- IsoTools saturation analysis is demonstrated on PacBio SequelII LRTS data; behavior on other platforms is not established in this context

## Evidence

- [intro] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [readme] Import of LRTS bam files and quantification context: "Import of LRTS bam files (aligned full length transcripts)."
- [methods] installation via pip: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [abstract] application to hepatocyte data demonstrating the platform context: "we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid"
- [readme] readthedocs tutorials as workflow source: [evidence span withheld — anchor status: external_doc]

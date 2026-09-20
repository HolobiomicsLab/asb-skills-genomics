---
name: long-read-transcriptome-sequencing-qc
description: Use when raw LRTS bam files (aligned full-length transcripts, e.g. PacBio
  SequelII IsoSeq data) need QC assessment, filter tagging, or when a saturation/depth
  analysis is needed to decide if rare transcripts (e.g. 1 TPM) are reliably observed.
when_to_use_negative:
- Input is short-read RNA-seq data rather than full-length long-read transcriptome
  sequencing alignments — IsoTools is designed for LRTS bam import of full length
  transcripts.
- The goal is downstream differential splicing detection or alternative splicing event
  definition rather than QC, filtering, or depth assessment.
- The data are not aligned yet — import requires bam files of aligned full length
  transcripts plus a reference annotation, not raw subreads or FASTQ.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: IsoTools
  role: Python-based LRTS analysis framework used to import bam/annotation, compute
    QC metrics, apply filters, and run the negative-binomial saturation analysis for
    sequencing-depth planning
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Language in which the QC and saturation-model steps are executed; used to
    define and fit the negative-binomial observation-probability model
- name: pip
  role: Installation mechanism for the isotools package (python3 -m pip install isotools)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:b48148c29dacbe00a82eefb2d298e7845a9cc5590851d4e704e0ea80a7ff953b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# long-read-transcriptome-sequencing-qc

## Summary

A skill for quality-controlling Long Read Transcriptome Sequencing (LRTS) data using IsoTools, a Python module for LRTS analysis. It covers computing QC metrics on imported PacBio/SequelII bam alignments, filtering suspect reads (e.g. FRAGMENTS not using a reference TSS or PAS), and using IsoTools' negative-binomial saturation analysis to judge whether sequencing depth suffices for transcript discovery.

## When to use

Use when raw LRTS bam files (aligned full-length transcripts, e.g. PacBio SequelII IsoSeq data) need QC assessment, filter tagging, or when a saturation/depth analysis is needed to decide if rare transcripts (e.g. 1 TPM) are reliably observed.

## When NOT to use

- Input is short-read RNA-seq data rather than full-length long-read transcriptome sequencing alignments — IsoTools is designed for LRTS bam import of full length transcripts.
- The goal is downstream differential splicing detection or alternative splicing event definition rather than QC, filtering, or depth assessment.
- The data are not aligned yet — import requires bam files of aligned full length transcripts plus a reference annotation, not raw subreads or FASTQ.

## Inputs

- LRTS bam files of aligned full-length transcripts (e.g. PacBio SequelII IsoSeq, per sample such as CTL and VPA groups)
- Reference annotation in gff3/gtf format (e.g. example.gff.gz)
- Reference genome FASTA for QC metrics computation (e.g. example.fa)
- Saturation analysis data with measured depth-detection relationship

## Outputs

- IsoTools pickle file with QC metrics (e.g. example_1_isotools.pkl)
- Filter tags on reads (e.g. FRAGMENTS tagged when no reference TSS/PAS is used)
- Fitted saturation curves of observation probability versus sequencing depth (saturation_curve.png)
- Predicted observation probabilities for transcripts at 1 TPM and >=2 TPM at reported depths

## How to apply

Import the reference annotation in gff3/gtf format with Transcriptome.from_reference(), then add each sample's aligned full-length transcript bam via add_sample_from_bam (recording sample_name, group, and platform such as 'SequelII'). Compute quality control metrics against a reference genome FASTA with add_qc_metrics() and save the transcriptome object as a pkl file for fast reuse; apply isoseq.add_filter() to tag suspect reads, noting the default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS. For depth planning, use the IsoTools saturation analysis (notebooks/04_saturation_analysis.ipynb): model observation probability as the cumulative distribution function of a negative binomial with parameters n (depth), r (minimum required coverage), and p (per-transcript sampling probability proportional to RNA concentration in TPM), applying the IsoSeq two-copy discovery threshold so a transcript counts as observed once at least two reads are sampled. Evaluate the fitted curve across a depth grid and plot observation-probability curves versus sequencing depth.

## Related tools

- **IsoTools** (Python-based LRTS analysis framework used to import bam/annotation, compute QC metrics, apply filters, and run the negative-binomial saturation analysis for sequencing-depth planning) — https://github.com/MatthiasLienhard/isotools
- **Python** (Language in which the QC and saturation-model steps are executed; used to define and fit the negative-binomial observation-probability model)
- **pip** (Installation mechanism for the isotools package (python3 -m pip install isotools))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('../tests/data/example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('../tests/data/example.fa'); isoseq.save('../tests/data/example_1_isotools.pkl')
```

## Evaluation signals

- The saved IsoTools pkl file (e.g. example_1_isotools.pkl) loads back a Transcriptome with QC metrics computed against the reference FASTA.
- The FRAGMENTS filter tags only reads that do not use a reference TSS or PAS; reads using reference TSS/PAS should remain untagged.
- The fitted negative-binomial saturation curve predicts that at the reported hepatocyte depths one-TPM transcripts have over 70% observation probability.
- The fitted curve predicts transcripts expressed at two TPM or more are detected with at least two reads with close to 100% probability at the reported depths.
- The plotted observation-probability curves versus depth are exported (saturation_curve.png) and are monotonic, saturating toward probability 1.

## Limitations

- The default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS; other quality defects are not covered by this default rule.
- Saturation probability estimates depend on the assumption that per-transcript sampling probability is proportional to RNA concentration (TPM); deviations from this assumption will bias depth predictions.
- QC requires aligned full-length transcript bam files plus a gff3/gtf reference annotation; it cannot be applied to unaligned reads or missing annotation.

## Evidence

- [readme] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [intro] Computation of quality control metrics.: "Computation of quality control metrics."
- [discussion] Default filter tags only non-reference end usage: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] Isotools installation and QC save commands: "python3 -m pip install isotools"
- [intro] Import requires bam and gff3/gtf annotation: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."

---
name: rtts-truncation-fraction-quantification
description: Use when PacBio LRTS bam-aligned full-length transcript data has QC metrics
  computed against a genome FASTA and you must report per-sample fractions of reads
  tagged for truncation, RTTS, internal priming, or FRAGMENTS.
when_to_use_negative:
- The transcriptome pickle has not yet had QC metrics computed via add_qc_metrics
  against a genome FASTA — artifact tags will be missing
- Data are short-read RNA-seq rather than aligned full-length long-read (LRTS) bam
  imports, which IsoTools is designed for
- The goal is differential alternative splicing detection rather than technical artifact
  rate estimation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: IsoTools
  role: Python LRTS analysis framework whose Transcriptome API provides add_qc_metrics
    and per-read artifact tagging used to compute artifact fractions
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Language in which the Transcriptome API is called to load the pickle, run
    QC, aggregate artifact tags, and write qc_artifact_fractions.csv
- name: pip
  role: Installation channel for the isotools package (python3 -m pip install isotools)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:dc69ab7de90c8e08056de1dc7069804fa4abb3d9a5a9ef58a9442ff9f391e347
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# rtts-truncation-fraction-quantification

## Summary

Quantify the fraction of long-read transcriptome sequencing (LRTS) reads affected by technical artifacts — transcript truncation, RTTS, internal priming, and FRAGMENTS — in PacBio hepatocyte samples (CTL vs VPA) using IsoTools per-read QC metrics, reproducing reported fractions of 10.5% (CTL) and 9.3% (VPA) affected reads.

## When to use

Use when PacBio LRTS bam-aligned full-length transcript data has QC metrics computed against a genome FASTA and you must report per-sample fractions of reads tagged for truncation, RTTS, internal priming, or FRAGMENTS.

## When NOT to use

- The transcriptome pickle has not yet had QC metrics computed via add_qc_metrics against a genome FASTA — artifact tags will be missing
- Data are short-read RNA-seq rather than aligned full-length long-read (LRTS) bam imports, which IsoTools is designed for
- The goal is differential alternative splicing detection rather than technical artifact rate estimation

## Inputs

- Filtered IsoTools transcriptome pickle file (e.g. example_1_isotools.pkl) containing CTL and VPA PacBio SequelII samples
- Genome FASTA file (e.g. genome.fa) used as reference for add_qc_metrics
- Sample group labels (CTL, VPA) defined at bam import

## Outputs

- qc_artifact_fractions.csv with per-group read counts and affected-read fractions per artifact class (truncation, RTTS, internal priming, FRAGMENTS)
- Total artifact-affected read fractions per sample group (expected: 10.5% CTL, 9.3% VPA)

## How to apply

Load the filtered IsoTools transcriptome pickle with the Transcriptome API, then run isoseq.add_qc_metrics('genome.fa') to compute per-read quality control metrics against the genome FASTA. Extract the artifact tags — truncation, RTTS, internal priming, and FRAGMENTS (which marks reads that do not use a reference TSS or PAS) — for the control (CTL) and treatment (VPA) sample groups. Aggregate per-group read counts per artifact class and compute the fraction of affected reads for each class, writing the results to qc_artifact_fractions.csv. Validate against the reported totals: total affected reads 10.5% (CTL) and 9.3% (VPA), truncation 4.4%/3.8%, RTTS 3.8%/3.4%, and internal priming 2.6%/2.4%, within acceptable rounding tolerance; truncation should be the largest contributor.

## Related tools

- **IsoTools** (Python LRTS analysis framework whose Transcriptome API provides add_qc_metrics and per-read artifact tagging used to compute artifact fractions) — https://github.com/MatthiasLienhard/isotools
- **Python** (Language in which the Transcriptome API is called to load the pickle, run QC, aggregate artifact tags, and write qc_artifact_fractions.csv)
- **pip** (Installation channel for the isotools package (python3 -m pip install isotools))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); isoseq.add_qc_metrics('genome.fa'); [fraction per artifact tag for groups CTL and VPA]
```

## Evaluation signals

- Total artifact-affected read fraction reproduces 10.5% for CTL and 9.3% for VPA within acceptable rounding tolerance
- Per-class fractions match truncation 4.4%/3.8%, RTTS 3.8%/3.4%, internal priming 2.6%/2.4% (CTL/VPA)
- Truncation is the largest single contributor among the artifact classes in both groups
- qc_artifact_fractions.csv is produced with one fraction per artifact class per sample group
- FRAGMENTS-tagged reads correspond to reads not using a reference TSS or PAS, consistent with the default filter definition

## Limitations

- Artifact fractions depend on QC metrics having been computed against the correct genome FASTA; the FRAGMENTS tag semantics changed so the default filter only tags reads that do not use a reference TSS or PAS
- Values are subject to rounding tolerance when comparing against the published fractions

## Evidence

- [intro] Computation of quality control metrics.: "Computation of quality control metrics."
- [intro] IsoTools is a python module for LRTS analysis: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [discussion] Default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] README: add_qc_metrics usage: [evidence span withheld — anchor status: external_doc]
- [readme] README: import bam samples CTL and VPA: "for sa in ('CTL', 'VPA'):
    isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa, platform='SequelII')"

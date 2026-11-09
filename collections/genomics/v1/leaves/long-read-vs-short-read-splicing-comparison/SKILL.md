---
name: long-read-vs-short-read-splicing-comparison
description: Use when you have a reconstructed long-read transcriptome (e.g. IsoTools
  pickle from PacBio LRTS) plus paired short-read RNA-seq, and want to know whether
  PSI values agree per splicing event category.
when_to_use_negative:
- Your goal is de novo transcriptome reconstruction or QC of LRTS reads rather than
  cross-platform PSI agreement
- You have only long-read data and no paired short-read RNA-seq, so rMATS quantification
  for comparison is impossible
- Your events are mutually exclusive exons and you expect strong concordance — this
  category showed r=-0.05 in the source study
edam_operation: http://edamontology.org/operation_0451
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python LRTS framework used to define alternative splicing events from segment
    graphs and export them in rMATS format
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Short-read tool used to compute PSI values for the exported events from paired
    RNA-seq data
- name: Python
  role: Language in which the IsoTools workflow, event extraction, and correlation
    analysis are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:6f358e76524bcd5ad1bdeb76c1ecf76ec3a6434b94955ee3d415e3166c323158
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: http://edamontology.org/operation_0451
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# long-read-vs-short-read-splicing-comparison

## Summary

Compares PSI values of alternative splicing events defined from long-read transcriptome sequencing (LRTS) data with short-read RNA-seq quantification by rMATS, using IsoTools' segment-graph event definition and rMATS-format export. It quantifies per-category agreement (ES, 5AS, 3AS, IR, ME) via Pearson correlation.

## When to use

Use when you have a reconstructed long-read transcriptome (e.g. IsoTools pickle from PacBio LRTS) plus paired short-read RNA-seq, and want to know whether PSI values agree per splicing event category.

## When NOT to use

- Your goal is de novo transcriptome reconstruction or QC of LRTS reads rather than cross-platform PSI agreement
- You have only long-read data and no paired short-read RNA-seq, so rMATS quantification for comparison is impossible
- Your events are mutually exclusive exons and you expect strong concordance — this category showed r=-0.05 in the source study

## Inputs

- Reconstructed LRTS transcriptome pickle (e.g. example_isotools.pkl) loadable with IsoTools Transcriptome
- Paired short-read RNA-seq data (e.g. BAM/FASTQ) for the same samples as rMATS input
- Reference annotation in gff3/gtf format (used when building the transcriptome)

## Outputs

- rMATS-format event files exported from IsoTools
- Per-category event counts and Pearson PSI correlation coefficients (e.g. splice_events_psi_correlations.tsv)

## How to apply

Load the reconstructed LRTS transcriptome (e.g. example_isotools.pkl) with IsoTools' Transcriptome class. Extract binary alternative splicing events from the gene segment graphs, categorizing them as intron retention (IR), exon skipping (ES), alternative 3' splice sites (3AS), alternative 5' splice sites (5AS), and mutually exclusive exons (ME). Export these events in rMATS event format using IsoTools' export function so short-read quantification is directly comparable. Compute PSI values for each event category from the paired short-read RNA-seq data with rMATS and correlate them per category against the long-read PSI values. Write per-category event counts and Pearson correlation coefficients to a table such as splice_events_psi_correlations.tsv. In the hepatocyte case study, agreement was high for ES (r=0.84), 5AS (r=0.80), 3AS (r=0.74) and IR (r=0.58), but essentially absent for ME events (r=-0.05).

## Related tools

- **IsoTools** (Python LRTS framework used to define alternative splicing events from segment graphs and export them in rMATS format) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read tool used to compute PSI values for the exported events from paired RNA-seq data)
- **Python** (Language in which the IsoTools workflow, event extraction, and correlation analysis are executed)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); isoseq.alt_splicing(); isoseq.export_splicing_events('rmats_events.tsv')
```

## Evaluation signals

- Output table contains event counts for all five categories: approx. 3797 IR, 2713 ES, 1526 3AS, 1243 5AS, 376 ME
- PSI correlations fall in expected ranges: ES r=0.84, 5AS r=0.80, 3AS r=0.74, IR r=0.58
- ME events show no meaningful long-read/short-read PSI agreement (r=-0.05), serving as a negative-control category
- Exported events parse as valid rMATS event format for the short-read quantification step

## Limitations

- Mutually exclusive exons (ME) show no agreement between LRTS and short-read PSI (r=-0.05) and should not be treated as concordant
- Intron retention shows the weakest agreement among concordant categories (r=0.58)
- Event counts and correlations are validated against a human hepatocyte PacBio (SequelII) dataset treated with valproic acid; other datasets may differ

## Evidence

- [intro] graph-based event definition: "Definition of alternative splicing events based on segment graphs."
- [discussion] export for rMATS: "added option to export alternative splicing events for MISO and rMATS"
- [abstract] LRTS vs short-read value: "Our results indicate that LRTS can provide valuable insights into alternative splicing, particularly in terms of complex and differential splicing patterns, in comparison to short-read RNA-seq."
- [readme] IsoTools import workflow: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."

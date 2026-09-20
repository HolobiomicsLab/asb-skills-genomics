---
name: alternative-splicing-event-classification
description: Use when aligned full-length LRTS reads (e.g. PacBio SequelII bam) plus
  a gff3/gtf reference annotation are available and you need graph-based definition
  and classification of alternative splicing events (IR, ES, 3AS, 5AS, ME) for quantification
  or export.
when_to_use_negative:
- Input is short-read-only RNA-seq without full-length LRTS alignments — segment-graph
  event definition from IsoTools requires long-read transcript structures
- The goal is transcriptome reconstruction or QC metric computation from raw bam files
  rather than event classification from an already-reconstructed transcriptome
- Only differential splicing statistics (beta-binomial) between sample groups are
  needed and events are already defined
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: IsoTools
  role: Python module that defines alternative splicing events from gene segment graphs
    and exports them for rMATS/MISO
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Receives IsoTools-exported events and provides short-read PSI quantification
    for cross-correlation
- name: Python
  role: Language in which IsoTools is implemented; pip-installable and used via the
    Transcriptome class API
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:71aef70eac589ffe9d6550d387ddab6db54a09d4bb01c035d83fb7e0def99476
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# alternative-splicing-event-classification

## Summary

Defines and classifies alternative splicing events (IR, ES, 3AS, 5AS, ME) from long-read transcriptome sequencing (LRTS) data using IsoTools' segment-graph method, and exports events in rMATS/MISO format so PSI values can be cross-validated against short-read quantification.

## When to use

Use when aligned full-length LRTS reads (e.g. PacBio SequelII bam) plus a gff3/gtf reference annotation are available and you need graph-based definition and classification of alternative splicing events (IR, ES, 3AS, 5AS, ME) for quantification or export.

## When NOT to use

- Input is short-read-only RNA-seq without full-length LRTS alignments — segment-graph event definition from IsoTools requires long-read transcript structures
- The goal is transcriptome reconstruction or QC metric computation from raw bam files rather than event classification from an already-reconstructed transcriptome
- Only differential splicing statistics (beta-binomial) between sample groups are needed and events are already defined

## Inputs

- IsoTools transcriptome pickle (e.g. example_isotools.pkl) reconstructed from LRTS bam files
- Reference annotation in gff3/gtf format
- Paired short-read RNA-seq data (for rMATS PSI quantification)

## Outputs

- Classified alternative splicing events (IR, ES, 3AS, 5AS, ME) per gene segment graph
- Events exported in rMATS event format (optionally MISO format)
- Per-category event counts and Pearson PSI correlations table (splice_events_psi_correlations.tsv)

## How to apply

Load the reconstructed transcriptome pickle (e.g. example_isotools.pkl) with the IsoTools Transcriptome class, then derive binary alternative splicing events from each gene's segment graph using IsoTools' graph-based event definition, categorizing events as intron retention (IR), exon skipping (ES), alternative 3' splice sites (3AS), alternative 5' splice sites (5AS), and mutually exclusive exons (ME). Use the add_filter step (e.g. isoseq.add_filter(); isoseq.save('example_isotools.pkl')) before event extraction so fragments and other low-quality classes do not pollute the event set — note the default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS. Export the defined events in rMATS event format via IsoTools' export function (MISO export is also supported) so they can be paired with short-read RNA-seq quantification. Compute PSI per event category from the short-read data and correlate per category with the long-read PSI values to judge agreement. Rationale: IsoTools integrates a graph-based method for identifying alternative splicing events with a beta-binomial approach for differential testing, so event calls come directly from the segment graph rather than per-transcript annotation comparison.

## Related tools

- **IsoTools** (Python module that defines alternative splicing events from gene segment graphs and exports them for rMATS/MISO) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Receives IsoTools-exported events and provides short-read PSI quantification for cross-correlation)
- **Python** (Language in which IsoTools is implemented; pip-installable and used via the Transcriptome class API)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('../tests/data/example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- Output table contains event counts for all five categories, approximately 3797 IR, 2713 ES, 1526 3AS, 1243 5AS, and 376 ME events
- Per-category Pearson PSI correlations fall in expected ranges: ES r=0.84, 5AS r=0.8, 3AS r=0.74, IR r=0.58
- ME events show near-zero/negative correlation (r=-0.05) between LRTS and short-read PSI — a sanity anchor rather than a failure
- splice_events_psi_correlations.tsv is produced with per-category counts and correlation coefficients
- Events export cleanly in rMATS event format for downstream quantification

## Limitations

- Mutually exclusive exon (ME) events show essentially no PSI agreement between long-read and short-read quantification (r=-0.05), so ME results should not be cross-validated this way
- Agreement with short-read quantification is category-dependent and varies widely (IR r=0.58 vs ES r=0.84)
- The default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS, so other fragment-like artifacts may require manual filters before event extraction

## Evidence

- [intro] Definition of alternative splicing events based on segment graphs.: "Definition of alternative splicing events based on segment graphs."
- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [discussion] added option to export alternative splicing events for MISO and rMATS: "added option to export alternative splicing events for MISO and rMATS"
- [readme] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [readme] Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format.: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."

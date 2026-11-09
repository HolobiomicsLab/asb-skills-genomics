---
name: rmats-event-format-export
description: Use when you have an IsoTools-reconstructed LRTS transcriptome (e.g.
  example_isotools.pkl) and need to compare long-read splicing event PSI values against
  short-read RNA-seq quantified by rMATS.
when_to_use_negative:
- Input is short-read RNA-seq alone with no long-read transcriptome reconstruction
  — run rMATS directly instead of exporting events from IsoTools
- The goal is differential splicing testing between sample groups rather than cross-platform
  PSI agreement — use IsoTools' beta-binomial differential event detection instead
- Events of interest are limited to mutually exclusive exons — ME events show no PSI
  agreement between LRTS and rMATS (r=-0.05), so rMATS comparison is uninformative
  for them
edam_operation: http://edamontology.org/operation_0262
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python framework used to define alternative splicing events from segment graphs
    and export them in rMATS event format
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Short-read splicing quantification tool against whose event format and PSI
    values the LRTS events are compared
- name: Python
  role: Language in which the IsoTools export and correlation workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:099e44af6cbdc43a15075b6729a0d70868dd43c467e8cd340dd09d1e93dbfce5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - rMATS
  edam_operation: http://edamontology.org/operation_0262
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# rmats-event-format-export

## Summary

Exports IsoTools graph-defined alternative splicing events (IR, ES, 3AS, 5AS, ME) from a long-read transcriptome into rMATS event format so PSI values from LRTS can be directly correlated with short-read rMATS quantification.

## When to use

Use when you have an IsoTools-reconstructed LRTS transcriptome (e.g. example_isotools.pkl) and need to compare long-read splicing event PSI values against short-read RNA-seq quantified by rMATS.

## When NOT to use

- Input is short-read RNA-seq alone with no long-read transcriptome reconstruction — run rMATS directly instead of exporting events from IsoTools
- The goal is differential splicing testing between sample groups rather than cross-platform PSI agreement — use IsoTools' beta-binomial differential event detection instead
- Events of interest are limited to mutually exclusive exons — ME events show no PSI agreement between LRTS and rMATS (r=-0.05), so rMATS comparison is uninformative for them

## Inputs

- IsoTools transcriptome pickle (e.g. example_isotools.pkl) from PacBio LRTS hepatocyte data
- Paired short-read RNA-seq data (BAM) for rMATS PSI quantification
- Reference annotation in gff3/gtf format

## Outputs

- rMATS-format event files for IR, ES, 3AS, 5AS, and ME categories
- Per-category event counts and Pearson PSI correlation coefficients (e.g. splice_events_psi_correlations.tsv)

## How to apply

Load the reconstructed hepatocyte transcriptome pickle with the IsoTools Transcriptome class, then extract binary alternative splicing events from the gene segment graphs using IsoTools' graph-based event definition, categorizing them as intron retention (IR), exon skipping (ES), alternative 3' splice sites (3AS), alternative 5' splice sites (5AS), and mutually exclusive exons (ME). Export the defined events in rMATS event format using IsoTools' export function so paired short-read RNA-seq data can be quantified on the same event coordinates. Compute PSI values per event category from the short-read data and correlate them per category with the LRTS PSI values using Pearson correlation. Expected agreement is high for ES (r=0.84), 5AS (r=0.8), and 3AS (r=0.74), moderate for IR (r=0.58), and absent for ME (r=-0.05); write per-category event counts and correlations to a TSV (e.g. splice_events_psi_correlations.tsv).

## Related tools

- **IsoTools** (Python framework used to define alternative splicing events from segment graphs and export them in rMATS event format) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read splicing quantification tool against whose event format and PSI values the LRTS events are compared)
- **Python** (Language in which the IsoTools export and correlation workflow is executed)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); isoseq.alternative_splicing_events(); isoseq.export_splicing_events('rmats_events.tsv')
```

## Evaluation signals

- Output table contains event counts for all five categories, approx. 3797 IR, 2713 ES, 1526 3AS, 1243 5AS, 376 ME
- PSI correlations fall in expected ranges: ES r=0.84, 5AS r=0.8, 3AS r=0.74, IR r=0.58
- ME events show near-zero or negative correlation (r≈-0.05), which is an expected negative control
- Exported event files parse as valid rMATS event format for downstream PSI computation

## Limitations

- Mutually exclusive exon (ME) events do not show LRTS–short-read PSI agreement (r=-0.05), limiting cross-platform validation for this category
- Intron retention agreement is only moderate (r=0.58), weaker than ES, 5AS and 3AS
- Requires a reconstructed IsoTools transcriptome pickle; raw BAM imports must first go through transcriptome reconstruction

## Evidence

- [other] IsoTools-detected splicing events quantified from LRTS show high PSI correlation with rMATS short-read quantification for ES (r = 0.84), 5AS (r = 0.8), 3AS (r = 0.74), and IR (r = 0.58), but not for ME events (r = -0.05).: [evidence span withheld — anchor status: not_found]
- [intro] Definition of alternative splicing events based on segment graphs.: "Definition of alternative splicing events based on segment graphs"
- [discussion] added option to export alternative splicing events for MISO and rMATS: "added option to export alternative splicing events for MISO and rMATS"
- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution: "a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution"
- [readme] python3 -m pip install isotools: [evidence span withheld — anchor status: external_doc]

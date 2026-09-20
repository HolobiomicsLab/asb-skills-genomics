---
name: psi-correlation-across-sequencing-protocols
description: Use when paired long-read (e.g. PacBio LRTS) and short-read RNA-seq data
  are available for the same sample and you need to validate that splice events detected
  from full-length transcript alignments agree in PSI with short-read quantification.
when_to_use_negative:
- Only a single sequencing protocol is available — rMATS PSI cannot be computed without
  the paired short-read RNA-seq data to compare against
- The transcriptome reconstruction has not yet been performed (no IsoTools pickle
  exists); import LRTS BAM files and reference annotation first
- You only need de novo event detection or differential splicing testing on long-read
  data alone; IsoTools' beta-binomial differential analysis is the appropriate path,
  not cross-protocol correlation
edam_operation: http://edamontology.org/operation_0451
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python LRTS analysis framework used to load the transcriptome, define alternative
    splicing events from segment graphs, and export events in rMATS format
  repo: https://github.com/MatthiasLienhard/isotools
- name: rMATS
  role: Short-read RNA-seq tool used to compute PSI values on the exported event definitions
    for cross-protocol comparison
- name: Python
  role: Language in which the IsoTools-based event extraction, export, and correlation
    analysis is scripted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:f86199bf47a110c83f3966a3f0ca92f4e2a68d7501dfabd929310d970dc94f7a
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# psi-correlation-across-sequencing-protocols

## Summary

Cross-protocol validation skill that quantifies percent-spliced-in (PSI) values of alternative splicing events from long-read transcriptome sequencing (LRTS) using IsoTools' graph-based event definitions, exports them in rMATS event format, and computes per-category Pearson correlations against short-read RNA-seq rMATS quantification to assess agreement.

## When to use

Use when paired long-read (e.g. PacBio LRTS) and short-read RNA-seq data are available for the same sample and you need to validate that splice events detected from full-length transcript alignments agree in PSI with short-read quantification.

## When NOT to use

- Only a single sequencing protocol is available — rMATS PSI cannot be computed without the paired short-read RNA-seq data to compare against
- The transcriptome reconstruction has not yet been performed (no IsoTools pickle exists); import LRTS BAM files and reference annotation first
- You only need de novo event detection or differential splicing testing on long-read data alone; IsoTools' beta-binomial differential analysis is the appropriate path, not cross-protocol correlation

## Inputs

- Reconstructed LRTS transcriptome pickle (e.g. example_isotools.pkl) loadable by IsoTools' Transcriptome class
- Paired short-read RNA-seq BAM files for the same samples (for rMATS quantification)
- Reference annotation in gff3/gtf format used to build the transcriptome

## Outputs

- Tabular file of per-category event counts and Pearson PSI correlation coefficients (e.g. splice_events_psi_correlations.tsv)
- Alternative splicing events exported in rMATS event format

## How to apply

Load the reconstructed transcriptome (e.g. example_isotools.pkl) with IsoTools' Transcriptome class, then extract binary alternative splicing events from the gene segment graphs, categorizing them as intron retention (IR), exon skipping (ES), alternative 3' splice sites (3AS), alternative 5' splice sites (5AS), and mutually exclusive exons (ME). Export the defined events in rMATS event format with IsoTools' export function so that short-read RNA-seq data can be quantified on identical event definitions. Compute PSI per event category from the short-read data with rMATS and calculate Pearson correlations per category against the LRTS-derived PSI values, writing event counts and correlation coefficients to a table (e.g. splice_events_psi_correlations.tsv). Rationale: IsoTools defines events on segment graphs so long- and short-read quantifications refer to the same splice junction structures, making PSI directly comparable; agreement varies strongly by event type, so correlation must be reported per category.

## Related tools

- **IsoTools** (Python LRTS analysis framework used to load the transcriptome, define alternative splicing events from segment graphs, and export events in rMATS format) — https://github.com/MatthiasLienhard/isotools
- **rMATS** (Short-read RNA-seq tool used to compute PSI values on the exported event definitions for cross-protocol comparison)
- **Python** (Language in which the IsoTools-based event extraction, export, and correlation analysis is scripted)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); events = isoseq.alternative_splicing_events(); isoseq.write_gtf('events_rmats.gtf')
```

## Evaluation signals

- Output table contains event counts for all five categories in expected ranges: approx. 3797 IR, 2713 ES, 1526 3AS, 1243 5AS, 376 ME
- PSI correlation coefficients fall in expected ranges: ES r=0.84, 5AS r=0.8, 3AS r=0.74, IR r=0.58
- ME events are expected to show no agreement (r=-0.05); a strong ME correlation is a red flag
- Exported events parse correctly as rMATS input, confirming the rMATS event format export succeeded
- PSI values for long and short reads are computed on identical event definitions (same splice junctions), not on independently detected event sets

## Limitations

- Agreement is event-type dependent: mutually exclusive exons (ME) show no correlation between protocols (r = -0.05), while IR is only moderate (r = 0.58); expect lower agreement than ES/AS categories
- The event counts and correlation values are specific to the PacBio LRTS human hepatocyte (valproic acid-treated) case study; other datasets will yield different numbers
- Default IsoTools filtering tags FRAGMENTS reads that do not use a reference TSS or PAS, which affects which transcripts enter the event graph

## Evidence

- [intro] Definition of alternative splicing events based on segment graphs: "Definition of alternative splicing events based on segment graphs."
- [discussion] MISO/rMATS export option: "added option to export alternative splicing events for MISO and rMATS"
- [abstract] graph-based event identification: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution"
- [methods] pip installation: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [readme] README basic import workflow: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."

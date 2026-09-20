---
name: mutually-exclusive-exon-analysis
description: Use when long-read transcriptome data (aligned full-length transcript
  BAMs) must be queried for specific alternative splicing events such as mutually
  exclusive exons or exon skipping/inclusion, and per-condition PSI values are needed.
when_to_use_negative:
- Input is short-read RNA-seq rather than long-read (LRTS) full-length transcript
  alignments — IsoTools is designed for LRTS analysis
- No processed Transcriptome pickle exists and raw bam/gff3 inputs plus a genome FASTA
  for add_qc_metrics are unavailable
- The question is de novo discovery of all differential events rather than reproducing/quantifying
  specific known events
edam_operation: http://edamontology.org/operation_0447
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python LRTS framework providing SegmentGraph-based definition of alternative
    splicing events, event quantification (PSI), and differential testing; used to
    load the Transcriptome pickle and retrieve/quantify the SLC39A14 and FN1 events
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Execution environment for the IsoTools workflow (installable with pip as 'isotools')
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:87e81a152fc8b69d6afc0f71dfe33078284eb2a767954d68bb10105bb22eb360
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_0447
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# mutually-exclusive-exon-analysis

## Summary

Quantify mutually exclusive exon events (e.g. SLC39A14 exons 4A/4B) and exon inclusion events (e.g. FN1 exon 32/EDA) from PacBio LRTS data using IsoTools' SegmentGraph-based alternative splicing machinery, and compute per-group PSI values.

## When to use

Use when long-read transcriptome data (aligned full-length transcript BAMs) must be queried for specific alternative splicing events such as mutually exclusive exons or exon skipping/inclusion, and per-condition PSI values are needed.

## When NOT to use

- Input is short-read RNA-seq rather than long-read (LRTS) full-length transcript alignments — IsoTools is designed for LRTS analysis
- No processed Transcriptome pickle exists and raw bam/gff3 inputs plus a genome FASTA for add_qc_metrics are unavailable
- The question is de novo discovery of all differential events rather than reproducing/quantifying specific known events

## Inputs

- Processed IsoTools Transcriptome pickle (e.g. example_isotools.pkl) built from gff3 reference annotation and aligned PacBio LRTS bam files
- Reference annotation in gff3/gtf format (e.g. example.gff.gz)
- Aligned full-length transcript BAM files per sample (CTL and VPA groups, SequelII platform)

## Outputs

- Per-condition PSI table for the target events (SLC39A14 exon 4B proportion, FN1 EDA PSI in control vs VPA) written to psi_values.csv
- Retrieved alternative splicing event objects (mutually exclusive exon at SLC39A14 exons 4A/4B; exon inclusion at FN1 exon 32/EDA)

## How to apply

Load the processed IsoTools Transcriptome object from the saved pickle (e.g. example_isotools.pkl), which was built with from_reference on a gff3 annotation, add_sample_from_bam per sample, add_qc_metrics, make_index, and add_filter. Use IsoTools' SegmentGraph-based alternative splicing event machinery to retrieve the mutually exclusive exon event at SLC39A14 exons 4A/4B and the exon inclusion event at FN1 exon 32 (EDA). Quantify PSI per sample group with IsoTools' event quantification, aggregating transcript support counts for the inclusion versus skipping isoforms within the control (CTL) and VPA groups. Assemble the per-condition PSI values (SLC39A14 exon 4B proportion and FN1 EDA PSI in control vs VPA) into a table and write it to psi_values.csv. Correctness is judged by reproducing the reported events, e.g. FN1 EDA PSI rising from 9% in CTL to 15% in VPA-treated hepatocytes with q-value = 3.55E−111.

## Related tools

- **IsoTools** (Python LRTS framework providing SegmentGraph-based definition of alternative splicing events, event quantification (PSI), and differential testing; used to load the Transcriptome pickle and retrieve/quantify the SLC39A14 and FN1 events) — https://github.com/MatthiasLienhard/isotools
- **Python** (Execution environment for the IsoTools workflow (installable with pip as 'isotools'))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); psi = isoseq.alternative_splicing_events(query={'gene_name': 'FN1'})
```

## Evaluation signals

- FN1 exon 32 (EDA) PSI reproduces the reported values: 9% in CTL rising to 15% under VPA treatment, with q-value = 3.55E−111 (most significant differential ES event)
- The mutually exclusive exon event at SLC39A14 exons 4A/4B is successfully retrieved from the SegmentGraph event machinery
- PSI values are computed per sample group by aggregating transcript support counts for inclusion versus skipping isoforms within CTL and VPA
- psi_values.csv is produced containing per-condition PSI values for both the SLC39A14 exon 4B proportion and FN1 EDA event

## Limitations

- Relies on the prior preprocessing workflow (from_reference, add_sample_from_bam, add_qc_metrics, make_index, add_filter, save) having produced the pickle; results cannot be computed from unprocessed bams alone
- PSI estimates depend on aggregated transcript support counts, so low-coverage events may be unreliable
- IsoTools was demonstrated on PacBio SequelII LRTS data of human hepatocytes treated with valproic acid; other platforms/conditions are untested in this context

## Evidence

- [intro] Definition of alternative splicing events based on segment graphs.: "Definition of alternative splicing events based on segment graphs."
- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [methods] isoseq.add_filter()
    isoseq.save('example_isotools.pkl'): "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [readme] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."

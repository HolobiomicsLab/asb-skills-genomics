---
name: psi-computation-from-transcript-support-counts
description: Use when you have an IsoTools Transcriptome object built from PacBio
  LRTS bam files and need per-condition PSI values for a specific alternative splicing
  event (e.g., FN1 exon 32 EDA or SLC39A14 exon 4A/4B) in defined sample groups.
when_to_use_negative:
- Input is short-read RNA-seq data rather than aligned full-length long-read transcripts
  (bam files of full-length transcripts), since IsoTools event definition is based
  on segment graphs over LRTS reads
- You only have raw bam files without a constructed Transcriptome object — the import/QC
  workflow (from_reference, add_sample_from_bam, add_qc_metrics, make_index, add_filter)
  must be run first
- The splicing event is not representable as an IsoTools segment-graph event type
  (e.g., only gene-level expression quantification is needed)
edam_operation: http://edamontology.org/operation_0451
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python LRTS analysis framework used to define alternative splicing events
    from segment graphs, quantify event PSI from transcript support counts, and detect
    differential splicing between sample groups
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Execution environment in which the IsoTools Transcriptome is loaded, events
    are retrieved and quantified, and psi_values.csv is assembled
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:f66ef9c88fd0c0d21c0620802a0147484c4559b4ed90e6a60cc459c640e20083
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_0451
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# psi-computation-from-transcript-support-counts

## Summary

Compute per-sample PSI (percent-spliced-in) values for alternative splicing events such as exon skipping and exon inclusion from long-read transcriptome data using IsoTools' segment-graph event machinery, aggregating transcript support counts for inclusion versus skipping isoforms across sample groups (e.g., CTL vs VPA-treated hepatocytes).

## When to use

Use when you have an IsoTools Transcriptome object built from PacBio LRTS bam files and need per-condition PSI values for a specific alternative splicing event (e.g., FN1 exon 32 EDA or SLC39A14 exon 4A/4B) in defined sample groups.

## When NOT to use

- Input is short-read RNA-seq data rather than aligned full-length long-read transcripts (bam files of full-length transcripts), since IsoTools event definition is based on segment graphs over LRTS reads
- You only have raw bam files without a constructed Transcriptome object — the import/QC workflow (from_reference, add_sample_from_bam, add_qc_metrics, make_index, add_filter) must be run first
- The splicing event is not representable as an IsoTools segment-graph event type (e.g., only gene-level expression quantification is needed)

## Inputs

- Processed IsoTools Transcriptome pickle file (e.g., example_isotools.pkl) built from PacBio LRTS bam alignments and gff3/gtf reference annotation
- Sample group definitions (e.g., CTL vs VPA groups of SequelII hepatocyte samples)
- Gene/event identifiers of interest (e.g., SLC39A14 exons 4A/4B, FN1 exon 32/EDA)

## Outputs

- Per-sample and per-group PSI values for the queried alternative splicing events
- A table of per-condition PSI values written to CSV (e.g., psi_values.csv)

## How to apply

Load the processed IsoTools Transcriptome object (e.g., example_isotools.pkl, produced by from_reference on the gff3 annotation, add_sample_from_bam for each sample, add_qc_metrics, make_index, add_filter, save). Use IsoTools' SegmentGraph-based alternative splicing event machinery to retrieve the event of interest, such as the mutually exclusive exon event at SLC39A14 exons 4A/4B or the exon inclusion event at FN1 exon 32/EDA. Quantify the event with IsoTools' event quantification, aggregating transcript support counts for the inclusion versus skipping isoforms within each sample group (control vs VPA). Assemble the per-condition PSI values into a table (e.g., psi_values.csv) and, for differential significance, rely on IsoTools' beta-binomial statistical framework that underlies differential event detection.

## Related tools

- **IsoTools** (Python LRTS analysis framework used to define alternative splicing events from segment graphs, quantify event PSI from transcript support counts, and detect differential splicing between sample groups) — https://github.com/MatthiasLienhard/isotools
- **Python** (Execution environment in which the IsoTools Transcriptome is loaded, events are retrieved and quantified, and psi_values.csv is assembled)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); ev = next(e for g in isoseq.genes('FN1') for e in g.alt_splicing if e.type == 'exon skipping'); psi = ev.psi(groups={'CTL': [...], 'VPA': [...]})
```

## Evaluation signals

- FN1 exon 32 (EDA) PSI should reproduce ~9% in the CTL group and ~15% in the VPA-treated hepatocyte group, matching the reported differential ES event
- The SLC39A14 exon 4B proportion should reflect the mutually exclusive exon 4A/4B event retrieved via the segment-graph machinery
- PSI values must be bounded between 0 and 1 and derived from aggregated support counts of inclusion versus skipping isoforms within each group
- psi_values.csv should contain one row per event per condition with the per-group PSI assembled from the IsoTools event quantification
- The Transcriptome pickle must have been produced through the documented chain: from_reference, add_sample_from_bam, add_qc_metrics, make_index, add_filter, save

## Limitations

- PSI estimates depend on prior filtering (e.g., the FRAGMENTS filter tags reads that do not use a reference TSS or PAS), so filter choices affect the support counts
- Event definitions require alternative splicing events to be captured by the segment-graph representation; complex events must be retrievable through IsoTools' event types
- Differential significance relies on IsoTools' beta-binomial statistical approach, not on the PSI values alone

## Evidence

- [abstract] graph-based event PSI quantification: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] segment-graph event definition: "Definition of alternative splicing events based on segment graphs."
- [intro] differential splicing detection: "Detection of differential alternative splicing between samples and groups of samples."
- [methods] filter step before save: "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [discussion] FRAGMENTS filter caveat: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] readme import workflow: [evidence span withheld — anchor status: external_doc]

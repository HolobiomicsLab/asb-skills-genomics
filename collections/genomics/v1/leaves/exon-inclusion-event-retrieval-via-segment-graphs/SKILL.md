---
name: exon-inclusion-event-retrieval-via-segment-graphs
description: Use when you need the per-sample or per-group PSI (inclusion level) of
  a known exon inclusion or mutually exclusive exon event from long-read (e.g. PacBio
  SequelII) hepatocyte transcriptome data stored as an IsoTools pickle.
when_to_use_negative:
- Input is short-read RNA-seq data — IsoTools' segment-graph events and beta-binomial
  differential testing are designed for long-read transcriptome sequencing (LRTS)
- The Transcriptome object has not yet been filtered and indexed (add_qc_metrics,
  add_filter, make_index) — event retrieval will not reflect quality-controlled transcript
  support
- The event of interest is a complex or novel splicing pattern not representable as
  a segment-graph alternative splicing event (exon skipping / mutually exclusive exons)
edam_operation: http://edamontology.org/operation_0451
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python module providing the Transcriptome object, SegmentGraph-based alternative
    splicing event definition, and per-group event quantification (PSI) from LRTS
    bam imports
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Execution environment in which the IsoTools Transcriptome is loaded and PSI
    tables are assembled and written to psi_values.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:78adaeea3b0dfa4f43ad170afa55c1d0eb25c42fba90c085a47017b8dfdb8db1
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
yes_rate: 0.6666666666666666
n_claims: 9
---

# exon-inclusion-event-retrieval-via-segment-graphs

## Summary

Retrieve and quantify specific exon inclusion (ES) alternative splicing events, such as FN1 exon 32 (EDA) and the SLC39A14 mutually exclusive exons 4A/4B, from a processed IsoTools Transcriptome object using the SegmentGraph-based event machinery on PacBio long-read transcriptome data.

## When to use

Use when you need the per-sample or per-group PSI (inclusion level) of a known exon inclusion or mutually exclusive exon event from long-read (e.g. PacBio SequelII) hepatocyte transcriptome data stored as an IsoTools pickle.

## When NOT to use

- Input is short-read RNA-seq data — IsoTools' segment-graph events and beta-binomial differential testing are designed for long-read transcriptome sequencing (LRTS)
- The Transcriptome object has not yet been filtered and indexed (add_qc_metrics, add_filter, make_index) — event retrieval will not reflect quality-controlled transcript support
- The event of interest is a complex or novel splicing pattern not representable as a segment-graph alternative splicing event (exon skipping / mutually exclusive exons)

## Inputs

- Processed IsoTools Transcriptome pickle (example_isotools.pkl) built from a gff3/gtf reference annotation
- Aligned long-read bam files (full-length PacBio SequelII transcripts) per sample, e.g. CTL and VPA hepatocytes
- Gene/event identifiers of interest (e.g. FN1 exon 32 / EDA, SLC39A14 exons 4A/4B)

## Outputs

- Per-condition PSI values for each retrieved event (inclusion vs skipping transcript counts)
- psi_values.csv table of SLC39A14 exon 4B proportion and FN1 EDA PSI in control vs VPA
- Event significance statistics (q-values) for differential exon inclusion

## How to apply

Load the processed IsoTools Transcriptome object (example_isotools.pkl), which must have been built with from_reference on a gff3 annotation, add_sample_from_bam for each sample (e.g. CTL and VPA groups), add_qc_metrics, and make_index. Use the SegmentGraph-based alternative splicing event machinery to retrieve the exon inclusion event at FN1 exon 32 (EDA) and the mutually exclusive exon event at SLC39A14 exons 4A/4B. Quantify PSI per sample group by aggregating transcript support counts for the inclusion versus skipping isoforms within each condition (control vs VPA), so PSI values are directly supported by full-length transcript evidence. Assemble the per-condition PSI values (e.g. FN1 EDA rising from 9% in CTL to 15% under VPA, q = 3.55E-111) into a table such as psi_values.csv. Correctness is judged by reproducing the reported PSI values and the event being flagged as the most significant differential ES event.

## Related tools

- **IsoTools** (Python module providing the Transcriptome object, SegmentGraph-based alternative splicing event definition, and per-group event quantification (PSI) from LRTS bam imports) — https://github.com/MatthiasLienhard/isotools
- **Python** (Execution environment in which the IsoTools Transcriptome is loaded and PSI tables are assembled and written to psi_values.csv)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl')
```

## Evaluation signals

- FN1 EDA PSI in control and VPA groups reproduces the reported values (9% CTL vs 15% VPA) within transcript-count rounding error
- FN1 exon 32 (EDA) is identified as the most significant differential ES event with q-value ≈ 3.55E-111
- SLC39A14 event resolves as mutually exclusive exons 4A/4B with exon 4B proportion computed per condition, not as a generic skipped exon
- PSI values in psi_values.csv correspond to inclusion versus skipping transcript support counts aggregated over samples within each group (CTL vs VPA)
- Retrieval only works on a Transcriptome that passed the full pipeline (add_qc_metrics, add_filter, make_index, save) — verify object provenance

## Limitations

- Differential event detection relies on the beta-binomial statistical model, so low transcript support counts in either group can destabilize PSI estimates
- Default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS; transcripts failing other quality aspects may still contribute to event counts unless additional filters are applied
- Events must be representable as segment-graph alternatives; highly complex splicing patterns may not map cleanly to a single ES/MXE event type

## Evidence

- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] Definition of alternative splicing events based on segment graphs.: "Definition of alternative splicing events based on segment graphs."
- [readme] Import of reference annotation in gff3/gtf format.: "Import of reference annotation in gff3/gtf format."
- [readme] Detection of differential alternative splicing between samples and groups of samples.: "Detection of differential alternative splicing between samples and groups of samples."
- [readme] isoseq.add_qc_metrics('../tests/data/example.fa'); isoseq.save('../tests/data/example_1_isotools.pkl'): "isoseq.add_qc_metrics('../tests/data/example.fa')
isoseq.save('../tests/data/example_1_isotools.pkl')"

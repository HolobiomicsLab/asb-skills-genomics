---
name: alternative-splicing-event-quantification-from-long-read-transcriptomes
description: Use when you have aligned full-length long-read transcripts (bam) for
  sample groups (e.g. CTL vs VPA-treated hepatocytes) and need event-level PSI values
  and differential splicing statistics like q-values for specific exons.
when_to_use_negative:
- Input is short-read RNA-seq data rather than aligned full-length long-read transcripts
  — the method's strength is complex/differential splicing patterns resolved by LRTS
- You only need transcript-level expression quantification without any alternative
  splicing event analysis
- No reference annotation (gff3/gtf) is available, since the workflow builds the Transcriptome
  from_reference before events can be defined
edam_operation: http://edamontology.org/operation_0446
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3320
tools:
- name: IsoTools
  role: Python LRTS analysis framework used to build the Transcriptome object, define
    alternative splicing events from segment graphs, compute per-group PSI, and detect
    differential events via the beta-binomial model
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Runtime language in which the IsoTools pipeline (from_reference, add_sample_from_bam,
    add_qc_metrics, add_filter, event quantification) is scripted
  repo: https://github.com/MatthiasLienhard/isotools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:db03aa2740d8dba9a580955ee0c74aa5bc723d34069d3b564467fdaba4ba143a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_0446
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# alternative-splicing-event-quantification-from-long-read-transcriptomes

## Summary

Quantify per-sample and per-group PSI (percent spliced-in) values for alternative splicing events — such as exon skipping/inclusion and mutually exclusive exons — from long-read transcriptome sequencing (LRTS) data using IsoTools' segment graph-based event machinery and beta-binomial differential testing, as demonstrated on PacBio data from valproic acid-treated human hepatocytes.

## When to use

Use when you have aligned full-length long-read transcripts (bam) for sample groups (e.g. CTL vs VPA-treated hepatocytes) and need event-level PSI values and differential splicing statistics like q-values for specific exons.

## When NOT to use

- Input is short-read RNA-seq data rather than aligned full-length long-read transcripts — the method's strength is complex/differential splicing patterns resolved by LRTS
- You only need transcript-level expression quantification without any alternative splicing event analysis
- No reference annotation (gff3/gtf) is available, since the workflow builds the Transcriptome from_reference before events can be defined

## Inputs

- Bam files of aligned full-length LRTS transcripts (one per sample, e.g. PacBio SequelII)
- Reference annotation in gff3 or gtf format
- Reference genome/transcriptome fasta (for add_qc_metrics)
- Sample-to-group mapping (e.g. CTL vs VPA)

## Outputs

- Per-sample and per-group PSI values for alternative splicing events (e.g. SLC39A14 exon 4B proportion, FN1 exon 32/EDA PSI)
- Differential splicing statistics (q-values) from the beta-binomial model
- Tabular summary such as psi_values.csv
- Persisted Transcriptome pickle (example_isotools.pkl)

## How to apply

Build an IsoTools Transcriptome object with Transcriptome.from_reference on the gff3/gtf reference annotation, then call add_sample_from_bam for each sample (recording sample_name, group, and platform, e.g. 'SequelII'), run add_qc_metrics against the reference fasta, and save the object with save() to a .pkl for fast reuse; optionally apply add_filter before saving. Use IsoTools' SegmentGraph-based alternative splicing event machinery to retrieve the event of interest — e.g. the mutually exclusive exon event at SLC39A14 exons 4A/4B or the exon inclusion event at FN1 exon 32/EDA. Quantify the event with IsoTools' event PSI computation, aggregating transcript support counts for the inclusion versus skipping isoforms within each condition group. Differential significance is assessed with IsoTools' beta-binomial-based statistical approach; in the VPA hepatocyte example, FN1 EDA inclusion rose from 9% (CTL) to 15% (VPA) with q-value = 3.55E−111. Finally assemble the per-condition PSI values into a table (e.g. psi_values.csv).

## Related tools

- **IsoTools** (Python LRTS analysis framework used to build the Transcriptome object, define alternative splicing events from segment graphs, compute per-group PSI, and detect differential events via the beta-binomial model) — https://github.com/MatthiasLienhard/isotools
- **Python** (Runtime language in which the IsoTools pipeline (from_reference, add_sample_from_bam, add_qc_metrics, add_filter, event quantification) is scripted) — https://github.com/MatthiasLienhard/isotools

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('tests/data/example.gff.gz'); isoseq.add_sample_from_bam('example_1_CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('tests/data/example.fa'); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- The Transcriptome object loads without error from the saved .pkl and all expected samples/groups (e.g. CTL, VPA) are present
- The requested event (e.g. FN1 exon 32/EDA exon inclusion, SLC39A14 exons 4A/4B mutually exclusive exons) is retrieved by the segment-graph event machinery
- PSI values fall in the expected per-condition ranges, e.g. FN1 EDA PSI ≈ 9% in CTL vs ≈ 15% in VPA
- Differential testing returns the expected significance, e.g. FN1 EDA q-value ≈ 3.55E−111
- The assembled psi_values.csv contains one row per event with per-condition PSI columns and parses as valid tabular data

## Limitations

- Default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS, so fragment-related artifacts may require explicit add_filter attention
- Event quantification requires transcript support counts per isoform; low-coverage events may yield unstable PSI estimates
- The q-value and PSI conclusions (e.g. 3.55E−111 for FN1 EDA) are specific to the valproic acid hepatocyte PacBio dataset and must be re-estimated for new data

## Evidence

- [abstract] we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events: "we integrate a graph-based method for identifying alternative splicing events and a statistical approach based on the beta-binomial distribution for detecting differential events"
- [intro] Definition of alternative splicing events based on segment graphs: "Definition of alternative splicing events based on segment graphs."
- [methods] isoseq.add_filter()
    isoseq.save('example_isotools.pkl'): "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [readme] Import of LRTS bam files (aligned full length transcripts).: "Import of LRTS bam files (aligned full length transcripts)."
- [readme] isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa, platform='SequelII'): [evidence span withheld — anchor status: external_doc]
- [discussion] Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"

---
name: internal-priming-artifact-assessment
description: Use when you have aligned PacBio full-length transcript (LRTS) BAM files
  plus a genome FASTA and reference annotation, and you need to know what fraction
  of reads per sample group are affected by internal priming, truncation, RTTS, or
  fragments.
when_to_use_negative:
- Input is short-read RNA-seq data — IsoTools QC artifact tagging is designed for
  LRTS bam files of aligned full length transcripts
- You need differential alternative splicing or transcript reconstruction rather than
  QC artifact fractions — use IsoTools' segment-graph splicing event or beta-binomial
  differential analysis functionality instead
- BAM files have not been imported with sample names and groups into a Transcriptome
  object, so per-group aggregation is impossible
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3573
tools:
- name: IsoTools
  role: Python LRTS analysis framework providing add_qc_metrics to compute per-read
    quality control metrics and the artifact tags used for internal priming assessment
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Language used to load the transcriptome pickle via the Transcriptome API,
    extract artifact tags, aggregate per-group fractions, and write qc_artifact_fractions.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:6af535dc3f9355f17cff10be7819f1c6a5c6f28f7e36284a484080a72f40efcf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3573
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# Internal priming artifact assessment

## Summary

Quantifies the fraction of PacBio LRTS reads affected by technical artifacts — internal priming, transcript truncation, RTTS, and FRAGMENTS (reads lacking a reference TSS/PAS) — by computing per-read QC metrics in IsoTools and aggregating per-group artifact tag fractions, e.g. for CTL vs VPA-treated human hepatocyte samples.

## When to use

Use when you have aligned PacBio full-length transcript (LRTS) BAM files plus a genome FASTA and reference annotation, and you need to know what fraction of reads per sample group are affected by internal priming, truncation, RTTS, or fragments.

## When NOT to use

- Input is short-read RNA-seq data — IsoTools QC artifact tagging is designed for LRTS bam files of aligned full length transcripts
- You need differential alternative splicing or transcript reconstruction rather than QC artifact fractions — use IsoTools' segment-graph splicing event or beta-binomial differential analysis functionality instead
- BAM files have not been imported with sample names and groups into a Transcriptome object, so per-group aggregation is impossible

## Inputs

- Filtered IsoTools transcriptome pickle object (.pkl) with sample groups (e.g. CTL, VPA)
- Genome FASTA file (e.g. genome.fa) used for QC metric computation
- Aligned PacBio LRTS BAM files of full-length transcripts imported via add_sample_from_bam
- Reference annotation in gff3/gtf format (for TSS/PAS-based FRAGMENTS tagging)

## Outputs

- qc_artifact_fractions.csv with per-group fraction of reads affected per artifact class (internal priming, truncation, RTTS, FRAGMENTS)
- Per-read QC metrics stored in the transcriptome object after add_qc_metrics

## How to apply

Load the filtered IsoTools transcriptome pickle object with the Transcriptome API in Python, then run isoseq.add_qc_metrics('genome.fa') to compute per-read QC metrics against the genome FASTA. Extract the artifact tags — truncation, RTTS, internal priming, and FRAGMENTS (where FRAGMENTS marks reads that do not use a reference TSS or PAS) — for each sample group such as CTL and VPA. Aggregate per-group read counts per artifact class and compute the fraction of affected reads for each class, writing the result to qc_artifact_fractions.csv. Validate by comparing against reported expectations; for the hepatocyte dataset the total affected reads should reproduce 10.5% (CTL) and 9.3% (VPA), with truncation 4.4%/3.8%, RTTS 3.8%/3.4%, and internal priming 2.6%/2.4% within rounding tolerance.

## Related tools

- **IsoTools** (Python LRTS analysis framework providing add_qc_metrics to compute per-read quality control metrics and the artifact tags used for internal priming assessment) — https://github.com/MatthiasLienhard/isotools
- **Python** (Language used to load the transcriptome pickle via the Transcriptome API, extract artifact tags, aggregate per-group fractions, and write qc_artifact_fractions.csv)

## Examples

```
isoseq.add_qc_metrics('../tests/data/example.fa'); isoseq.save('../tests/data/example_1_isotools.pkl')
```

## Evaluation signals

- Total artifact-affected read fractions reproduce the reported 10.5% (CTL) and 9.3% (VPA) within acceptable rounding tolerance
- Per-class fractions match truncation 4.4%/3.8%, RTTS 3.8%/3.4%, internal priming 2.6%/2.4% for CTL/VPA
- qc_artifact_fractions.csv contains one fraction per artifact class (truncation, RTTS, internal priming, FRAGMENTS) per sample group
- add_qc_metrics ran against the genome FASTA without errors and every read carries artifact tag annotations
- FRAGMENTS-tagged reads are exactly those that do not use a reference TSS or PAS (per the default filter behavior)

## Limitations

- The default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS, so FRAGMENTS fractions depend on annotation quality of TSS/PAS sites
- Fractions are sample- and dataset-specific — the 10.5%/9.3% hepatocyte values are validation targets only for that CTL/VPA PacBio SequelII dataset
- A pickle transcriptome with sample groups must already exist; add_qc_metrics requires the genome FASTA path supplied at runtime

## Evidence

- [other] QC metrics workflow from source card: [evidence span withheld — anchor status: not_found]
- [discussion] Artifact tag definition: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [other] Reported artifact fractions: [evidence span withheld — anchor status: not_found]
- [readme] README QC computation feature: "Computation of quality control metrics."
- [readme] README qc metrics invocation: "isoseq.add_qc_metrics('../tests/data/example.fa')
isoseq.save('../tests/data/example_1_isotools.pkl')"

---
name: transcript-artifact-tagging
description: Use when a filtered IsoTools transcriptome pickle of PacBio LRTS samples
  needs per-read QC against a reference genome, to quantify artifact classes (truncation,
  RTTS, internal priming, FRAGMENTS) per group.
when_to_use_negative:
- Input is raw LRTS BAM files not yet imported into an IsoTools Transcriptome object
  with samples and QC metrics — import and filter first (Transcriptome.from_reference
  / add_sample_from_bam)
- The goal is differential alternative splicing detection or transcriptome reconstruction
  rather than technical artifact quantification
- Data is short-read RNA-seq rather than LRTS full-length transcript alignments
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: IsoTools
  role: Python LRTS analysis framework providing add_qc_metrics for per-read quality
    control and artifact tagging (truncation, RTTS, internal priming, FRAGMENTS) on
    the transcriptome object
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Language in which the IsoTools Transcriptome API is used to load the pickle,
    run QC, aggregate per-group artifact fractions, and write the CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:69480bcbd47a0ec1ddacf91fc7fcdeb40011f4073d87e1a12b669d8abf0cd8ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transcript-artifact-tagging

## Summary

Computes per-read quality control metrics for long-read transcriptome sequencing (LRTS) data with IsoTools and tags technical artifacts (transcript truncation, RTTS, internal priming, FRAGMENTS), then aggregates the fraction of affected reads per sample group. It was used to quantify that 10.5%/9.3% of CTL/VPA hepatocyte PacBio reads carry artifacts.

## When to use

Use when a filtered IsoTools transcriptome pickle of PacBio LRTS samples needs per-read QC against a reference genome, to quantify artifact classes (truncation, RTTS, internal priming, FRAGMENTS) per group.

## When NOT to use

- Input is raw LRTS BAM files not yet imported into an IsoTools Transcriptome object with samples and QC metrics — import and filter first (Transcriptome.from_reference / add_sample_from_bam)
- The goal is differential alternative splicing detection or transcriptome reconstruction rather than technical artifact quantification
- Data is short-read RNA-seq rather than LRTS full-length transcript alignments

## Inputs

- Filtered IsoTools transcriptome pickle object (.pkl, e.g. example_1_isotools.pkl) with sample groups (e.g. CTL, VPA)
- Reference genome FASTA (e.g. genome.fa) against which QC metrics are computed

## Outputs

- qc_artifact_fractions.csv: per-group fraction of reads affected per artifact class (truncation, RTTS, internal priming, FRAGMENTS, total)
- Transcriptome object annotated with per-read QC metrics

## How to apply

Load the filtered IsoTools transcriptome pickle object via the Transcriptome API, then call isoseq.add_qc_metrics('genome.fa') so per-read quality control metrics are computed against the genome FASTA. Extract the artifact tags — truncation, RTTS, internal priming, and FRAGMENTS (the FRAGMENTS tag now only marks reads that do not use a reference TSS or PAS) — for each sample group such as the CTL and VPA hepatocyte samples. Aggregate per-group read counts per artifact class and compute the fraction of affected reads for each class, writing the result to qc_artifact_fractions.csv. The rationale is that PacBio LRTS reads are susceptible to specific library-prep and alignment artifacts that IsoTools can quantify from the same alignment BAMs used for transcriptome reconstruction.

## Related tools

- **IsoTools** (Python LRTS analysis framework providing add_qc_metrics for per-read quality control and artifact tagging (truncation, RTTS, internal priming, FRAGMENTS) on the transcriptome object) — https://github.com/MatthiasLienhard/isotools
- **Python** (Language in which the IsoTools Transcriptome API is used to load the pickle, run QC, aggregate per-group artifact fractions, and write the CSV)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.load('example_isotools.pkl'); isoseq.add_qc_metrics('genome.fa')
```

## Evaluation signals

- qc_artifact_fractions.csv reproduces the reported totals: 10.5% (CTL) and 9.3% (VPA) affected reads, within rounding tolerance
- Per-class fractions match the reference values: truncation 4.4%/3.8%, RTTS 3.8%/3.4%, internal priming 2.6%/2.4% (CTL/VPA)
- Per-group artifact fractions sum to the reported total affected fraction within rounding tolerance
- QC metrics run successfully against the genome FASTA with no missing samples in the CTL/VPA groups

## Limitations

- The default FRAGMENTS filter changed semantics: it now only tags reads that do not use a reference TSS or PAS, so fractions depend on the IsoTools version and filter applied before QC
- Artifact fractions depend on prior filtering of the transcriptome object (isoseq.add_filter()); unfiltered input yields different numbers
- Validated specifically on PacBio SequelII hepatocyte data (CTL vs VPA); behavior on other platforms or tissues is not established here

## Evidence

- [intro] Computation of quality control metrics.: "Computation of quality control metrics."
- [discussion] Default FRAGMENTS filter semantics: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] QC metrics call in README usage example: [evidence span withheld — anchor status: external_doc]
- [abstract] Application context for artifact QC: "we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid"

---
name: long-read-qc-metrics-computation
description: Use when you have aligned PacBio LRTS BAM samples (e.g. CTL/VPA hepatocyte
  groups) already imported into an IsoTools Transcriptome object and need to quantify
  what fraction of reads are affected by technical artifacts such as truncation, RTTS,
  or internal priming.
when_to_use_negative:
- The transcriptome pickle has not yet had QC metrics computed and lacks the prerequisite
  genome FASTA — run add_qc_metrics first rather than attempting to extract artifact
  tags from unannotated reads
- You need differential alternative splicing analysis or transcript reconstruction
  rather than technical artifact quantification — those are separate IsoTools workflows
  (segment-graph splicing events, beta-binomial differential testing)
- Your data is short-read RNA-seq, not aligned long-read (PacBio LRTS) full-length
  transcript BAM files
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: IsoTools
  role: Python framework providing the Transcriptome API and add_qc_metrics for computing
    per-read QC metrics and artifact tags (truncation, RTTS, internal priming, FRAGMENTS)
    on LRTS samples
  repo: https://github.com/MatthiasLienhard/isotools
- name: Python
  role: Runtime language used to drive the IsoTools Transcriptome API for tag extraction,
    per-group aggregation, and CSV export of artifact fractions
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:071d7d617a3dfb34e50883d0c31abfa687b6f159f23fa2e3ab741729518c14eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# long-read-qc-metrics-computation

## Summary

Computes per-read quality control metrics for Long Read Transcriptome Sequencing (LRTS) data with IsoTools' add_qc_metrics against a genome FASTA, then aggregates artifact tags (truncation, RTTS, internal priming, FRAGMENTS) per sample group to quantify the fraction of reads affected by technical artifacts.

## When to use

Use when you have aligned PacBio LRTS BAM samples (e.g. CTL/VPA hepatocyte groups) already imported into an IsoTools Transcriptome object and need to quantify what fraction of reads are affected by technical artifacts such as truncation, RTTS, or internal priming.

## When NOT to use

- The transcriptome pickle has not yet had QC metrics computed and lacks the prerequisite genome FASTA — run add_qc_metrics first rather than attempting to extract artifact tags from unannotated reads
- You need differential alternative splicing analysis or transcript reconstruction rather than technical artifact quantification — those are separate IsoTools workflows (segment-graph splicing events, beta-binomial differential testing)
- Your data is short-read RNA-seq, not aligned long-read (PacBio LRTS) full-length transcript BAM files

## Inputs

- Filtered IsoTools transcriptome pickle file (.pkl) containing imported LRTS BAM samples (e.g. CTL and VPA groups)
- Reference genome FASTA file (e.g. genome.fa)
- Sample group definitions (control vs treatment sample names)

## Outputs

- Per-read QC artifact tags (truncation, RTTS, internal priming, FRAGMENTS) attached to transcripts in the Transcriptome object
- Per-group artifact fraction table, e.g. qc_artifact_fractions.csv, with fraction of affected reads per artifact class

## How to apply

Load the filtered IsoTools transcriptome pickle via the Transcriptome API and run isoseq.add_qc_metrics('genome.fa') to compute per-read QC metrics against the reference genome FASTA. Extract the artifact tags — truncation, RTTS, internal priming, and FRAGMENTS (FRAGMENTS tags reads that do not use a reference TSS or PAS) — for each sample group (e.g. CTL and VPA hepatocytes). Aggregate per-group read counts per artifact class and compute the fraction of affected reads for each class, writing the result to a table such as qc_artifact_fractions.csv. Validate by comparing the computed fractions against reported values within rounding tolerance (e.g. total affected reads 10.5% CTL / 9.3% VPA, truncation 4.4%/3.8%, RTTS 3.8%/3.4%, internal priming 2.6%/2.4%); transcript truncation is expected to be the largest contributor.

## Related tools

- **IsoTools** (Python framework providing the Transcriptome API and add_qc_metrics for computing per-read QC metrics and artifact tags (truncation, RTTS, internal priming, FRAGMENTS) on LRTS samples) — https://github.com/MatthiasLienhard/isotools
- **Python** (Runtime language used to drive the IsoTools Transcriptome API for tag extraction, per-group aggregation, and CSV export of artifact fractions)

## Examples

```
isoseq.add_qc_metrics('genome.fa'); tags = [(t.truncation, t.RTTS, t.internal_priming, t.FRAGMENTS) for t in isoseq.transcripts]
```

## Evaluation signals

- Total artifact-affected read fractions match reported values: 10.5% for CTL and 9.3% for VPA hepatocyte samples within rounding tolerance
- Per-class fractions reproduce truncation 4.4%/3.8%, RTTS 3.8%/3.4%, and internal priming 2.6%/2.4% (CTL/VPA) within acceptable rounding tolerance
- Transcript truncation emerges as the largest artifact contributor in both sample groups
- qc_artifact_fractions.csv is produced with one fraction per artifact class per group and per-group read counts summing to the total sample reads
- FRAGMENTS-tagged reads correspond to transcripts that do not use a reference TSS or PAS, consistent with the IsoTools filter definition

## Limitations

- QC metrics must be computed against a matching reference genome FASTA and a reference annotation; mismatched references will corrupt artifact tagging
- The default filter's FRAGMENTS tag definition (reads not using a reference TSS or PAS) depends on annotation completeness; unannotated true TSS/PAS usage may be misclassified as fragments
- Reported fractions (e.g. 10.5%/9.3%) are specific to the PacBio SequelII hepatocyte CTL/VPA dataset and should be re-validated on new datasets rather than assumed

## Evidence

- [intro] Computation of quality control metrics.: "Computation of quality control metrics."
- [discussion] Default FRAGMENTS tag definition: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] README usage showing add_qc_metrics call: "isoseq.add_qc_metrics('../tests/data/example.fa')
isoseq.save('../tests/data/example_1_isotools.pkl')"
- [intro] IsoTools is a python module for LRTS analysis: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."

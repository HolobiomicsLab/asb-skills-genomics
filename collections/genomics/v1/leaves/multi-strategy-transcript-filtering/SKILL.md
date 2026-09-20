---
name: multi-strategy-transcript-filtering
description: Use when reconstructing transcripts from aligned full-length LRTS BAM
  files (e.g. PacBio hepatocyte CTL/VPA data) and you need to choose a filtering stringency
  by comparing permissive, balanced, and strict add_filter outcomes.
when_to_use_negative:
- Input data are short-read RNA-seq libraries rather than aligned full-length LRTS
  BAM files
- The goal is alternative splicing event detection or differential splicing analysis
  rather than choosing a transcript filtering stringency
- Only one filtering strategy is required and no cross-strategy comparison of counts
  or novel-transcript fractions is needed
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3116
tools:
- name: isotools
  role: Python LRTS analysis framework providing Transcriptome.from_reference, add_sample_from_bam,
    add_qc_metrics, make_index, and add_filter with DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER
    / DEFAULT_REF_TRANSCRIPT_FILTER constants for the multi-strategy filtering comparison
  repo: https://github.com/MatthiasLienhard/isotools
- name: pip
  role: Installs the isotools package from PyPI (python3 -m pip install isotools)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:0cf46600f0c835f5642754f5235cfecbc88454af52f70d656e5f36354af4ba14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - isotools
  - pip
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3116
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# multi-strategy-transcript-filtering

## Summary

A skill for running IsoTools transcriptome reconstruction of long-read transcriptome sequencing (LRTS) data under multiple filtering stringencies (permissive, balanced, strict) using the DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants, then comparing gene/transcript counts and GENCODE-novel fractions across strategies.

## When to use

Use when reconstructing transcripts from aligned full-length LRTS BAM files (e.g. PacBio hepatocyte CTL/VPA data) and you need to choose a filtering stringency by comparing permissive, balanced, and strict add_filter outcomes.

## When NOT to use

- Input data are short-read RNA-seq libraries rather than aligned full-length LRTS BAM files
- The goal is alternative splicing event detection or differential splicing analysis rather than choosing a transcript filtering stringency
- Only one filtering strategy is required and no cross-strategy comparison of counts or novel-transcript fractions is needed

## Inputs

- GENCODE reference transcript annotation in gff3/gtf format
- BAM files of aligned full-length LRTS transcripts (e.g. hepatocyte CTL and VPA samples)
- Genome FASTA for quality control metrics

## Outputs

- Per-strategy gene and transcript counts (targets: 157,134 / 57,698 / 21,582 transcripts in 21,447 / 15,315 / 12,267 genes)
- Fraction of GENCODE-novel transcripts per filtering strategy
- filter_strategy_summary.csv with per-strategy gene/transcript totals

## How to apply

Install isotools with pip, load the GENCODE reference annotation in gff3/gtf format into a Transcriptome via Transcriptome.from_reference, import each hepatocyte LRTS BAM with add_sample_from_bam assigning samples to CTL or VPA groups, then add_qc_metrics against the genome FASTA and make_index. Run add_filter once per strategy using the DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants, recording gene and transcript totals under each. The strict strategy retains transcripts contributing at least 5% of gene expression with at least 7 reads, yielding a target of 21,582 transcripts in 12,267 genes from the combined CTL and VPA hepatocyte reads. For each strategy, compute the fraction of GENCODE-novel transcripts from the biologically motivated transcript classification tags (32% GENCODE-novel under the strict strategy) and assemble a filter_strategy_summary.csv.

## Related tools

- **isotools** (Python LRTS analysis framework providing Transcriptome.from_reference, add_sample_from_bam, add_qc_metrics, make_index, and add_filter with DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER / DEFAULT_REF_TRANSCRIPT_FILTER constants for the multi-strategy filtering comparison) — https://github.com/MatthiasLienhard/isotools
- **pip** (Installs the isotools package from PyPI (python3 -m pip install isotools))

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('gencode.gff.gz'); isoseq.add_sample_from_bam('CTL.bam', sample_name='CTL', group='CTL'); isoseq.add_filter(DEFAULT_TRANSCRIPT_FILTER)
```

## Evaluation signals

- Strict-filter transcript and gene totals match the reported 21,582 transcripts in 12,267 genes
- Balanced-filter totals match 57,698 transcripts in 15,315 genes; permissive-filter totals match 157,134 transcripts in 21,447 genes
- Fraction of GENCODE-novel transcripts computed from classification tags equals 32% under the strict strategy
- filter_strategy_summary.csv rows for all three strategies are populated and internally consistent with the recorded run outputs
- Each sample is correctly assigned to its vehicle (CTL) or valproic-acid-treated (VPA) group after add_sample_from_bam

## Limitations

- Default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS, so fragment handling depends on reference annotation completeness
- Counts are specific to the combined CTL and VPA hepatocyte PacBio LRTS data and the GENCODE reference; other datasets will yield different totals
- Requires quality control metrics computed against the genome FASTA and a built transcript index before add_filter can be meaningfully applied

## Evidence

- [intro] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [intro] Annotation and classification of novel transcripts: "Annotation and classification of novel transcripts with biologically motivated classification scheme."
- [methods] pip installation: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [discussion] FRAGMENTS default filter: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [methods] add_filter usage: "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [readme] BAM import workflow: [evidence span withheld — anchor status: external_doc]

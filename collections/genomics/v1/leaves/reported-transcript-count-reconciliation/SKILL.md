---
name: reported-transcript-count-reconciliation
description: Use when reproducing or validating published IsoTools transcriptome reconstruction
  results from long-read BAM files and a GENCODE reference, and you must confirm gene/transcript
  counts match reported filtering-strategy totals.
when_to_use_negative:
- Your data are short-read RNA-seq rather than long-read transcriptome sequencing
  (LRTS) BAM files of full-length transcripts
- You need differential alternative splicing detection or splicing event identification
  rather than transcript count reconciliation under filtering strategies
- No reference annotation in gff3/gtf format is available for the organism, since
  classification of GENCODE-novel transcripts requires it
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python-based LRTS analysis framework used to import annotation and BAM files,
    reconstruct the transcriptome, and apply DEFAULT_* filter constants to produce
    gene/transcript counts
  repo: https://github.com/MatthiasLienhard/isotools
- name: pip
  role: Package installer used to install the isotools module from PyPI (python3 -m
    pip install isotools)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:e7a38c33c7527331babd5e37cb35925adbc7539d917da1d5e613d167d9c70397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# reported-transcript-count-reconciliation

## Summary

Reproduce IsoTools transcriptome reconstruction of LRTS data and verify that gene and transcript counts under a given filtering strategy match the reported totals (e.g. 21,582 transcripts in 12,267 genes under the strict filter), including the fraction of GENCODE-novel transcripts.

## When to use

Use when reproducing or validating published IsoTools transcriptome reconstruction results from long-read BAM files and a GENCODE reference, and you must confirm gene/transcript counts match reported filtering-strategy totals.

## When NOT to use

- Your data are short-read RNA-seq rather than long-read transcriptome sequencing (LRTS) BAM files of full-length transcripts
- You need differential alternative splicing detection or splicing event identification rather than transcript count reconciliation under filtering strategies
- No reference annotation in gff3/gtf format is available for the organism, since classification of GENCODE-novel transcripts requires it

## Inputs

- GENCODE reference transcript annotation in gff3/gtf format
- Deposited LRTS BAM files of aligned full-length transcripts (hepatocyte CTL and VPA samples)
- Genome FASTA file for quality control metrics

## Outputs

- Per-strategy gene and transcript counts (e.g. 12,267 genes and 21,582 transcripts under the strict filter)
- Fraction of GENCODE-novel transcripts per filtering strategy (e.g. 32% under the strict filter)
- filter_strategy_summary.csv summarizing counts per strategy

## How to apply

Install the isotools Python package (python3 -m pip install isotools) and load the GENCODE reference annotation in gff3/gtf format into a Transcriptome object with Transcriptome.from_reference. Import the aligned full-length LRTS BAM files (here hepatocyte CTL and VPA PacBio SequelII samples) with add_sample_from_bam, assigning each sample to its vehicle or valproic-acid group, then run add_qc_metrics against the genome FASTA and make_index. Run add_filter once per strategy using the DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants; the strict filter keeps transcripts contributing at least 5% of gene expression with at least 7 reads, with targets of 157,134 / 57,698 / 21,582 transcripts in 21,447 / 15,315 / 12,267 genes. Compute the fraction of GENCODE-novel transcripts per strategy from the biologically motivated classification tags (32% under the strict filter), assemble the per-strategy summary into filter_strategy_summary.csv, and verify the totals match the reported numbers.

## Related tools

- **IsoTools** (Python-based LRTS analysis framework used to import annotation and BAM files, reconstruct the transcriptome, and apply DEFAULT_* filter constants to produce gene/transcript counts) — https://github.com/MatthiasLienhard/isotools
- **pip** (Package installer used to install the isotools module from PyPI (python3 -m pip install isotools))

## Examples

```
python3 -m pip install isotools; isoseq = Transcriptome.from_reference('gencode.gtf'); isoseq.add_sample_from_bam('ctl.bam', sample_name='CTL', group='CTL', platform='SequelII')
```

## Evaluation signals

- Under the strict filtering strategy (at least 5% of gene expression, at least 7 reads), gene and transcript totals equal 12,267 genes and 21,582 transcripts for the combined CTL and VPA hepatocyte reconstruction
- The permissive and balanced strategies yield 157,134 transcripts in 21,447 genes and 57,698 transcripts in 15,315 genes, respectively
- 32% of strict-filter transcripts are classified as not found in GENCODE using the biologically motivated transcript classification tags
- The imported samples carry correct group labels (CTL vehicle vs VPA valproic acid) and platform (SequelII), and the transcript index is built via make_index before filtering
- filter_strategy_summary.csv row totals reconcile with the reported target counts for all three strategies

## Limitations

- Counts depend on the specific hepatocyte CTL/VPA PacBio SequelII data and GENCODE annotation; other datasets will not match the reported totals
- The default FRAGMENTS filter only tags reads that do not use a reference TSS or PAS, so filter behavior changed across IsoTools versions and affects reconciled counts
- Filtering requires prior computation of QC metrics (add_qc_metrics with the genome FASTA) and index construction; skipping these steps changes the filter results

## Evidence

- [results] IsoTools reconstruction of the combined CTL and VPA hepatocyte long reads yields 12 267 genes and 21 582 transcripts, of which 32% are not found in GENCODE: [evidence span withheld — anchor status: not_found]
- [intro] Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format. Computation of quality control metrics.: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format. Computation of quality control metrics."
- [methods] The package can be installed with pip: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [discussion] Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [readme] isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa, platform='SequelII'): [evidence span withheld — anchor status: external_doc]

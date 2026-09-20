---
name: lrt-bam-import-against-gencode-annotation
description: 'Use when starting IsoTools-based LRTS analysis: you have BAM files of
  aligned full-length transcripts (e.g. PacBio SequelII hepatocyte CTL/VPA samples)
  and a GENCODE gff3/gtf reference to load before QC and filtering.'
when_to_use_negative:
- Input reads are short-read RNA-seq alignments rather than full-length LRTS BAM files
- No reference annotation in gff3/gtf format is available and a de novo-only reconstruction
  is required without GENCODE-based classification
- The goal is differential splicing analysis on an already imported and indexed Transcriptome;
  use the downstream alternative-splicing workflow instead
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: isotools
  role: Python module whose Transcriptome.from_reference (gff3/gtf) and add_sample_from_bam
    methods import LRTS BAM data against the reference annotation
  repo: https://github.com/MatthiasLienhard/isotools
- name: pip
  role: Package manager used to install isotools from PyPI
  repo: https://pypi.org/project/isotools
- name: Python
  role: Runtime environment for the isotools import workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:609c33536b721f7dd6bc169e6a723f1803d66ec4bdd28f7c46121f2dc3bab2c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - isotools
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# lrt-bam-import-against-gencode-annotation

## Summary

Import aligned full-length long-read transcriptome sequencing (LRTS) BAM files into an IsoTools Transcriptome object on top of a GENCODE gff3/gtf reference annotation, so that novel transcripts can be annotated, quality-controlled, and filtered downstream.

## When to use

Use when starting IsoTools-based LRTS analysis: you have BAM files of aligned full-length transcripts (e.g. PacBio SequelII hepatocyte CTL/VPA samples) and a GENCODE gff3/gtf reference to load before QC and filtering.

## When NOT to use

- Input reads are short-read RNA-seq alignments rather than full-length LRTS BAM files
- No reference annotation in gff3/gtf format is available and a de novo-only reconstruction is required without GENCODE-based classification
- The goal is differential splicing analysis on an already imported and indexed Transcriptome; use the downstream alternative-splicing workflow instead

## Inputs

- BAM files of aligned full-length LRTS transcripts (e.g. deposited hepatocyte CTL and VPA PacBio SequelII BAMs)
- Reference transcript annotation in gff3/gtf format (GENCODE)
- Genome FASTA file for quality-control metrics

## Outputs

- IsoTools Transcriptome object indexed via make_index, holding per-sample LRTS reads annotated against GENCODE
- Saved isotools pickle (.pkl) for fast re-import
- QC metrics per sample against the genome FASTA

## How to apply

Install the isotools Python package (python3 -m pip install isotools) and build a Transcriptome object from the GENCODE reference annotation in gff3/gtf format with Transcriptome.from_reference. For each sample, call add_sample_from_bam with the deposited aligned full-length transcript BAM, supplying a sample_name, its group (e.g. CTL vehicle or VPA valproic-acid-treated), and the platform (e.g. SequelII). Then compute quality-control metrics against the genome FASTA with add_qc_metrics and build the transcript index with make_index so later add_filter runs (DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, DEFAULT_REF_TRANSCRIPT_FILTER) can recover expected totals such as 21,582 transcripts in 12,267 genes under the strict strategy. Saving the imported object as a pickle with save() allows faster re-import in subsequent sessions. The rationale is that the gff3/gtf reference anchors the biologically motivated transcript classification used to tag GENCODE-novel transcripts (32% of strict-filter transcripts here).

## Related tools

- **isotools** (Python module whose Transcriptome.from_reference (gff3/gtf) and add_sample_from_bam methods import LRTS BAM data against the reference annotation) — https://github.com/MatthiasLienhard/isotools
- **pip** (Package manager used to install isotools from PyPI) — https://pypi.org/project/isotools
- **Python** (Runtime environment for the isotools import workflow)

## Examples

```
isoseq = Transcriptome.from_reference('gencode.gff.gz'); [isoseq.add_sample_from_bam(f'hepatocyte_{sa}.bam', sample_name=sa, group=sa, platform='SequelII') for sa in ('CTL','VPA')]
```

## Evaluation signals

- Transcriptome object is created without errors from the GENCODE gff3/gtf and each BAM import assigns the correct sample_name and group (CTL vs VPA)
- add_qc_metrics runs successfully against the genome FASTA and the saved pickle reloads cleanly
- Downstream permissive/balanced/strict add_filter totals match reported targets (e.g. 21,582 transcripts in 12,267 genes under strict filtering)
- Biologically motivated classification tags identify the expected fraction of GENCODE-novel transcripts (32% under the strict strategy)

## Limitations

- Requires a reference annotation in gff3/gtf format; the workflow is annotation-driven rather than purely de novo
- BAM files must contain aligned full-length transcripts; partial or fragmented alignments are not the intended input
- Renamed samples/groups must match the experimental design (vehicle vs valproic-acid-treated) or downstream group comparisons will be incorrect

## Evidence

- [intro] Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format.: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."
- [intro] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [methods] The package can be installed with pip: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [readme] README usage snippet showing from_reference and add_sample_from_bam with CTL/VPA samples: "isoseq = Transcriptome.from_reference('tests/data/example.gff.gz')
for sa in ('CTL', 'VPA'):
    isoseq.add_sample_from_bam(f'../tests/data/example_1_{sa}.bam', sample_name=sa, group=sa,"

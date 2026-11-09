---
name: novel-transcript-fraction-quantification
description: Decision support for choosing a filtering stringency (permissive/balanced/strict)
  when reconstructing transcripts from LRTS BAM files against GENCODE, and for reporting
  how many reconstructed transcripts are novel versus annotated.
when_to_use_negative:
- Input is short-read RNA-seq data rather than full-length LRTS BAM alignments — IsoTools
  is designed for LRTS
- No reference annotation in gff3/gtf format is available for novel-transcript classification
  against GENCODE
- The goal is differential alternative splicing event detection rather than transcript-level
  filtering and novelty quantification
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: isotools
  role: Python framework used to import BAMs and GENCODE annotation, apply DEFAULT_*_FILTER
    constants via add_filter, and classify transcripts as GENCODE-novel
  repo: https://github.com/MatthiasLienhard/isotools
- name: pip
  role: Installs the isotools package
- name: Python
  role: Runtime language in which the Transcriptome workflow and filter strategies
    are executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:39e52bc4f90aee6aedcb982eb033b57cca1baf3dce36a510359a547749916577
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - IsoTools
  - Python
  - isotools
  - pip
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# novel-transcript-fraction-quantification

## Summary

Quantifies the fraction of GENCODE-novel transcripts recovered by IsoTools transcriptome reconstruction from hepatocyte PacBio LRTS data, stratified by filtering strategy. Under the strict filter (transcripts contributing ≥5% of gene expression with ≥7 reads), reconstruction yields 21,582 transcripts in 12,267 genes, of which 32% are absent from GENCODE.

## When to use

Decision support for choosing a filtering stringency (permissive/balanced/strict) when reconstructing transcripts from LRTS BAM files against GENCODE, and for reporting how many reconstructed transcripts are novel versus annotated.

## When NOT to use

- Input is short-read RNA-seq data rather than full-length LRTS BAM alignments — IsoTools is designed for LRTS
- No reference annotation in gff3/gtf format is available for novel-transcript classification against GENCODE
- The goal is differential alternative splicing event detection rather than transcript-level filtering and novelty quantification

## Inputs

- BAM files of aligned full-length long-read transcripts (hepatocyte PacBio LRTS, CTL and VPA groups)
- GENCODE reference annotation in gff3/gtf format
- Genome FASTA for QC metrics

## Outputs

- Per-strategy gene and transcript counts (targets: 157,134 / 57,698 / 21,582 transcripts in 21,447 / 15,315 / 12,267 genes)
- Fraction of GENCODE-novel transcripts per strategy (32% under strict filtering)
- filter_strategy_summary.csv

## How to apply

Install isotools via pip and load the GENCODE gff3/gtf reference into a Transcriptome object with Transcriptome.from_reference. Import the aligned full-length transcript BAM files with add_sample_from_bam, assigning each hepatocyte sample to its control (CTL) or valproic-acid-treated (VPA) group, then compute QC metrics against the genome FASTA (add_qc_metrics) and build the transcript index (make_index). Run add_filter once per strategy using the DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants; the strict strategy requires transcripts to contribute at least 5% of gene expression with at least 7 reads. Classify transcripts with the biologically motivated classification tags and compute the fraction whose tags indicate they are not present in GENCODE per strategy. The strict filter is the conservative choice: it recovers 21,582 transcripts in 12,267 genes (versus 157,134/21,447 permissive and 57,698/15,315 balanced), with 32% GENCODE-novel transcripts.

## Related tools

- **isotools** (Python framework used to import BAMs and GENCODE annotation, apply DEFAULT_*_FILTER constants via add_filter, and classify transcripts as GENCODE-novel) — https://github.com/MatthiasLienhard/isotools
- **pip** (Installs the isotools package)
- **Python** (Runtime language in which the Transcriptome workflow and filter strategies are executed)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('gencode.gtf'); isoseq.add_sample_from_bam('CTL.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('genome.fa')
```

## Evaluation signals

- Gene/transcript totals per strategy match reported targets: 157,134 transcripts / 21,447 genes (permissive), 57,698 / 15,315 (balanced), 21,582 / 12,267 (strict)
- Strict-filter GENCODE-novel fraction equals ~32% of the 21,582 reconstructed transcripts
- All filter strategies were run with DEFAULT_GENE_FILTER, DEFAULT_TRANSCRIPT_FILTER, and DEFAULT_REF_TRANSCRIPT_FILTER constants
- Each BAM sample is correctly assigned to its CTL or VPA group before index building
- filter_strategy_summary.csv contains a row per strategy with consistent gene and transcript counts

## Limitations

- Filtering stringency trades sensitivity for specificity: the strict filter retains only ~14% of the permissive transcript count (21,582 vs 157,134), so novel-transcript fractions are strongly strategy-dependent
- Novelty is judged relative to GENCODE, so novel fractions are annotation-version dependent

## Evidence

- [intro] Annotation and classification of novel transcripts with biologically motivated classification scheme.: "Annotation and classification of novel transcripts with biologically motivated classification scheme."
- [readme] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [methods] The package can be installed with pip: "The package can be installed with pip:

.. code-block:: bash

    python3 -m pip install isotools"
- [intro] Import of LRTS bam files and reference annotation: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."

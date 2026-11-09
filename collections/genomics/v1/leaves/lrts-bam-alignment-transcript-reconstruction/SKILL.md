---
name: lrts-bam-alignment-transcript-reconstruction
description: Use when you have LRTS BAM alignments (e.g. PacBio SequelII full-length
  reads) plus a gff3/gtf reference annotation and need to reconstruct and quantify
  transcripts, comparing tool-dependent or filter-dependent transcript recovery.
when_to_use_negative:
- Input is short-read RNA-seq data rather than long-read alignments of full-length
  transcripts — IsoTools is a module for LRTS analysis
- You only need alternative splicing event detection or differential splicing between
  sample groups rather than transcriptome reconstruction and tool comparison
- You lack a reference annotation in gff3/gtf format and cannot supply one, since
  annotation and classification of novel transcripts depend on it
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: IsoTools
  role: Python framework used to import LRTS BAM alignments and gff3/gtf annotation,
    reconstruct the transcriptome under permissive and balanced filter configurations,
    and quantify transcripts
  repo: https://github.com/MatthiasLienhard/isotools
- name: TALON
  role: Run on the same BAM alignment with default filtering parameters to produce
    a comparison transcript set
- name: FLAIR
  role: Run on the same BAM alignment with default filtering parameters to produce
    a comparison transcript set
- name: IsoQuant
  role: Run on the same BAM alignment with default filtering parameters to produce
    a comparison transcript set
- name: Bambu
  role: Run on the same BAM alignment with default filtering parameters to produce
    a comparison transcript set
- name: Python
  role: Language in which IsoTools is implemented and in which the overlap analysis
    is scripted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:9f8f7888fc8e64e2a5db9c3efa69b5197c1052d5788b13de30e8bdf7035c8d46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MatthiasLienhard/isotools
  tools:
  - Bambu
  - FLAIR
  - IsoQuant
  - IsoTools
  - Python
  - TALON
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# lrts-bam-alignment-transcript-reconstruction

## Summary

Reconstruct transcriptomes from long-read transcriptome sequencing (LRTS) BAM alignments using IsoTools, a Python framework that imports aligned full-length transcripts plus gff3/gtf reference annotation and performs transcriptome reconstruction with configurable filtering (e.g. permissive vs. balanced configurations) to control novel transcript recovery.

## When to use

Use when you have LRTS BAM alignments (e.g. PacBio SequelII full-length reads) plus a gff3/gtf reference annotation and need to reconstruct and quantify transcripts, comparing tool-dependent or filter-dependent transcript recovery.

## When NOT to use

- Input is short-read RNA-seq data rather than long-read alignments of full-length transcripts — IsoTools is a module for LRTS analysis
- You only need alternative splicing event detection or differential splicing between sample groups rather than transcriptome reconstruction and tool comparison
- You lack a reference annotation in gff3/gtf format and cannot supply one, since annotation and classification of novel transcripts depend on it

## Inputs

- LRTS BAM file of aligned full-length transcripts (e.g. PacBio SequelII hepatocyte alignments)
- Reference annotation in gff3 or gtf format
- Reference genome FASTA (for QC metrics, e.g. tests/data/example.fa)
- Reconstructed transcript sets from comparison tools (TALON, FLAIR, IsoQuant, Bambu)

## Outputs

- IsoTools Transcriptome object saved as .pkl (filtered or unfiltered)
- Reconstructed and quantified transcript sets per filtering configuration
- Tool transcript overlap table (e.g. tool_transcript_overlap_table.tsv) with five-tool intersection and per-tool unique counts

## How to apply

Load the LRTS BAM alignment and the reference annotation (gff3/gtf) into an IsoTools Transcriptome object using Transcriptome.from_reference and add_sample_from_bam, computing QC metrics and saving as a pkl for faster re-import. Run comparison tools such as TALON, FLAIR, IsoQuant, and Bambu on the same alignment with their default filtering parameters to produce reconstructed transcript sets. Run IsoTools reconstruction twice: once with the permissive filter configuration (add_filter using DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER settings) to maximize novel transcript recovery, and once with the balanced configuration, under which FRAGMENTS-class reads not using a reference TSS or PAS are excluded. Parse all five transcript sets and compute multi-way overlap sets (five-tool intersection plus per-tool unique sets); on the human hepatocyte PacBio dataset this yields 20,642 transcripts shared by all five tools, with uniquely-IsoTools transcripts dropping from 72,733 (permissive) to 5,564 (balanced, <10% unique).

## Related tools

- **IsoTools** (Python framework used to import LRTS BAM alignments and gff3/gtf annotation, reconstruct the transcriptome under permissive and balanced filter configurations, and quantify transcripts) — https://github.com/MatthiasLienhard/isotools
- **TALON** (Run on the same BAM alignment with default filtering parameters to produce a comparison transcript set)
- **FLAIR** (Run on the same BAM alignment with default filtering parameters to produce a comparison transcript set)
- **IsoQuant** (Run on the same BAM alignment with default filtering parameters to produce a comparison transcript set)
- **Bambu** (Run on the same BAM alignment with default filtering parameters to produce a comparison transcript set)
- **Python** (Language in which IsoTools is implemented and in which the overlap analysis is scripted)

## Examples

```
from isotools import Transcriptome; isoseq = Transcriptome.from_reference('annotation.gff.gz'); isoseq.add_sample_from_bam('hepatocyte.bam', sample_name='CTL', group='CTL', platform='SequelII'); isoseq.add_qc_metrics('genome.fa'); isoseq.add_filter(); isoseq.save('isotools.pkl')
```

## Evaluation signals

- The overlap table shows 20,642 transcripts shared by all five tools (five-tool intersection)
- Uniquely-IsoTools transcript count is 72,733 under the permissive filter configuration
- Uniquely-IsoTools transcript count drops to 5,564 (less than 10% unique) under the balanced filter configuration that excludes FRAGMENTS-class reads not using a reference TSS or PAS
- The Transcriptome object imports without error from BAM and gff3/gtf and can be saved and re-imported as a pkl file for faster loading
- QC metrics have been computed (add_qc_metrics) before downstream filtering and reconstruction

## Limitations

- The number of identified (and uniquely recovered) transcripts is strongly dependent on the chosen filtering criteria — permissive filtering inflates unique recovery (72,733) while balanced filtering drops it below 10% (5,564)
- Default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS, so filter choice directly changes which reads survive
- Results were demonstrated on PacBio LRTS data of human hepatocytes treated with the HDAC inhibitor valproic acid; transferability to other platforms/datasets is not established by this card

## Evidence

- [intro] IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis.: "IsoTools is a python module for Long Read Transcriptome Sequencing (LRTS) analysis."
- [intro] Import of LRTS bam files (aligned full length transcripts).: "Import of LRTS bam files (aligned full length transcripts). Import of reference annotation in gff3/gtf format."
- [discussion] Default filter of FRAGMENTS only tags reads that do not use a reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [methods] isoseq.add_filter() then save pkl: "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [abstract] We introduce IsoTools, a Python-based LRTS analysis framework: "We introduce IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts."
- [abstract] we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid: "we applied IsoTools to PacBio LRTS data of human hepatocytes treated with the histone deacetylase inhibitor valproic acid"
- [readme] add_sample_from_bam usage from README: [evidence span withheld — anchor status: external_doc]

---
name: transcript-filter-stringency-tuning
description: Use when reconstructing transcriptomes from LRTS BAM alignments with
  IsoTools and you must decide how filter stringency (e.g. FRAGMENTS-class reads lacking
  reference TSS/PAS) affects the number of uniquely recovered transcripts.
when_to_use_negative:
- Input is short-read RNA-seq data — IsoTools is designed for Long Read Transcriptome
  Sequencing (LRTS) analysis of aligned full-length transcripts.
- Reads are unaligned; the workflow starts from a BAM of aligned full-length transcripts
  plus a reference annotation, not raw signal or FASTQ.
- The goal is alternative splicing event detection or differential splicing rather
  than transcriptome reconstruction stringency assessment.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: IsoTools
  role: Python framework used to import the LRTS BAM and gff3/gtf annotation, apply
    permissive and balanced filter configurations via add_filter, and produce filtered
    transcriptome objects
  repo: https://github.com/MatthiasLienhard/isotools
- name: TALON
  role: Comparison LRTS reconstruction tool run on the same BAM with default filtering
    to benchmark unique/shared transcript recovery
- name: FLAIR
  role: Comparison LRTS reconstruction tool run on the same BAM with default filtering
- name: IsoQuant
  role: Comparison LRTS reconstruction tool run on the same BAM with default filtering
- name: Bambu
  role: Comparison LRTS reconstruction tool run on the same BAM with default filtering
- name: Python
  role: Language in which IsoTools is implemented and in which the filtering and overlap-tabulation
    scripts are written
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad364
    title: 'IsoTools: a flexible workflow for long-read transcriptome sequencing analysis'
  source_content_hash: sha256:fbf101f1823f97585f044ab39fc39e62cd2268199eb779dc7b5331ad47cee315
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# transcript-filter-stringency-tuning

## Summary

Tune IsoTools transcript filtering stringency when reconstructing transcriptomes from long-read RNA-seq (LRTS) alignments, comparing permissive versus balanced filter configurations to trade off novel transcript recovery against specificity. The skill is grounded in the IsoTools paper's hepatocyte benchmark where permissive filtering uniquely recovered 72,733 transcripts but balanced filtering reduced unique transcripts to 5,564.

## When to use

Use when reconstructing transcriptomes from LRTS BAM alignments with IsoTools and you must decide how filter stringency (e.g. FRAGMENTS-class reads lacking reference TSS/PAS) affects the number of uniquely recovered transcripts.

## When NOT to use

- Input is short-read RNA-seq data — IsoTools is designed for Long Read Transcriptome Sequencing (LRTS) analysis of aligned full-length transcripts.
- Reads are unaligned; the workflow starts from a BAM of aligned full-length transcripts plus a reference annotation, not raw signal or FASTQ.
- The goal is alternative splicing event detection or differential splicing rather than transcriptome reconstruction stringency assessment.

## Inputs

- LRTS BAM file of aligned full-length transcripts (e.g. PacBio SequelII hepatocyte data)
- Reference annotation in gff3 or gtf format
- IsoTools filter configuration (DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER exports)
- Transcript sets reconstructed by comparison tools (TALON, FLAIR, IsoQuant, Bambu) on the same alignment

## Outputs

- Permissively filtered IsoTools transcriptome object (pkl)
- Balanced-filter IsoTools transcriptome object with FRAGMENTS-class reads lacking reference TSS/PAS excluded
- Multi-way transcript overlap table (tool_transcript_overlap_table.tsv) with five-tool intersection and per-tool unique counts

## How to apply

Load the LRTS BAM alignment and reference annotation (gff3/gtf) into an IsoTools Transcriptome object, then run transcriptome reconstruction twice: once with the permissive filter configuration applied via add_filter using the exported DEFAULT_GENE_FILTER / DEFAULT_TRANSCRIPT_FILTER settings to maximize novel transcript recovery, and once with the balanced configuration, under which FRAGMENTS-class reads not using a reference TSS or PAS are excluded. Save each filtered transcriptome object (e.g. as a pkl) so both stringency variants can be compared downstream. Tabulate overlap with other LRTS reconstruction tools (TALON, FLAIR, IsoQuant, Bambu run on the same BAM with default filters) to quantify how many transcripts are shared by all five tools versus uniquely recovered per tool. The rationale is that the number of identified and uniquely recovered transcripts is strongly dependent on the chosen filtering criteria, so reporting both stringency regimes makes the reconstruction comparison interpretable.

## Related tools

- **IsoTools** (Python framework used to import the LRTS BAM and gff3/gtf annotation, apply permissive and balanced filter configurations via add_filter, and produce filtered transcriptome objects) — https://github.com/MatthiasLienhard/isotools
- **TALON** (Comparison LRTS reconstruction tool run on the same BAM with default filtering to benchmark unique/shared transcript recovery)
- **FLAIR** (Comparison LRTS reconstruction tool run on the same BAM with default filtering)
- **IsoQuant** (Comparison LRTS reconstruction tool run on the same BAM with default filtering)
- **Bambu** (Comparison LRTS reconstruction tool run on the same BAM with default filtering)
- **Python** (Language in which IsoTools is implemented and in which the filtering and overlap-tabulation scripts are written)

## Examples

```
from isotools import Transcriptome; isoseq.add_filter(); isoseq.save('example_isotools.pkl')
```

## Evaluation signals

- The five-tool overlap table shows 20,642 transcripts shared by all five tools.
- Uniquely-IsoTools transcript counts drop from 72,733 under the permissive filter to 5,564 under the balanced filter.
- With balanced filtering, the proportion of uniquely identified IsoTools transcripts falls to less than 10%.
- Both filtered transcriptome objects save and reload successfully (e.g. isoseq.save('example_isotools.pkl') followed by re-import).
- tool_transcript_overlap_table.tsv is produced with five-tool intersection plus per-tool unique sets.

## Limitations

- Uniquely recovered transcript counts are highly sensitive to filter stringency, so permissive-filter claims of novelty can be inflated (72,733 unique transcripts drop to 5,564 with balanced filtering).
- The balanced filter excludes FRAGMENTS-class reads that do not use a reference TSS or PAS, which removes reads that may still represent genuine novel transcripts.
- Comparisons depend on comparison tools being run with their default filtering parameters on the same alignment; changing those defaults changes the overlap baseline.

## Evidence

- [results] With permissive filtering, 72 733 transcripts are uniquely recovered by IsoTools; this proportion of uniquely identified transcripts drops to less than 10% (5564 unique transcripts) with the balanced filtering strategy.: [evidence span withheld — anchor status: not_found]
- [discussion] Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS: "Default filter of FRAGMENTS now only tags reads that do not use a reference TSS or PAS"
- [methods] isoseq.add_filter()
    isoseq.save('example_isotools.pkl'): "isoseq.add_filter()
    isoseq.save('example_isotools.pkl')"
- [abstract] We introduce IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts.: "We introduce IsoTools, a Python-based LRTS analysis framework that offers a wide range of functionality for transcriptome reconstruction and quantification of transcripts."
- [readme] Import of LRTS bam files (aligned full length transcripts).
* Import of reference annotation in gff3/gtf format.: "Import of LRTS bam files (aligned full length transcripts).
* Import of reference annotation in gff3/gtf format."

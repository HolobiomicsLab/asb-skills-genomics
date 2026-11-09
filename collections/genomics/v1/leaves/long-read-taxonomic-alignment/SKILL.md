---
name: long-read-taxonomic-alignment
description: Use when quality-controlled ARG-containing long reads (Nanopore or PacBio)
  need candidate species assignments and alignment scores against a GTDB-derived reference,
  as the host-collection step of an ARG profiling pipeline.
when_to_use_negative:
- 'Short-read data: this skill relies on minimap2''s map-ont preset and long-read
  overlapping; second-generation short-read surveillance methods linking ARGs to hosts
  are explicitly noted as limited'
- 'De novo identification of ARG genomic contexts and hosts in new, understudied environments:
  reference-dependent alignment cannot classify unseen species or novel plasmid syntenies
  — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods
  may be better suited'
- 'ARGs originating from point mutations: read-based profiling cannot reliably distinguish
  SNPs from sequencing errors, so such ARGs are excluded from the database and will
  not be resolved'
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Argo
  role: Pipeline whose host-collection step performs this alignment-based candidate
    species labeling and aggregation
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: Performs the base-level '-cx map-ont' alignment of ARG-containing reads to
    the ARG-tailored GTDB .mmi index with secondary alignments retained
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:ef6f8b6aa488218b23523d4394bd0032e40671c4d501dda03698f3325340d71c
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  repo_url: https://github.com/xinehc/argo
  tools:
  - Argo
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# long-read-taxonomic-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collects host information for ARG-containing long reads by mapping them with minimap2 base-level alignment against an ARG-tailored GTDB reference (up to 10,000 bp ARG-containing genomic regions), retaining abundant secondary alignments so per-read candidate species labels and alignment scores can be aggregated downstream.

## When to use

Use when quality-controlled ARG-containing long reads (Nanopore or PacBio) need candidate species assignments and alignment scores against a GTDB-derived reference, as the host-collection step of an ARG profiling pipeline.

## When NOT to use

- Short-read data: this skill relies on minimap2's map-ont preset and long-read overlapping; second-generation short-read surveillance methods linking ARGs to hosts are explicitly noted as limited
- De novo identification of ARG genomic contexts and hosts in new, understudied environments: reference-dependent alignment cannot classify unseen species or novel plasmid syntenies — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods may be better suited
- ARGs originating from point mutations: read-based profiling cannot reliably distinguish SNPs from sequencing errors, so such ARGs are excluded from the database and will not be resolved

## Inputs

- Quality-controlled long reads (Nanopore or PacBio, e.g., FASTA/FASTQ such as example.fa.gz)
- Indexed .mmi minimap2 reference built from ARG-tailored GTDB nucleotide sequences (10,000 bp ARG-containing regions from GTDB R09-RS220 assemblies)

## Outputs

- Per-read candidate-species/alignment-score table as a tab-delimited TSV (candidate_species_alignments.tsv) with read ID, species, AS score, and full GTDB lineage string
- Validated read-species pairs (ai,j = 1) aggregated into candidate species sets s_j for downstream set-cover labeling

## How to apply

First index the ARG-tailored GTDB nucleotide sequences (10,000 bp ARG-containing regions extracted from 596,663 GTDB R09-RS220 assemblies, 1,394,827 unique clustered sequences) with 'minimap2 -x map-ont -d' to build .mmi indices. Then align the quality-controlled ARG-containing reads with base-level alignment using '-cx map-ont -f 0 -N 2147483647 -p 0.9' so that abundant secondary alignments are retained rather than suppressed. Sort all reported alignments by alignment score (AS) in descending order and record only the top alignment for each read-species combination. Mark a read-species pair ai,j = 1 (valid) only when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', and aggregate valid pairs into candidate species sets s_j = {i | ai,j = 1}. Finally, write a tab-delimited per-read table with read ID, species, AS score, and the full GTDB lineage string (e.g., Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus).

## Related tools

- **Argo** (Pipeline whose host-collection step performs this alignment-based candidate species labeling and aggregation) — https://github.com/xinehc/argo
- **minimap2** (Performs the base-level '-cx map-ont' alignment of ARG-containing reads to the ARG-tailored GTDB .mmi index with secondary alignments retained)

## Examples

```
minimap2 -cx map-ont -f 0 -N 2147483647 -p 0.9 database/gtdb.mmi example.fa.gz > alignments.paf
```

## Evaluation signals

- candidate_species_alignments.tsv exists and parses as valid TSV with present lineage and alignment-score fields
- Lineage strings resolve to full GTDB lineages consistent with the deposited example JSON (e.g., containing 'Staphylococcus aureus')
- Each read-species combination has exactly one (top-scoring) alignment recorded after AS-descending sorting
- Retained read-species pairs satisfy the validity rule max(AS/0.995, AS+50) > max_j' AS_i,j'
- Number of unique reference sequences aligns with expectation (1,394,827 clustered sequences from 596,663 GTDB R09-RS220 assemblies)

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- ARGs of species below one genome copy (1x) are marked 'unclassified' by default, so extremely rare species are missed
- Insufficient coverage hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering

## Evidence

- [other] minimap2 base-level alignment to an ARG-tailored reference taxonomy database derived from ARG-containing genomic regions (up to 10,000 bp) of GTDB assemblies: [evidence span withheld — anchor status: not_found]
- [other] retain secondary alignments: [evidence span withheld — anchor status: not_found]
- [other] AS validity filter: [evidence span withheld — anchor status: not_found]

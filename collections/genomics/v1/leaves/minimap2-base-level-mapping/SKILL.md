---
name: minimap2-base-level-mapping
description: Use when long (ONT/PacBio) reads must be aligned to a large nucleotide
  reference (e.g., an ARG-tailored GTDB database) and per-read alignment scores plus
  candidate taxonomic labels are needed, not just a single hit.
when_to_use_negative:
- Reads are short second-generation reads — Argo's host-tracking is designed for long
  reads, where overlapping provides host-identification accuracy gains over per-read
  classifiers like Kraken2/Centrifuge
- Only a single best-hit taxonomic assignment is needed and secondary alignments would
  be wasted effort — the -f 0 -N 2147483647 settings specifically retain many secondary
  hits for set-cover aggregation
- De novo identification of ARG genomic contexts in understudied environments is the
  goal — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk)
  methods are recommended there
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: minimap2
  role: Builds .mmi indices of the GTDB-derived reference and performs base-level
    alignment ('-cx map-ont -f 0 -N 2147483647 -p 0.9') of ARG-containing reads to
    generate candidate species labels with AS scores
  repo: https://github.com/xinehc/argo
- name: Argo
  role: Wraps the minimap2 mapping step as the host-information-collection stage of
    its pipeline and aggregates the resulting alignments into candidate species sets
    via set covering
  repo: https://github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:b3ddf43501d9a09dac86796711d445a2b82c8c66c212974f7c3ef5d99849d9cf
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
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# minimap2-base-level-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs base-level (non-overlapping) alignment of quality-controlled long reads to a reference database with minimap2, retaining abundant secondary alignments so that per-read candidate species labels and alignment scores (AS) can be collected for downstream taxonomic aggregation in Argo.

## When to use

Use when long (ONT/PacBio) reads must be aligned to a large nucleotide reference (e.g., an ARG-tailored GTDB database) and per-read alignment scores plus candidate taxonomic labels are needed, not just a single best hit.

## When NOT to use

- Reads are short second-generation reads — Argo's host-tracking is designed for long reads, where overlapping provides host-identification accuracy gains over per-read classifiers like Kraken2/Centrifuge
- Only a single best-hit taxonomic assignment is needed and secondary alignments would be wasted effort — the -f 0 -N 2147483647 settings specifically retain many secondary hits for set-cover aggregation
- De novo identification of ARG genomic contexts in understudied environments is the goal — assembly-based (metaFlye + Kraken2) or binning-based (SemiBin2 + GTDB-Tk) methods are recommended there

## Inputs

- Quality-controlled long reads (Nanopore or PacBio), e.g., FASTA/FASTQ such as example.fa.gz
- Indexed .mmi reference files built from GTDB nucleotide sequences (10,000 bp ARG-containing regions from 596,663 GTDB R09-RS220 assemblies, 1,394,827 unique sequences after clustering)
- minimap2 with preset map-ont

## Outputs

- Tab-delimited per-read candidate-species/alignment-score table (candidate_species_alignments.tsv) with read ID, species, AS score, and full lineage string (e.g., Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus)
- Validated read-species pairs aggregated into candidate species sets s_j = {i | ai,j = 1}

## How to apply

First build reference indices from the GTDB-derived FASTA files using 'minimap2 -x map-ont -d' to create .mmi files (done per-file, parallelizable across CPUs as shown in the Argo README). Then align quality-controlled reads with base-level alignment using '-cx map-ont -f 0 -N 2147483647 -p 0.9'; setting -f 0 to 0 and -N to its maximum retains all secondary alignments, which is essential because host information for ARG-containing reads may be supported by non-primary hits. Sort reported alignments by alignment score (AS) descending and keep only the top alignment per read-species pair. Mark a read-species pair ai,j = 1 (valid) only when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', then aggregate valid pairs into candidate species sets s_j = {i | ai,j = 1}. Emit a tab-delimited table of read ID, species, AS score, and full GTDB lineage string.

## Related tools

- **minimap2** (Builds .mmi indices of the GTDB-derived reference and performs base-level alignment ('-cx map-ont -f 0 -N 2147483647 -p 0.9') of ARG-containing reads to generate candidate species labels with AS scores) — https://github.com/xinehc/argo
- **Argo** (Wraps the minimap2 mapping step as the host-information-collection stage of its pipeline and aggregates the resulting alignments into candidate species sets via set covering) — https://github.com/xinehc/argo

## Examples

```
minimap2 -cx map-ont -f 0 -N 2147483647 -p 0.9 database/gtdb.mmi reads.fa.gz
```

## Evaluation signals

- candidate_species_alignments.tsv exists and parses as TSV with present lineage and alignment-score (AS) fields
- Lineage strings resolve to full GTDB lineages consistent with the deposited example JSON (e.g., containing 'Staphylococcus aureus')
- Secondary-alignment retention is confirmed: -N is set to 2147483647 and -f to 0 so that abundant secondary alignments appear in the alignment output
- Filtering is applied: only pairs satisfying max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j' are marked valid and aggregated into candidate species sets
- Only the top-scoring alignment per read-species combination is recorded after sorting alignments by AS in descending order

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Species present below ~1x genome copy default to 'unclassified'; insufficient coverage leads to inadequate read overlaps, less effective read clustering, and unstable normalization, weakening the mapping-based host assignments

## Evidence

- [methods] In the host-collection step, ARG-containing reads are mapped with minimap2's base-level alignment to an ARG-tailored reference taxonomy database derived from ARG-containing genomic regions (up to 10,000 bp) of GTDB assemblies: [evidence span withheld — anchor status: not_found]
- [methods] Sort and record top per read-species alignment: [evidence span withheld — anchor status: not_found]

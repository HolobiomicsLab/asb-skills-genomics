---
name: candidate-species-set-aggregation
description: 'Use when reproducing or auditing Argo''s host-collection stage: you
  have quality-controlled, ARG-containing Nanopore/PacBio reads and need per-read
  candidate species labels with AS scores aggregated into per-species candidate sets.'
when_to_use_negative:
- Input is short-read metagenomic data (second-generation reads) — this step depends
  on long-read base-level alignment and the read-overlapping strategy, not short-read
  taxonomic classifiers like Kraken2 or Centrifuge
- Reads have not first been identified as ARG-containing — the step operates on ARG-containing
  reads only; ARG detection/filtering happens upstream in Argo's pipeline
- You need per-read taxonomic assignments rather than cluster-level candidate sets
  — Argo deliberately labels read clusters collectively via set cover instead of individual
  reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Argo
  role: 'Implements the host-collection step: runs the minimap2 alignment, applies
    the AS-threshold filter max(AS/0.995, AS+50) > max cross-species AS, and aggregates
    valid pairs into candidate species sets'
  repo: github.com/xinehc/argo
- name: minimap2
  role: Performs the base-level alignment of ARG-containing reads to the ARG-tailored
    GTDB index ('-cx map-ont -f 0 -N 2147483647 -p 0.9') and builds the .mmi reference
    index ('-x map-ont -d')
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:9b962a1a8986a8796afeb84923a52e61786d6782ed877b007a6065ee0c84318e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinehc/argo
  tools:
  - Argo
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# candidate-species-set-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the second (host-information collection) step of Argo: ARG-containing long reads are base-level aligned with minimap2 to an ARG-tailored GTDB reference (10,000 bp ARG-containing regions), and alignment-score-filtered read-species pairs are aggregated into candidate species sets for later set-cover labeling.

## When to use

Use when reproducing or auditing Argo's host-collection stage: you have quality-controlled, ARG-containing Nanopore/PacBio reads and need per-read candidate species labels with AS scores aggregated into per-species candidate sets.

## When NOT to use

- Input is short-read metagenomic data (second-generation reads) — this step depends on long-read base-level alignment and the read-overlapping strategy, not short-read taxonomic classifiers like Kraken2 or Centrifuge
- Reads have not first been identified as ARG-containing — the step operates on ARG-containing reads only; ARG detection/filtering happens upstream in Argo's pipeline
- You need per-read taxonomic assignments rather than cluster-level candidate sets — Argo deliberately labels read clusters collectively via set cover instead of individual reads

## Inputs

- Quality-controlled ARG-containing long reads (FASTA/FASTQ, e.g., the deposited example.fa.gz; Nanopore or PacBio)
- Indexed ARG-tailored GTDB database (.mmi) built from 10,000 bp ARG-containing genomic regions of GTDB R09-RS220 assemblies (1,394,827 unique clustered sequences)

## Outputs

- candidate_species_alignments.tsv: tab-delimited per-read table with read ID, species, AS score, and full GTDB lineage string
- Candidate species sets s_j = {i | ai,j = 1} mapping each candidate species to its valid reads, consumed by the downstream set-cover cluster labeling

## How to apply

Index the ARG-tailored GTDB nucleotide database — 10,000 bp ARG-containing regions extracted from 596,663 GTDB R09-RS220 assemblies, giving 1,394,827 unique sequences after clustering — with 'minimap2 -x map-ont -d' to build .mmi indices. Align the quality-controlled ARG-containing reads (e.g., the deposited example.fa.gz) using minimap2 base-level alignment with '-cx map-ont -f 0 -N 2147483647 -p 0.9', where '-f 0 -N 2147483647' retains abundant secondary alignments that a default mapping would discard. Sort all reported alignments by alignment score (AS) descending and keep only the top alignment for each read-species combination. Mark ai,j = 1 (valid) only when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', then aggregate valid read-species pairs into candidate species sets s_j = {i | ai,j = 1}. Emit a tab-delimited per-read table (candidate_species_alignments.tsv) containing read ID, species, AS score, and the full lineage string (e.g., Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus).

## Related tools

- **Argo** (Implements the host-collection step: runs the minimap2 alignment, applies the AS-threshold filter max(AS/0.995, AS+50) > max cross-species AS, and aggregates valid pairs into candidate species sets) — github.com/xinehc/argo
- **minimap2** (Performs the base-level alignment of ARG-containing reads to the ARG-tailored GTDB index ('-cx map-ont -f 0 -N 2147483647 -p 0.9') and builds the .mmi reference index ('-x map-ont -d'))

## Examples

```
minimap2 -cx map-ont -f 0 -N 2147483647 -p 0.9 database/gtdb.mmi example.fa.gz > alignments.paf
```

## Evaluation signals

- candidate_species_alignments.tsv exists, parses as TSV, and carries populated species, AS-score, and lineage fields
- Lineage strings resolve to full GTDB lineages consistent with the deposited example JSON (e.g., containing 'Staphylococcus aureus')
- Every retained (read, species) pair satisfies the validity rule max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j' for that read
- After AS-descending sorting, exactly one (top) alignment is recorded per read-species combination, with no duplicate read-species rows
- The '-f 0 -N 2147483647 -p 0.9' settings were used so abundant secondary alignments are present before filtering (filtering with default minimap2 secondary limits would under-report candidate species)

## Limitations

- Argo is inherently reference-based: unseen species or novel plasmid syntenies absent from the ARG-tailored GTDB database may be misclassified or left unassigned
- The 10,000 bp region cap compresses GTDB (from more than 1.5 Tbp to less than 15 Gbp), so host labeling is restricted to evidence within ARG-containing windows rather than whole genomes
- The aggregation step provides raw candidate sets only; it cannot by itself resolve species labels — final labeling requires the downstream read-cluster set-cover step and fails under insufficient coverage with inadequate read overlaps

## Evidence

- [methods] marking ai,j = 1 (valid) only when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', and aggregate the valid read-species pairs into candidate species sets s_j = {i | ai,j = 1}: [evidence span withheld — anchor status: not_found]

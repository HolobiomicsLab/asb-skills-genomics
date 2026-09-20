---
name: alignment-score-based-secondary-alignment-filtering
description: Use when mapping quality-controlled ARG-containing ONT/PacBio reads to
  an ARG-tailored GTDB reference and you need abundant secondary alignments plus per-species
  alignment-score filtering to build candidate host species sets.
when_to_use_negative:
- Input is short-read data — Argo takes quality-controlled long reads (Nanopore or
  PacBio) and the -x map-ont preset is tuned for ONT reads.
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied
  environments — the article states assembly-based (metaFlye + Kraken2) and binning-based
  (SemiBin2 + GTDB-Tk) methods may be better suited there.
- ARGs arise from point mutations — read-based profiling cannot reliably distinguish
  SNPs from sequencing errors, so such ARGs are excluded from Argo's database.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: performs base-level alignment of ARG-containing reads to the ARG-tailored
    GTDB .mmi index with '-cx map-ont -f 0 -N 2147483647 -p 0.9' so abundant secondary
    alignments and AS scores are reported
- name: Argo
  role: 'orchestrates the host-collection step: sorts alignments by AS, applies the
    max(AS/0.995, AS+50) validity rule per read-species pair, aggregates valid pairs
    into candidate species sets, and emits the TSV with lineage strings'
  repo: github.com/xinehc/argo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:f0f36f003ca82f2a59f98db75df481770972c46dab2eaf71c702203d0ac3642d
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
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# alignment-score-based-secondary-alignment-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Argo's host-collection step: ARG-containing long reads are base-level aligned with minimap2 against an ARG-tailored GTDB database (10,000 bp ARG-containing regions from 596,663 GTDB R09-RS220 assemblies, 1,394,827 clustered sequences), then alignment scores (AS) are used to retain only valid read-species pairs that are aggregated into candidate species sets for set-cover labeling.

## When to use

Use when mapping quality-controlled ARG-containing ONT/PacBio reads to an ARG-tailored GTDB reference and you need abundant secondary alignments plus per-species alignment-score filtering to build candidate host species sets.

## When NOT to use

- Input is short-read data — Argo takes quality-controlled long reads (Nanopore or PacBio) and the -x map-ont preset is tuned for ONT reads.
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — the article states assembly-based (metaFlye + Kraken2) and binning-based (SemiBin2 + GTDB-Tk) methods may be better suited there.
- ARGs arise from point mutations — read-based profiling cannot reliably distinguish SNPs from sequencing errors, so such ARGs are excluded from Argo's database.

## Inputs

- quality-controlled ARG-containing long reads (e.g., example.fa.gz Nanopore/PacBio FASTA/FASTQ)
- ARG-tailored GTDB reference database (up to 10,000 bp ARG-containing regions from 596,663 GTDB R09-RS220 assemblies; 1,394,827 unique sequences after clustering)
- prebuilt minimap2 .mmi index of the ARG-tailored GTDB sequences

## Outputs

- tab-delimited candidate_species_alignments.tsv with read ID, species, AS alignment score, and full GTDB lineage string per read-species pair
- candidate species sets s_j = {i | ai,j = 1} of valid read-species pairs for downstream set-cover host assignment

## How to apply

First index the ARG-tailored GTDB nucleotide sequences with 'minimap2 -x map-ont -d' to build .mmi reference indices. Then align the quality-controlled ARG-containing reads with minimap2 base-level alignment using '-cx map-ont -f 0 -N 2147483647 -p 0.9', where the near-unbounded -N 2147483647 retains abundant secondary alignments because ARGs transferred horizontally can occur in multiple species. Sort all reported alignments by alignment score (AS) in descending order and keep only the top alignment for each read-species combination. Mark a pair ai,j = 1 (valid) only when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', i.e., the species' best AS must exceed the best score over all other species after either a ratio or additive-slack correction, then aggregate valid pairs into candidate species sets s_j = {i | ai,j = 1}. Write the per-read candidate-species table as a tab-delimited file with read ID, species, AS score, and the full GTDB lineage string (e.g., Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus), which Argo later consolidates on a read-cluster basis via set cover.

## Related tools

- **minimap2** (performs base-level alignment of ARG-containing reads to the ARG-tailored GTDB .mmi index with '-cx map-ont -f 0 -N 2147483647 -p 0.9' so abundant secondary alignments and AS scores are reported)
- **Argo** (orchestrates the host-collection step: sorts alignments by AS, applies the max(AS/0.995, AS+50) validity rule per read-species pair, aggregates valid pairs into candidate species sets, and emits the TSV with lineage strings) — github.com/xinehc/argo

## Examples

```
minimap2 -cx map-ont -f 0 -N 2147483647 -p 0.9 database/gtdb.mmi example.fa.gz > candidate_species_alignments.paf
```

## Evaluation signals

- candidate_species_alignments.tsv exists in TSV format with present lineage and alignment-score fields
- lineage strings resolve to full GTDB lineages consistent with the deposited example JSON (e.g., containing 'Staphylococcus aureus' with the chain Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus)
- for every retained pair, max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j' holds; any pair failing the ratio-or-additive-slack test is dropped
- after AS-descending sort, exactly one alignment (the top-scoring) is recorded per read-species combination, while multiple species per read are retained
- multiple candidate species per read appear for ARG-carrying reads, reflecting the retained secondary alignments (-N 2147483647, -f 0)

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies.
- Insufficient coverage hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering; species below 1× genome copy are marked 'unclassified' by default.
- Highly similar ARGs are clustered (e.g., blaOXA-1 and blaOXA-1042 grouped as blaOXA), lowering resolution for ORF-based annotation or detailed sequence typing.

## Evidence

- [readme] uses minimap2's base-level alignment with GTDB to obtain raw species assignments and consolidates these assignments on a *read cluster* basis: "It uses minimap2's base-level alignment with GTDB to obtain raw species assignments and consolidates these assignments on a *read cluster* basis"
- [intro] ARGs prone to HGT appear in multiple locations across species: "as ARGs are prone to horizontal gene transfer, they may appear in multiple genetic locations (e.g., plasmids or chromosomes) across different species"
- [abstract] Argo assigns labels to clusters of reads collectively: "Argo significantly enhances the resolution of ARG detection by assigning taxonomic labels collectively to clusters of reads, rather than to individual reads."
- [discussion] aggregated evidence from local read clusters aids species labeling: "aids in correct species labeling with aggregated evidence collected from local read clusters"

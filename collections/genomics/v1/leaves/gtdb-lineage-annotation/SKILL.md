---
name: gtdb-lineage-annotation
description: Use when long metagenomic reads flagged as ARG-containing need species-level
  host candidates via alignment to an ARG-tailored GTDB reference, with lineage-annotated,
  score-ranked per-read output.
when_to_use_negative:
- Input is raw metagenomic reads that have not been screened for ARG content — ARG-free
  reads should be filtered first to bypass unnecessary alignments
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied
  environments — Argo is inherently reference-based and may misclassify unseen species
  or novel plasmid syntenies
- Short-read data requiring taxonomic classifiers like Kraken2 or Centrifuge — this
  skill is specific to minimap2 base-level alignment of long reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Argo
  role: Pipeline whose second step indexes the ARG-tailored GTDB database and generates
    per-read candidate species labels with alignment scores
  repo: https://github.com/xinehc/argo
- name: minimap2
  role: Performs base-level alignment ('-cx map-ont -f 0 -N 2147483647 -p 0.9') of
    ARG-containing reads to the indexed GTDB reference and builds .mmi indices with
    'map-ont -d'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-57088-y
    title: Species-resolved profiling of antibiotic resistance genes in complex metagenomes
      through long-read overlapping with Argo
  source_content_hash: sha256:14ff2b148c968b8142a1208ae04113b84ef98137cd0429f891aeede6ee3664d2
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# gtdb-lineage-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs Argo's host-information collection step: quality-controlled ARG-containing long reads are aligned with minimap2 base-level alignment against an ARG-tailored GTDB nucleotide database (10,000 bp ARG-containing regions), producing per-read candidate species labels with alignment scores and full GTDB lineage strings.

## When to use

Use when long metagenomic reads flagged as ARG-containing need species-level host candidates via alignment to an ARG-tailored GTDB reference, with lineage-annotated, score-ranked per-read output.

## When NOT to use

- Input is raw metagenomic reads that have not been screened for ARG content — ARG-free reads should be filtered first to bypass unnecessary alignments
- The goal is de novo identification of ARG genomic contexts and hosts in new, understudied environments — Argo is inherently reference-based and may misclassify unseen species or novel plasmid syntenies
- Short-read data requiring taxonomic classifiers like Kraken2 or Centrifuge — this skill is specific to minimap2 base-level alignment of long reads

## Inputs

- Quality-controlled ARG-containing long reads (Nanopore or PacBio, e.g., example.fa.gz)
- ARG-tailored GTDB nucleotide database (10,000 bp ARG-containing regions from GTDB R09-RS220 assemblies)
- minimap2 .mmi index of the ARG-tailored GTDB sequences

## Outputs

- candidate_species_alignments.tsv: per-read tab-delimited table of read ID, species, AS alignment score, and full GTDB lineage string
- Candidate species sets s_j aggregating valid read-species pairs for downstream set-cover species labeling

## How to apply

First build .mmi indices from the ARG-tailored GTDB nucleotide sequences (10,000 bp ARG-containing regions extracted from 596,663 GTDB R09-RS220 assemblies, clustered to 1,394,827 unique sequences) using 'minimap2 -x map-ont -d'. Then align the ARG-containing reads with minimap2 base-level alignment using '-cx map-ont -f 0 -N 2147483647 -p 0.9' so that abundant secondary alignments are retained rather than suppressed. Sort all reported alignments by alignment score (AS) in descending order, keeping only the top alignment per read-species pair. Apply the validity filter ai,j = 1 when max(AS_i,j / 0.995, AS_i,j + 50) > max_j' AS_i,j', and aggregate valid read-species pairs into candidate species sets s_j = {i | ai,j = 1}. Write the per-read table (read ID, species, AS score, full lineage string such as Bacteria;Bacillota;Bacilli;Staphylococcales;Staphylococcaceae;Staphylococcus;Staphylococcus aureus) as a tab-delimited file for downstream cluster-level set-cover labeling.

## Related tools

- **Argo** (Pipeline whose second step indexes the ARG-tailored GTDB database and generates per-read candidate species labels with alignment scores) — https://github.com/xinehc/argo
- **minimap2** (Performs base-level alignment ('-cx map-ont -f 0 -N 2147483647 -p 0.9') of ARG-containing reads to the indexed GTDB reference and builds .mmi indices with 'map-ont -d')

## Examples

```
minimap2 -cx map-ont -f 0 -N 2147483647 -p 0.9 database/gtdb.mmi example.fa.gz > candidate_species_alignments.paf
```

## Evaluation signals

- candidate_species_alignments.tsv exists and parses as tab-delimited TSV with present read ID, species, alignment-score (AS), and lineage fields
- Lineage strings resolve to full GTDB lineages (seven ranks, e.g., containing 'Staphylococcus aureus') consistent with the deposited example JSON
- Secondary alignments are retained: with '-N 2147483647 -p 0.9', multiple read-species pairs per read are reported, not a single best hit
- Validity filter check: a pair ai,j = 1 only when max(AS_i,j / 0.995, AS_i,j + 50) exceeds the maximum AS across species for that read
- Every reported species traceable to the 1,394,827 clustered ARG-containing GTDB regions rather than full genomes

## Limitations

- Argo is inherently reference-based and may face classification issues with unseen species or novel plasmid syntenies
- Insufficient coverage hinders accurate taxonomic classification due to inadequate read overlaps and less effective read clustering
- Argo considers ARGs of species with less than one genome copy (1×) as 'unclassified' by default, so this step cannot resolve extremely rare species

## Evidence

- [methods] minimap2 base-level alignment to ARG-tailored GTDB: [evidence span withheld — anchor status: not_found]
- [methods] secondary-alignment retention parameters: [evidence span withheld — anchor status: not_found]

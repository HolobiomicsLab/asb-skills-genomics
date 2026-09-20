---
name: host-read-mapping-removal
description: Use when processing untargeted Oxford Nanopore FASTQ reads from clinical
  (e.g. Lassa, Ebola, Dengue) or animal samples in ViMOP and host or reagent contamination
  must be removed after Centrifuge taxonomic filtering.
when_to_use_negative:
- The goal is to remove broad non-viral taxa (e.g. bacterial reads from a fecal sample)
  by taxonomic confidence — use --centrifuge_do_filter with --centrifuge_filter_min_score
  150 instead
- The goal is to enrich reads for specific target viruses (e.g. --targets "MARV,EBOV,FILO")
  — this is a positive selection step, not contaminant removal
- Reads have not yet been trimmed (adapter/primer removal) or Centrifuge-classified
  in the ViMOP workflow order
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: ViMOP
  role: Pipeline implementing the host/contaminant removal step, exposing --contamination_filters
    and contamination.yaml configuration
  repo: https://github.com/opr-group-bnitm/vimop
- name: minimap2
  role: Maps surviving reads against the host/reagent reference sequence sets so that
    mapped (contaminant) reads can be extracted and discarded
- name: Nextflow
  role: Workflow engine that executes the ViMOP read-cleaning stage
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:fb7968c2b86b204fe76850d6f4537f07474066e38011e6fb0a34d5da99010f89
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - Nextflow
  - ViMOP
  - minimap2
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# host-read-mapping-removal

## Summary

A read-cleaning step of the ViMOP nanopore pipeline in which reads surviving Centrifuge-based non-viral filtering are mapped against host/reagent reference sets (via minimap2 on references defined in contamination.yaml) and mapped reads are extracted and discarded to produce the cleaned read set.

## When to use

Use when processing untargeted Oxford Nanopore FASTQ reads from clinical (e.g. Lassa, Ebola, Dengue) or animal samples in ViMOP and host or reagent contamination must be removed after Centrifuge taxonomic filtering.

## When NOT to use

- The goal is to remove broad non-viral taxa (e.g. bacterial reads from a fecal sample) by taxonomic confidence — use --centrifuge_do_filter with --centrifuge_filter_min_score 150 instead
- The goal is to enrich reads for specific target viruses (e.g. --targets "MARV,EBOV,FILO") — this is a positive selection step, not contaminant removal
- Reads have not yet been trimmed (adapter/primer removal) or Centrifuge-classified in the ViMOP workflow order

## Inputs

- Cleaned post-Centrifuge FASTQ reads (per sample or barcode01/barcode02 subdirectories passed via --fastq)
- contamination.yaml config defining reference sets (e.g. human_dna, mouse, mastomys, reagent)
- Host/reagent reference FASTA files such as GCF_000001405.39_GRCh38.p13_genomic.fna.gz
- --contamination_filters option selecting which reference sets to filter against

## Outputs

- Cleaned read set (FASTQ) with no reads mapping to the selected host/reagent references
- reads.tsv per-sample read summary in the sample's tables directory

## How to apply

Run this step after Centrifuge classification has removed non-viral reads (score >= --centrifuge_filter_min_score 150). Map the surviving reads with minimap2 against reference sequence sets selected via --contamination_filters (default human reads; e.g. "reagent,mouse" adds reagent and Mus musculus sets such as GCF_000001635.8_GRCm38.p6_genomic.fna.gz, while the human set is GCF_000001405.39_GRCh38.p13_genomic.fna.gz). These reference sets are defined with key values in the contamination.yaml config of the ViMOP_DB. Extract reads that map to the contaminant references and discard them, retaining the cleaned read set; the mapping-based extraction catches host and reagent sequences that Centrifuge classification may miss. Finally write the per-sample read summary (reads.tsv) to the tables directory.

## Related tools

- **ViMOP** (Pipeline implementing the host/contaminant removal step, exposing --contamination_filters and contamination.yaml configuration) — https://github.com/opr-group-bnitm/vimop
- **minimap2** (Maps surviving reads against the host/reagent reference sequence sets so that mapped (contaminant) reads can be extracted and discarded)
- **Nextflow** (Workflow engine that executes the ViMOP read-cleaning stage) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq reads/ --contamination_filters "reagent,mouse"
```

## Evaluation signals

- reads.tsv exists in the sample's tables directory
- Cleaned read set contains no reads mapping to the selected host/reagent references (e.g. human GCF_000001405.39_GRCh38.p13_genomic.fna.gz when the default filter is used)
- Cleaned read set also contains no reads classified as non-viral by Centrifuge with score >= 150
- Changing --contamination_filters (e.g. to "reagent,mouse") changes which reference sets (GRCm38 mouse, reagent) reads are extracted against
- Per-sample output is produced separately for barcode01/barcode02 subdirectories of --fastq

## Limitations

- Centrifuge classification accuracy is limited, so some false positives may be removed in the upstream taxonomic filtering that precedes this mapping step
- Reference sets are limited to those defined in contamination.yaml; hosts or contaminants not present in the reference FASTAs will not be removed
- ViMOP assemblies may contain inaccuracies for large DNA viruses with repetitive regions (e.g. mpox), so downstream outputs should be reviewed (e.g. alignment .bam files)

## Evidence

- [methods] Host and contaminant reads are removed by mapping them against reference sequences and extracting those that map.: "Host and contaminant reads are removed by mapping them against reference sequences and extracting those that map."
- [readme] contamination filters option: "Use the option `--contamination_filters "reagent,mouse"` for example to remove mouse and reagent reads (the default is human reads)."
- [readme] contamination.yaml defines reference sets: "The database config file (contamination.yaml) defines the reference sets and the key values assigned to them."
- [readme] centrifuge filter parameter: "Deactivate this with `--centrifuge_do_filter false` and use `--centrifuge_filter_min_score 150` to set the minimum level of confidence (e.g. 150 in our example)."

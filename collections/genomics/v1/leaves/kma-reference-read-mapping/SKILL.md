---
name: kma-reference-read-mapping
description: Use when trimmed FASTQ read files from nanopore sequencing must be mapped
  against a size-filtered viral reference database so that per-sample viral consensus
  genomes can be assembled and later taxonomically labelled.
when_to_use_negative:
- Input consists only of FASTA files — VirDetector accepts these from the alignment
  step onwards, so KMA read mapping of reads does not apply.
- Only a taxonomic label is needed without consensus genome reconstruction — Kraken2
  classification alone covers that goal.
- A reference sequence for reference-based assembly is already supplied via params.ref
  — the reference-based assembly path is used instead of KMA-based mapping of reads
  to the database.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: KMA
  role: maps trimmed per-sample reads against the size-filtered viral reference database
- name: VirDetector
  role: pipeline orchestrating KMA mapping, per-sample consensus assembly, Kraken2
    taxonomy and Krona visualisation
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: workflow engine executing the VirDetector metagenomics module (main.nf)
- name: Kraken2
  role: assigns a taxonomic label to each assembled genome against the same viral
    reference database
- name: Krona
  role: renders one interactive taxonomic chart per sample from the classification
    output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:6f297de7dc178a88cb181b2b3c006d818492474e818c31509285a6d1696923d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - KMA
  - Kraken2
  - Krona
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kma-reference-read-mapping

## Summary

Reference-guided read mapping of trimmed nanopore FASTQ reads against the VirDetector size-filtered viral reference database (database/sequences.fasta, built via make_dbs.py) using KMA, producing per-sample mapped reads that are assembled into viral consensus genomes and subsequently taxonomically classified with Kraken2.

## When to use

Use when trimmed FASTQ read files from nanopore sequencing must be mapped against a size-filtered viral reference database so that per-sample viral consensus genomes can be assembled and later taxonomically labelled.

## When NOT to use

- Input consists only of FASTA files — VirDetector accepts these from the alignment step onwards, so KMA read mapping of reads does not apply.
- Only a taxonomic label is needed without consensus genome reconstruction — Kraken2 classification alone covers that goal.
- A reference sequence for reference-based assembly is already supplied via params.ref — the reference-based assembly path is used instead of KMA-based mapping of reads to the database.

## Inputs

- trimmed FASTQ read files per sample (.fastq, .fq, .fastq.gz, .fq.gz) from upstream preprocessing
- size-filtered viral reference database built from database/sequences.fasta via make_dbs.py
- sample grouping: files or subdirectories in the input directory, optionally filtered by a metadata file

## Outputs

- KMA-mapped reads per sample
- per-sample assembled viral consensus sequences
- single summarising Kraken2 taxonomic classification report
- one Krona interactive taxonomic chart per sample

## How to apply

Load the trimmed FASTQ files produced during upstream preprocessing and treat each file or each subdirectory in the input folder as one separate sample, matching VirDetector's input convention. Map the reads against the size-filtered viral reference database built from database/sequences.fasta via make_dbs.py using KMA, running within the Nextflow-based VirDetector pipeline. Assemble the KMA-mapped reads into per-sample viral consensus sequences. The database covers segmented viruses as independent L, M, and S entries, so mapping output feeds directly into Kraken2 taxonomy assignment per assembled genome, whose per-sample results are compiled into one summarising taxonomic classification report and one Krona chart per sample.

## Related tools

- **KMA** (maps trimmed per-sample reads against the size-filtered viral reference database)
- **VirDetector** (pipeline orchestrating KMA mapping, per-sample consensus assembly, Kraken2 taxonomy and Krona visualisation) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (workflow engine executing the VirDetector metagenomics module (main.nf))
- **Kraken2** (assigns a taxonomic label to each assembled genome against the same viral reference database)
- **Krona** (renders one interactive taxonomic chart per sample from the classification output)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- Each file or subdirectory in the input folder was treated as exactly one sample in the mapping step.
- KMA-mapped reads yielded per-sample viral consensus sequences rather than a single pooled assembly.
- Kraken2 output covers the segmented viruses represented as independent L, M, and S entries in the VirDetector database.
- One summarising taxonomic classification report plus exactly one Krona chart per sample is produced from the classification output.
- Mapping used the size-filtered database built from database/sequences.fasta via make_dbs.py, not raw unfiltered references.

## Limitations

- If a metadata file is specified in nextflow.config, only the samples defined in that file are passed on to subsequent processing steps, which silently drops unmapped samples.
- Taxonomy assignment is limited to the viruses contained in the VirDetector database, including segmented viruses as independent L, M, and S entries.
- The pipeline was developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation, and basecalling is excluded from the standard test execution.

## Evidence

- [other] KMA mapping enables Kraken2 labelling of assembled genomes: [evidence span withheld — anchor status: not_found]
- [readme] Sample grouping convention for the input directory: "Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample."
- [abstract] Pipeline performs taxonomic classification after preprocessing: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically"

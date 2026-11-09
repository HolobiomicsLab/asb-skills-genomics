---
name: kraken2-taxonomic-assignment
description: Use when FASTQ reads from nanopore sequencing samples must be taxonomically
  classified against VirDetector's curated viral reference database (database/sequences.fasta)
  as part of the metagenomics module, e.g. for virus surveillance without a known
  reference genome.
when_to_use_negative:
- The sample already has a known reference sequence and the goal is reference-based
  genome reconstruction and phylogenetic analysis rather than de novo metagenomic
  classification
- Only assembled FASTA contigs are available as input without the reads needed for
  the KMA mapping and assembly step
- Taxonomic classification against a comprehensive database (e.g. full RefSeq/NCBI
  nt) is required — VirDetector classifies only against its own curated size-filtered
  viral database of segmented viruses
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: VirDetector
  role: Nextflow-based pipeline that orchestrates the metagenomics module, defines
    samples from the input directory, and supplies the viral reference database
  repo: https://github.com/NLKaiser/VirDetector
- name: KMA
  role: Maps trimmed FASTQ reads against the size-filtered viral reference database
    before assembly
- name: Kraken2
  role: Assigns a taxonomic label to each assembled viral genome against the same
    VirDetector reference database
- name: Krona
  role: Renders one interactive taxonomic chart per sample from the Kraken2 classification
    output
- name: Nextflow
  role: Workflow engine executing the VirDetector pipeline steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:6bdd9c7ca9ec833f2a52e4501d7bdffafca12b86c5a11c011ec4c416ab9e06f0
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kraken2-taxonomic-assignment

## Summary

Assign taxonomic labels to assembled viral genomes within VirDetector's metagenomics step, where KMA-mapped reads are assembled into per-sample viral consensus sequences and Kraken2 classifies each genome against VirDetector's size-filtered viral reference database, with results summarised in one report and visualised per sample with Krona.

## When to use

Use when FASTQ reads from nanopore sequencing samples must be taxonomically classified against VirDetector's curated viral reference database (database/sequences.fasta) as part of the metagenomics module, e.g. for virus surveillance without a known reference genome.

## When NOT to use

- The sample already has a known reference sequence and the goal is reference-based genome reconstruction and phylogenetic analysis rather than de novo metagenomic classification
- Only assembled FASTA contigs are available as input without the reads needed for the KMA mapping and assembly step
- Taxonomic classification against a comprehensive database (e.g. full RefSeq/NCBI nt) is required — VirDetector classifies only against its own curated size-filtered viral database of segmented viruses

## Inputs

- Trimmed FASTQ read files per sample (.fastq, .fq, .fastq.gz, .fq.gz) from upstream preprocessing
- VirDetector viral reference database (database/sequences.fasta) filtered by make_dbs.py

## Outputs

- Per-sample viral consensus sequences assembled from KMA-mapped reads
- Per-sample Kraken2 taxonomic classifications of assembled genomes
- A single summarising taxonomic classification report across samples
- One interactive Krona chart per sample rendered from the classification output

## How to apply

Load the trimmed FASTQ files produced in upstream preprocessing and map them against the size-filtered viral reference database (built from database/sequences.fasta via make_dbs.py) using KMA, treating each file or subdirectory in the input folder as one sample. Assemble the KMA-mapped reads into per-sample viral consensus sequences. Run Kraken2 on each assembled genome against the same viral reference database, which covers segmented viruses as independent L, M, and S entries. Compile the per-sample Kraken2 outputs into a single summarising taxonomic classification report and render one interactive Krona chart per sample. The number of genomes assembled is determined by the KMA mapping and assembly of reads per sample, and each assembled genome receives exactly one Kraken2 label.

## Related tools

- **VirDetector** (Nextflow-based pipeline that orchestrates the metagenomics module, defines samples from the input directory, and supplies the viral reference database) — https://github.com/NLKaiser/VirDetector
- **KMA** (Maps trimmed FASTQ reads against the size-filtered viral reference database before assembly)
- **Kraken2** (Assigns a taxonomic label to each assembled viral genome against the same VirDetector reference database)
- **Krona** (Renders one interactive taxonomic chart per sample from the Kraken2 classification output)
- **Nextflow** (Workflow engine executing the VirDetector pipeline steps)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- Each file or subdirectory in the input folder is treated as exactly one sample and yields one Kraken2 classification and one Krona chart
- Every assembled genome receives a Kraken2 taxonomic label drawn from the VirDetector database, which represents segmented viruses as independent L, M, and S entries
- All per-sample Kraken2 results are compiled into one summarising taxonomic classification report (rather than scattered per-sample files)
- The KMA mapping and assembly step runs against the database built from database/sequences.fasta via make_dbs.py, not an ad hoc reference
- The full pipeline (including this module, minus basecalling) completes via 'nextflow run main.nf' with the bundled West Nile virus test FASTQ

## Limitations

- Classification is restricted to the viruses contained in VirDetector's own database (database/sequences.fasta); viruses outside it will not be identified
- If a metadata file is specified in nextflow.config, only the samples defined in that file are passed on to subsequent processing steps
- Basecalling is excluded from the pipeline's default test execution and must be explicitly configured (Dorado or Guppy, with flowcell and kit parameters) if raw FAST5/POD5 input is used

## Evidence

- [other] Kraken2 assigns a taxonomic label to each assembled genome (after reads are mapped against the database with KMA): [evidence span withheld — anchor status: not_found]
- [intro] It can taxonomically classify samples and reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach.: "It can taxonomically classify samples and reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach."
- [abstract] the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes: "the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"
- [readme] Files placed or created in the *input* directory are processed as separate samples: "Files placed or created in the *input* directory are processed as separate samples"
- [methods] The VirDetector database for phylogenetic inference currently contains the viruses:: "The VirDetector database for phylogenetic inference currently contains the viruses:"

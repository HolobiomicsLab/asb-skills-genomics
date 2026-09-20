---
name: metagenomic-consensus-genome-assembly
description: Use when nanopore FASTQ/FAST5/POD5 sample folders must be turned into
  per-sample viral consensus genomes with taxonomic labels (KMA → assembly → Kraken2
  → Krona) as part of the VirDetector pipeline.
when_to_use_negative:
- Input is already assembled FASTA contigs without reads — VirDetector accepts FASTA
  input only from the alignment step onwards, so the KMA mapping/assembly stage does
  not apply
- Genome reconstruction against a user-supplied reference sequence or hybrid assembly
  (params.ref / params.canu_genome_size) is intended instead of the metagenomics database
  route
- Raw, untrimmed reads that have not passed the pipeline's preprocessing and optional
  basecalling/trimming steps
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: VirDetector
  role: Encapsulates the metagenomics step and builds the size-filtered viral reference
    database via make_dbs.py
  repo: https://github.com/NLKaiser/VirDetector.git
- name: Nextflow
  role: Workflow engine that orchestrates the KMA mapping, assembly, Kraken2 and Krona
    steps in main.nf
- name: KMA
  role: Maps trimmed per-sample reads against the size-filtered viral reference database
    before consensus assembly
- name: Kraken2
  role: Assigns a taxonomic label to each assembled genome against the same viral
    reference database, including segmented L/M/S entries
- name: Krona
  role: Renders one interactive taxonomic chart per sample from the Kraken2 classification
    output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:4b0e95a9d784895481111106bc46e7d2117c7c432d32527c7d6b131254cce367
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector.git
  tools:
  - KMA
  - Kraken2
  - Krona
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# metagenomic-consensus-genome-assembly

## Summary

Performs VirDetector's metagenomics step: maps nanopore FASTQ reads against a size-filtered viral reference database with KMA, assembles per-sample viral consensus genomes, assigns taxonomy to each assembled genome with Kraken2, and reports/visualises results with a summary report and per-sample Krona charts.

## When to use

Use when nanopore FASTQ/FAST5/POD5 sample folders must be turned into per-sample viral consensus genomes with taxonomic labels (KMA → assembly → Kraken2 → Krona) as part of the VirDetector pipeline.

## When NOT to use

- Input is already assembled FASTA contigs without reads — VirDetector accepts FASTA input only from the alignment step onwards, so the KMA mapping/assembly stage does not apply
- Genome reconstruction against a user-supplied reference sequence or hybrid assembly (params.ref / params.canu_genome_size) is intended instead of the metagenomics database route
- Raw, untrimmed reads that have not passed the pipeline's preprocessing and optional basecalling/trimming steps

## Inputs

- Trimmed FASTQ read files per sample (.fastq, .fq, .fastq.gz, .fq.gz) produced by upstream preprocessing
- VirDetector viral reference database (database/sequences.fasta), size-filtered via make_dbs.py
- Input folder structure where each file or subdirectory defines one sample

## Outputs

- Per-sample viral consensus genome sequences (assembled from KMA-mapped reads)
- Kraken2 taxonomic labels for each assembled genome
- One summarising taxonomic classification report across all samples
- One interactive Krona chart per sample

## How to apply

Load the trimmed FASTQ read files produced during upstream preprocessing, treating each file or each subdirectory inside the input folder as one sample (files in a subdirectory are combined into a single sample). Map reads against the size-filtered viral reference database built from database/sequences.fasta via make_dbs.py using KMA, then assemble the KMA-mapped reads into per-sample viral consensus sequences. Assign a taxonomic label to each assembled genome with Kraken2 against the same viral reference database, which covers the segmented viruses as independent L, M and S entries. Compile the per-sample Kraken2 results into one summarising taxonomic classification report and render one interactive Krona chart per sample from the classification output. Correctness is judged by per-sample consensus FASTAs, a coherent Kraken2 classification report across samples, and Krona charts that reflect each sample's assigned taxonomy.

## Related tools

- **VirDetector** (Encapsulates the metagenomics step and builds the size-filtered viral reference database via make_dbs.py) — https://github.com/NLKaiser/VirDetector.git
- **Nextflow** (Workflow engine that orchestrates the KMA mapping, assembly, Kraken2 and Krona steps in main.nf)
- **KMA** (Maps trimmed per-sample reads against the size-filtered viral reference database before consensus assembly)
- **Kraken2** (Assigns a taxonomic label to each assembled genome against the same viral reference database, including segmented L/M/S entries)
- **Krona** (Renders one interactive taxonomic chart per sample from the Kraken2 classification output)

## Examples

```
conda activate VirDetector && nextflow run main.nf
```

## Evaluation signals

- Every sample defined by the input folder structure (file or subdirectory) yields a Kraken2 taxonomic label and appears in the summary report
- Each sample produces a Krona chart consistent with its Kraken2 classification output
- Consensus genomes are assembled only from reads mapped against the size-filtered database/sequences.fasta reference, so contig taxonomy must match Kraken2 labels
- Segmented viruses in the database appear as independent L, M and S entries in the classification results
- Running `nextflow run main.nf` on the bundled West Nile virus test FASTQ completes without errors and yields the expected classification

## Limitations

- Sample definition is tied to the input directory layout — files in a subdirectory are automatically combined into one sample, so a mis-grouped folder changes results
- If a metadata file is specified, only samples defined in that file are passed on to subsequent processing steps
- Basecalling is excluded from the default pipeline execution and must be explicitly selected with a basecaller (Guppy or Dorado) plus flowcell/kit parameters
- The viral reference database is limited to the viruses listed in database/sequences.fasta, so classification coverage is bounded by that database

## Evidence

- [full_text] Kraken2 assigns a taxonomic label to each assembled genome: [evidence span withheld — anchor status: not_found]
- [abstract] Classification and consensus reconstruction in abstract: "the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes, which are then used in"
- [readme] Sample definition from input layout: "Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample."
- [readme] Accepted input formats: "VirDetector accepts zipped and unzipped files in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz), FAST5 files (.fast5, .fast5.gz) and POD5 files (.pod5, .pod5.gz) as an input."
- [readme] Pipeline execution command: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:"

---
name: segmented-viral-reference-database-handling
description: Use when metagenomic taxonomic classification of viral nanopore samples
  must correctly represent segmented viruses, requiring each L, M, or S segment to
  be an independent entry in the VirDetector reference database used by KMA and Kraken2.
when_to_use_negative:
- Input is FASTA assemblies rather than reads that need KMA mapping — VirDetector
  accepts FASTA files but they are used from the alignment step onwards, not for this
  metagenomics mapping/classification flow
- Samples target non-segmented viruses only, where segment-level database entries
  provide no additional discrimination and a simpler classification suffices
- A reference-based or hybrid assembly of a single known genome is the goal instead
  of per-sample taxonomic classification of metagenomic viral content
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VirDetector
  role: Pipeline orchestrating database construction (make_dbs.py), KMA mapping, assembly,
    Kraken2 classification and Krona visualisation for the metagenomics module
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine executing the metagenomics steps of VirDetector
  repo: https://github.com/NLKaiser/VirDetector
- name: KMA
  role: Maps trimmed sample reads against the size-filtered viral reference database
    containing independent L, M, S segment entries
- name: Kraken2
  role: Assigns a taxonomic label to each assembled genome against the same viral
    reference database, covering segmented viruses
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
  source_content_hash: sha256:9b0531d7865cfb6f51dfdd6e9cf6d01cdf66569d6589c3524dd4da9bd0b14051
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# segmented-viral-reference-database-handling

## Summary

Handles segmented viruses (viruses with independent L, M, and S genome segments) within VirDetector's metagenomics step by treating each segment as an independent reference entry in the size-filtered viral reference database, so that KMA-mapped reads can be assembled and Kraken2 can assign taxonomy at the segment level.

## When to use

Use when metagenomic taxonomic classification of viral nanopore samples must correctly represent segmented viruses, requiring each L, M, or S segment to be an independent entry in the VirDetector reference database used by KMA and Kraken2.

## When NOT to use

- Input is FASTA assemblies rather than reads that need KMA mapping — VirDetector accepts FASTA files but they are used from the alignment step onwards, not for this metagenomics mapping/classification flow
- Samples target non-segmented viruses only, where segment-level database entries provide no additional discrimination and a simpler classification suffices
- A reference-based or hybrid assembly of a single known genome is the goal instead of per-sample taxonomic classification of metagenomic viral content

## Inputs

- Trimmed FASTQ read files per sample (from upstream preprocessing)
- Size-filtered viral reference database built from database/sequences.fasta via make_dbs.py, including independent L, M, S segment entries for segmented viruses
- Input folder where each file or subdirectory is treated as one sample

## Outputs

- Per-sample KMA-mapped reads and assembled per-sample viral consensus sequences
- Kraken2 taxonomic labels for each assembled genome, covering segmented virus entries
- One summarising taxonomic classification report across samples
- One Krona interactive taxonomic chart per sample

## How to apply

Build the viral reference database from database/sequences.fasta using make_dbs.py, keeping each segment (independent L, M, S entries) of segmented viruses as a separate reference sequence rather than collapsing them. Map the trimmed FASTQ reads of each sample against this size-filtered database with KMA, treating each file or subdirectory in the input folder as one sample, then assemble the mapped reads into per-sample viral consensus sequences. Assign taxonomy to each assembled genome with Kraken2 against the same viral reference database, so segmented viruses can be recognised per segment. Finally, compile per-sample Kraken2 results into one summarising taxonomic report and render one Krona interactive chart per sample. The rationale is that without independent segment entries, Kraken2 cannot distinguish which segment contributed reads in a multi-segment viral genome.

## Related tools

- **VirDetector** (Pipeline orchestrating database construction (make_dbs.py), KMA mapping, assembly, Kraken2 classification and Krona visualisation for the metagenomics module) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine executing the metagenomics steps of VirDetector) — https://github.com/NLKaiser/VirDetector
- **KMA** (Maps trimmed sample reads against the size-filtered viral reference database containing independent L, M, S segment entries)
- **Kraken2** (Assigns a taxonomic label to each assembled genome against the same viral reference database, covering segmented viruses)
- **Krona** (Renders one interactive taxonomic chart per sample from the Kraken2 classification output)

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- Each segmented virus in the built database appears as separate independent L, M, and S entries rather than a single concatenated record
- Kraken2 reports taxonomic labels for assembled genomes that resolve to segment-level entries of segmented viruses where applicable
- Exactly one summarising taxonomic classification report is produced across all samples
- One Krona interactive chart exists per processed sample, consistent with the number of samples treated (files/subdirectories in the input folder)
- Assembled consensus sequences are derived from KMA-mapped reads only, confirming reads were mapped against the size-filtered database built from database/sequences.fasta

## Limitations

- Classification is limited to the viruses contained in the VirDetector database; taxonomy coverage is bounded by database/sequences.fasta
- If a metadata file is specified, only the samples defined in that file are passed on to subsequent processing steps
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation

## Evidence

- [methods] Kraken2 assigns taxonomy per assembled genome over segmented viruses: [evidence span withheld — anchor status: not_found]
- [methods] Reads are mapped against the size-filtered database with KMA: [evidence span withheld — anchor status: not_found]
- [methods] One summarising report and per-sample Krona charts: [evidence span withheld — anchor status: not_found]
- [readme] Sample definition rule for input files: "Files placed or created in the *input* directory are processed as separate samples, while files in each subdirectory inside the *input* directory are automatically combined into one sample."
- [abstract] Pipeline classifies each sample taxonomically and reconstructs consensus genomes: "After preprocessing the samples, including the possibility for basecalling, the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes"

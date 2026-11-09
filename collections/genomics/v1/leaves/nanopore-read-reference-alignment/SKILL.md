---
name: nanopore-read-reference-alignment
description: Use when a viral genome must be reconstructed from nanopore FASTQ reads
  for which a suitable reference FASTA already exists, e.g. a known virus such as
  West Nile virus where mapping beats de novo assembly.
when_to_use_negative:
- No suitable reference sequence is available and an automated hybrid assembly approach
  (e.g. with params.canu_genome_size set) should be used instead.
- Input is already assembled contigs or FASTA sequences rather than reads — VirDetector
  accepts FASTA input only from the alignment step onwards.
- The goal is taxonomic classification of unknown samples rather than reconstruction
  of a specific known viral genome.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: minimap2
  role: Aligns the sample's nanopore reads against the provided reference FASTA
- name: samtools
  role: Sorts and indexes the alignment BAM and calls the viral consensus genome from
    the sorted BAM
- name: Qualimap
  role: Produces mapping and coverage statistics visualisations from the sorted BAM
- name: VirDetector
  role: Nextflow pipeline orchestrating the reference-based genome reconstruction
    branch
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine executing the pipeline steps defined in main.nf and nextflow.config
  repo: https://github.com/NLKaiser/VirDetector
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:4baf9084458a3baa9cdd1acbeb0efabeae92b7b6b0b80eaa8aa538a5830af097
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - Nextflow
  - Qualimap
  - VirDetector
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nanopore-read-reference-alignment

## Summary

Reference-based alignment of filtered nanopore reads against a provided viral reference sequence using minimap2, with samtools-based BAM sorting/indexing, Qualimap coverage reporting, and samtools consensus genome generation — the reference branch of VirDetector's genome reconstruction layer.

## When to use

Use when a viral genome must be reconstructed from nanopore FASTQ reads for which a suitable reference FASTA already exists, e.g. a known virus such as West Nile virus where mapping beats de novo assembly.

## When NOT to use

- No suitable reference sequence is available and an automated hybrid assembly approach (e.g. with params.canu_genome_size set) should be used instead.
- Input is already assembled contigs or FASTA sequences rather than reads — VirDetector accepts FASTA input only from the alignment step onwards.
- The goal is taxonomic classification of unknown samples rather than reconstruction of a specific known viral genome.

## Inputs

- trimmed nanopore reads in FASTQ format (.fastq, .fq, .fastq.gz, .fq.gz)
- reference sequence in FASTA format specified via params.ref in nextflow.config

## Outputs

- sorted and indexed BAM alignment file
- Qualimap mapping and coverage statistics visualisations
- consensus genome FASTA (consensus_genome.fasta)

## How to apply

Load the trimmed FASTQ reads for the sample together with the reference FASTA specified via params.ref in nextflow.config (e.g. placed in the resources folder). Align the reads to the reference with minimap2, which handles the high error profile of Oxford Nanopore long reads, then sort and index the resulting BAM file with samtools. Run Qualimap on the sorted BAM to produce mapping and coverage statistics visualisations, which reveal how completely the reference is covered before trusting the consensus. Finally call the consensus genome from the sorted BAM with samtools, emitting consensus_genome.fasta for downstream phylogenetic analyses.

## Related tools

- **minimap2** (Aligns the sample's nanopore reads against the provided reference FASTA)
- **samtools** (Sorts and indexes the alignment BAM and calls the viral consensus genome from the sorted BAM)
- **Qualimap** (Produces mapping and coverage statistics visualisations from the sorted BAM)
- **VirDetector** (Nextflow pipeline orchestrating the reference-based genome reconstruction branch) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine executing the pipeline steps defined in main.nf and nextflow.config) — https://github.com/NLKaiser/VirDetector

## Examples

```
nextflow run main.nf
```

## Evaluation signals

- consensus_genome.fasta exists and is a valid FASTA of a reference-length genome
- Sorted and indexed BAM alignment files were generated for the sample
- Qualimap coverage reports were generated and show adequate coverage across the reference length
- The reference FASTA path resolves correctly via params.ref in nextflow.config

## Limitations

- Requires a suitable reference sequence; without one the reference-based branch cannot run and the hybrid assembly branch with an expected genome size (params.canu_genome_size) is needed instead.
- Only samples defined in the metadata file (if params.metadata is set) are passed on to subsequent processing steps.
- Pipeline developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation.

## Evidence

- [other] reference branch workflow: [evidence span withheld — anchor status: not_found]
- [other] alignment and consensus: [evidence span withheld — anchor status: not_found]
- [other] qualimap: [evidence span withheld — anchor status: not_found]
- [other] consensus output: [evidence span withheld — anchor status: not_found]
- [readme] readme reference parameter: "For the reference based assembly, you must specifiy the path to your reference sequence. Example: Place your reference sequence file in fasta format in the folder *resources*"

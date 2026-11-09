---
name: contig-blast-database-search
description: Use when nanopore FASTQ reads must be reconstructed into a viral consensus
  genome but no suitable reference sequence is provided, so a reference must be chosen
  automatically from assembled contigs.
when_to_use_negative:
- A reference sequence is already provided for reference-based assembly (set params.ref
  to the FASTA in the resources folder instead); the hybrid branch adds Canu assembly
  overhead
- Input consists only of FASTA files placed from the alignment step onwards, with
  no reads to assemble de novo
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: VirDetector
  role: Pipeline that hosts the hybrid assembly branch, runs BLAST-based reference
    selection, and feeds the chosen reference into downstream consensus reconstruction
  repo: https://github.com/NLKaiser/VirDetector
- name: Canu
  role: Performs the de novo assembly of trimmed nanopore FASTQ reads to produce the
    contigs BLASTed against the database; genome size set via params.canu_genome_size
- name: BLAST
  role: Aligns assembled contigs against database/sequences.fasta; the database sequence
    with the highest bit-score becomes the sample's reference
- name: Nextflow
  role: Workflow engine executing the pipeline ('nextflow run main.nf') including
    the hybrid assembly branch
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:9618ac7d2b31b5f4937d4e6061bf0516024b2f7f526b6be95fda54e697221479
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - BLAST
  - Canu
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# contig-blast-database-search

## Summary

Selects a reference sequence for VirDetector's hybrid genome-reconstruction layer by BLASTing Canu-assembled contigs against the VirDetector reference database (database/sequences.fasta) and picking the hit with the highest bit-score, which then feeds the standard reference-based consensus pipeline.

## When to use

Use when nanopore FASTQ reads must be reconstructed into a viral consensus genome but no suitable reference sequence is provided, so a reference must be chosen automatically from assembled contigs.

## When NOT to use

- A reference sequence is already provided for reference-based assembly (set params.ref to the FASTA in the resources folder instead); the hybrid branch adds Canu assembly overhead
- Input consists only of FASTA files placed from the alignment step onwards, with no reads to assemble de novo

## Inputs

- Trimmed FASTQ reads of a nanopore sample (e.g. West Nile virus test dataset)
- Canu-assembled contigs (de novo assembly output)
- VirDetector reference database (database/sequences.fasta)
- nextflow.config with params.assembly_hybrid enabled and params.canu_genome_size set

## Outputs

- Selected reference sequence from database/sequences.fasta (highest bit-score BLAST hit)
- Consensus genome FASTA (~11000 bases for West Nile virus) with structural variant calls
- Top BLAST hit taxonomic assignment (e.g. West Nile virus)

## How to apply

First enable the hybrid branch in nextflow.config by setting params.assembly_hybrid and specifying the expected genome size via params.canu_genome_size (e.g. "11k" for the ~11000-base West Nile virus genome), then run the de novo assembly with Canu through the Nextflow pipeline ('nextflow run main.nf' after 'conda activate VirDetector'). BLAST the resulting contigs against the VirDetector reference database at database/sequences.fasta. Select the database sequence with the highest bit-score as the reference for the sample. Map the original trimmed FASTQ reads against this selected reference and reconstruct the consensus genome, keeping structural variant calls as part of the standard reference-based branch — downstream steps are identical to the reference-based assembly.

## Related tools

- **VirDetector** (Pipeline that hosts the hybrid assembly branch, runs BLAST-based reference selection, and feeds the chosen reference into downstream consensus reconstruction) — https://github.com/NLKaiser/VirDetector
- **Canu** (Performs the de novo assembly of trimmed nanopore FASTQ reads to produce the contigs BLASTed against the database; genome size set via params.canu_genome_size)
- **BLAST** (Aligns assembled contigs against database/sequences.fasta; the database sequence with the highest bit-score becomes the sample's reference)
- **Nextflow** (Workflow engine executing the pipeline ('nextflow run main.nf') including the hybrid assembly branch)

## Examples

```
conda activate VirDetector; nextflow run main.nf  # with params.assembly_hybrid=true and params.canu_genome_size="11k" in nextflow.config
```

## Evaluation signals

- Consensus genome FASTA file exists and contains a single FASTA record
- Consensus genome length is close to the expected ~11000 bases for the West Nile virus genome
- Top BLAST hit in database/sequences.fasta is assigned to West Nile virus
- Selected reference corresponds to the database entry with the highest bit-score across all contig alignments
- Downstream outputs (consensus, structural variant calls) match what the reference-based branch would produce given the same selected reference

## Limitations

- Reference selection quality depends on the composition of database/sequences.fasta; only sequences present there can be selected
- params.canu_genome_size must approximate the true genome size (e.g. "11k" for West Nile virus); a wrong size can degrade the Canu de novo assembly
- Developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation

## Evidence

- [methods] hybrid assembly contig BLAST reference selection: [evidence span withheld — anchor status: not_found]
- [readme] hybrid assembly genome size parameter: "For the hybrid assembly, you must specify the expected genome size of your organism."
- [readme] genome size example: "Example: You can specify a genome size of 11.000 bases like this: **params.canu_genome_size = "11k"**"
- [readme] pipeline test command: "To test the execution of all steps of the pipeline, except basecalling, execute the following command:
``nnextflow run main.nf
"
- [intro] reference vs hybrid approach: "It can taxonomically classify samples and reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach."

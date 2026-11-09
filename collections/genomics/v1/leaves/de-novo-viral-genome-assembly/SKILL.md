---
name: de-novo-viral-genome-assembly
description: Use when no reference FASTA is available for a nanopore sample but the
  approximate genome size is known (e.g. ~11000 bases for West Nile virus), and a
  consensus genome is needed within the VirDetector pipeline.
when_to_use_negative:
- A user-supplied reference sequence is already configured via params.ref, in which
  case the plain reference-based assembly branch should be used instead of the hybrid
  approach.
- The expected genome size of the target organism is unknown, since Canu requires
  params.canu_genome_size to be set for the hybrid assembly.
- Input is already assembled contigs or FASTA sequences, which VirDetector passes
  to the alignment step onwards rather than through the assembly layer.
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: Canu
  role: performs the de novo assembly of trimmed nanopore reads using the expected
    genome size parameter
  repo: https://github.com/NLKaiser/VirDetector
- name: BLAST
  role: aligns the assembled contigs against database/sequences.fasta; the highest
    bit-score hit is selected as the reference
- name: Nextflow
  role: orchestrates the hybrid assembly branch of the VirDetector pipeline (nextflow
    run main.nf)
  repo: https://github.com/NLKaiser/VirDetector
- name: VirDetector
  role: pipeline providing the hybrid assembly mode, reference database, and downstream
    consensus/structural-variant steps
  repo: https://github.com/NLKaiser/VirDetector
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:7aef698fcfba2e74c794cc8089987cdd16ea516232039754588a8d0f673fa9cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NLKaiser/VirDetector
  tools:
  - BLAST
  - Canu
  - Nextflow
  - VirDetector
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# de-novo-viral-genome-assembly

## Summary

Reconstruct a viral consensus genome without a pre-specified reference by first assembling nanopore reads de novo with Canu, then selecting a reference sequence from the VirDetector database via BLAST bit-score, after which the standard reference-based consensus branch proceeds.

## When to use

Use when no reference FASTA is available for a nanopore sample but the approximate genome size is known (e.g. ~11000 bases for West Nile virus), and a consensus genome is needed within the VirDetector pipeline.

## When NOT to use

- A user-supplied reference sequence is already configured via params.ref, in which case the plain reference-based assembly branch should be used instead of the hybrid approach.
- The expected genome size of the target organism is unknown, since Canu requires params.canu_genome_size to be set for the hybrid assembly.
- Input is already assembled contigs or FASTA sequences, which VirDetector passes to the alignment step onwards rather than through the assembly layer.

## Inputs

- trimmed nanopore reads in FASTQ format (e.g. the West Nile virus test sample)
- expected genome size value for Canu (e.g. "11k")
- VirDetector reference database (database/sequences.fasta)

## Outputs

- assembled contigs from Canu de novo assembly
- BLAST hits of contigs against the reference database with bit-scores
- selected reference sequence (highest bit-score, e.g. West Nile virus NC_009942.1)
- consensus genome FASTA of ~11000 bases with a single FASTA record

## How to apply

Enable the hybrid assembly branch in nextflow.config by setting params.assembly_hybrid to enabled and params.canu_genome_size to the expected genome size (e.g. "11k" for the ~11000-base West Nile virus genome). Load the trimmed FASTQ reads of the sample and run the de novo assembly step with Canu via the Nextflow pipeline ('nextflow run main.nf' after 'conda activate VirDetector') to obtain assembled contigs. BLAST the resulting contigs against the VirDetector reference database (database/sequences.fasta) and select the database sequence with the highest bit-score as the reference for the sample. Then map the original trimmed reads against this selected reference and reconstruct the consensus genome, keeping structural variant calls as part of the standard reference-based branch. The rationale is that Canu's long-read de novo assembly plus a bit-score-ranked BLAST hit substitutes for a user-supplied reference, while downstream consensus generation remains identical to the reference-based workflow.

## Related tools

- **Canu** (performs the de novo assembly of trimmed nanopore reads using the expected genome size parameter) — https://github.com/NLKaiser/VirDetector
- **BLAST** (aligns the assembled contigs against database/sequences.fasta; the highest bit-score hit is selected as the reference)
- **Nextflow** (orchestrates the hybrid assembly branch of the VirDetector pipeline (nextflow run main.nf)) — https://github.com/NLKaiser/VirDetector
- **VirDetector** (pipeline providing the hybrid assembly mode, reference database, and downstream consensus/structural-variant steps) — https://github.com/NLKaiser/VirDetector

## Examples

```
conda activate VirDetector; nextflow run main.nf  # with params.assembly_hybrid = "true" and params.canu_genome_size = "11k" in nextflow.config
```

## Evaluation signals

- The consensus genome FASTA file exists and contains a single FASTA record.
- The consensus genome length is close to the expected ~11000 bases for the West Nile virus genome.
- The top BLAST hit assigned as reference corresponds to West Nile virus (e.g. accession NC_009942.1).
- params.assembly_hybrid is enabled and params.canu_genome_size matches the expected genome size (e.g. "11k") in nextflow.config.
- Structural variant calls from the reference-based branch are present alongside the consensus genome.

## Limitations

- Canu requires the expected genome size to be specified (params.canu_genome_size); an inaccurate genome size may degrade the de novo assembly.
- The installation can take several hours depending on available computing resources, since it includes the basecallers Guppy and Dorado.
- VirDetector has been developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation.

## Evidence

- [methods] Hybrid assembly de novo + BLAST reference selection: [evidence span withheld — anchor status: not_found]
- [readme] Genome size parameter: "For the hybrid assembly, you must specify the expected genome size of your organism. Example: You can specify a genome size of 11.000 bases like this: params.canu_genome_size = "11k""
- [methods] Reference-based branch identity: "subsequent steps are identical to the reference-based assembly"
- [readme] Test dataset context: "The repository contains a test input file of the West Nile virus in FASTQ format, as well as a reference sequence."
- [intro] Structural variant detection: "It also detects structural variants in the assembled genomes."

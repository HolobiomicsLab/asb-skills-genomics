---
name: viral-reference-based-assembly
description: Use when nanopore FASTQ reads of a viral sample are available and a reference
  genome sequence for the target virus exists, and a consensus genome is needed for
  downstream phylogenetic analysis.
when_to_use_negative:
- No reference genome is available for the target virus — VirDetector's automated
  hybrid assembly branch (with params.canu_genome_size, e.g. "11k") is the alternative
- Input consists only of already-assembled FASTA sequences, which VirDetector passes
  to the alignment step onwards rather than mapping to a reference
- Samples are not defined in a provided metadata file, since when metadata is specified
  only those samples are passed on to subsequent processing steps
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: minimap2
  role: Aligns the sample's trimmed nanopore reads against the provided reference
    FASTA to produce the alignment used for consensus calling
- name: samtools
  role: Sorts and indexes the minimap2 alignment (BAM) and calls the viral consensus
    genome from the sorted BAM, emitting consensus_genome.fasta
- name: Qualimap
  role: Generates mapping statistics and coverage visualisations on the sorted BAM
    to validate read placement across the reference
- name: VirDetector
  role: Nextflow pipeline that orchestrates the reference-based genome reconstruction
    branch and emits consensus_genome.fasta per sample
  repo: https://github.com/NLKaiser/VirDetector
- name: Nextflow
  role: Workflow engine executing the pipeline steps; the reference is configured
    via params.ref in nextflow.config
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:25e71e158de3506a569cde24427421b399d1c59473066349ab64f0baaf6de3b1
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
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# viral-reference-based-assembly

## Summary

Reconstructs a viral consensus genome from filtered nanopore reads by aligning them to a user-supplied reference FASTA within VirDetector's reference-based branch, combining minimap2 alignment, Qualimap coverage QC, and samtools consensus calling.

## When to use

Use when nanopore FASTQ reads of a viral sample are available and a reference genome sequence for the target virus exists, and a consensus genome is needed for downstream phylogenetic analysis.

## When NOT to use

- No reference genome is available for the target virus — VirDetector's automated hybrid assembly branch (with params.canu_genome_size, e.g. "11k") is the alternative
- Input consists only of already-assembled FASTA sequences, which VirDetector passes to the alignment step onwards rather than mapping to a reference
- Samples are not defined in a provided metadata file, since when metadata is specified only those samples are passed on to subsequent processing steps

## Inputs

- Trimmed nanopore FASTQ reads per sample (.fastq, .fq, .fastq.gz, .fq.gz)
- Reference genome sequence in FASTA format, placed in the resources folder and set via params.ref in nextflow.config

## Outputs

- consensus_genome.fasta (viral consensus genome of approximately reference length)
- Sorted and indexed BAM alignment of sample reads against the reference
- Qualimap mapping statistics and coverage visualisation reports

## How to apply

Load the trimmed FASTQ reads for the sample together with the reference FASTA specified via params.ref in nextflow.config (e.g. placed in the resources folder). Align the reads to the reference with minimap2, then sort and index the resulting BAM file with samtools. Run Qualimap on the sorted BAM to produce mapping statistics and coverage visualisations, which serve as the QC layer for the alignment. Call the consensus genome from the sorted BAM with samtools, emitting consensus_genome.fasta as the final output. The choice of reference is a decision point: VirDetector supports genome reconstruction 'either by using a reference sequence or with an automated hybrid assembly approach', so this skill applies only when a suitable reference is provided.

## Related tools

- **minimap2** (Aligns the sample's trimmed nanopore reads against the provided reference FASTA to produce the alignment used for consensus calling)
- **samtools** (Sorts and indexes the minimap2 alignment (BAM) and calls the viral consensus genome from the sorted BAM, emitting consensus_genome.fasta)
- **Qualimap** (Generates mapping statistics and coverage visualisations on the sorted BAM to validate read placement across the reference)
- **VirDetector** (Nextflow pipeline that orchestrates the reference-based genome reconstruction branch and emits consensus_genome.fasta per sample) — https://github.com/NLKaiser/VirDetector
- **Nextflow** (Workflow engine executing the pipeline steps; the reference is configured via params.ref in nextflow.config)

## Examples

```
nextflow run main.nf  # after setting params.ref = "${PWD}/resources/reference.fasta" in nextflow.config
```

## Evaluation signals

- consensus_genome.fasta exists for the sample and is a valid FASTA whose length matches the reference genome length
- Sorted and indexed BAM files were generated for the sample and can be inspected with samtools
- Qualimap reports were produced and show reads mapped across the reference with sensible coverage distribution
- The pipeline was executed as a Nextflow run (e.g. nextflow run main.nf) with a resolvable params.ref path, completing without errors
- If a test run is used, the West Nile virus test FASTQ and reference sequence reproduce the expected consensus output

## Limitations

- Requires a user-supplied reference sequence in FASTA format; without one the reference-based branch cannot run
- When a metadata file is specified, only samples listed in it are passed on to subsequent processing steps, so unlisted samples will be silently excluded
- The reference-based branch is one of two genome reconstruction strategies; for divergent or novel viruses it may underperform relative to the automated hybrid assembly approach

## Evidence

- [readme] reference configuration: "For the reference based assembly, you must specifiy the path to your reference sequence."
- [readme] test dataset: "The repository contains a test input file of the West Nile virus in FASTQ format, as well as a reference sequence."
- [intro] two reconstruction branches: "reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach"
- [abstract] consensus genomes: "the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes, which are then used in"

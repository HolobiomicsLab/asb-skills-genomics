---
name: bam-mapping-quality-reporting
description: Use when trimmed nanopore FASTQ reads should be aligned to a supplied
  reference sequence (params.ref) to generate alignment statistics and coverage reports
  prior to consensus genome reconstruction.
when_to_use_negative:
- No reference sequence is available and the sample must instead be reconstructed
  via VirDetector's automated hybrid assembly branch (params.canu_genome_size).
- Input files are already FASTA assemblies rather than reads — VirDetector only uses
  them from the alignment step onwards, so read mapping against a reference from raw
  reads does not apply.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: minimap2
  role: Aligns trimmed nanopore reads to the user-specified reference FASTA, producing
    the SAM/BAM alignment
  repo: https://github.com/NLKaiser/VirDetector
- name: samtools
  role: Sorts and indexes the minimap2 alignment and calls the consensus genome from
    the sorted BAM
  repo: https://github.com/NLKaiser/VirDetector
- name: Qualimap
  role: Generates mapping statistics and coverage visualisations from the sorted BAM
    as the quality-reporting layer
- name: Nextflow
  role: Orchestrates the mapping, quality reporting, and consensus steps within the
    VirDetector pipeline (nextflow run main.nf)
  repo: https://github.com/NLKaiser/VirDetector
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf029
    title: 'VirDetector: a bioinformatic pipeline for virus surveillance using nanopore
      sequencing'
  source_content_hash: sha256:ab4de9924608fa58bb7c8ce95e0498d168240fe804a0f6580bad054247799db4
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# BAM mapping and quality reporting

## Summary

Map filtered nanopore reads to a user-provided reference genome with minimap2, sort/index the resulting BAM with samtools, and characterise mapping quality and coverage with Qualimap. This is the reference-based branch VirDetector uses to produce a reliable viral consensus genome before consensus calling.

## When to use

Use when trimmed nanopore FASTQ reads should be aligned to a supplied reference sequence (params.ref) to generate alignment statistics and coverage reports prior to consensus genome reconstruction.

## When NOT to use

- No reference sequence is available and the sample must instead be reconstructed via VirDetector's automated hybrid assembly branch (params.canu_genome_size).
- Input files are already FASTA assemblies rather than reads — VirDetector only uses them from the alignment step onwards, so read mapping against a reference from raw reads does not apply.

## Inputs

- Trimmed nanopore FASTQ reads per sample (.fastq, .fastq.gz)
- Reference sequence FASTA (set via params.ref, e.g. resources/{your_file_name}.fasta)

## Outputs

- Sorted and indexed BAM alignment against the reference
- Qualimap mapping and coverage statistics reports/visualisations
- consensus_genome.fasta (reference-length viral consensus FASTA)

## How to apply

Load the trimmed FASTQ reads for the sample and the reference FASTA specified via params.ref in nextflow.config. Align the reads to the reference with minimap2 and sort/index the resulting BAM file with samtools. Run Qualimap on the sorted BAM to produce mapping statistics and coverage visualisations, which serve as the quality gate before consensus calling. Finally, the same sorted BAM is consumed by samtools to emit the consensus FASTA (consensus_genome.fasta). The rationale is that coverage and mapping statistics from Qualimap reveal whether the nanopore reads adequately and correctly cover the reference, so a consensus genome is only trusted when the alignment layer is validated.

## Related tools

- **minimap2** (Aligns trimmed nanopore reads to the user-specified reference FASTA, producing the SAM/BAM alignment) — https://github.com/NLKaiser/VirDetector
- **samtools** (Sorts and indexes the minimap2 alignment and calls the consensus genome from the sorted BAM) — https://github.com/NLKaiser/VirDetector
- **Qualimap** (Generates mapping statistics and coverage visualisations from the sorted BAM as the quality-reporting layer)
- **Nextflow** (Orchestrates the mapping, quality reporting, and consensus steps within the VirDetector pipeline (nextflow run main.nf)) — https://github.com/NLKaiser/VirDetector

## Examples

```
params.ref = "${PWD}/resources/{your_file_name}.fasta"  # then: nextflow run main.nf
```

## Evaluation signals

- consensus_genome.fasta exists and is a valid FASTA of a reference-length genome
- Sorted BAM and index files were generated from the minimap2 alignment
- Qualimap coverage reports were produced for the sample
- Consensus sequence length matches the reference specified via params.ref
- Pipeline completes with nextflow run main.nf on the bundled West Nile virus test FASTQ and reference

## Limitations

- Reference-based reconstruction depends on a user-supplied reference; the path must be set explicitly in nextflow.config (params.ref) or the branch cannot run.
- VirDetector was developed and tested under Ubuntu 22.04 LTS; other operating systems require the Docker installation.
- Coverage statistics alone do not guarantee consensus correctness — the consensus is only reference-length if the reads cover the reference adequately.

## Evidence

- [readme] Reference configuration requirement: "For the reference based assembly, you must specifiy the path to your reference sequence."
- [intro] Reference-based vs hybrid assembly branches: "reconstructs viral genomes, either by using a reference sequence or with an automated hybrid assembly approach"
- [abstract] Consensus genome output feeds phylogenetics: "the pipeline classifies each sample taxonomically and reconstructs the viral consensus genomes, which are then used in"
- [readme] Pipeline execution entry point: "nextflow run main.nf"

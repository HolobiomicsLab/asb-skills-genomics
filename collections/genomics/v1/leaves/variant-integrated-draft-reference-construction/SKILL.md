---
name: variant-integrated-draft-reference-construction
description: Use when a reference genome has been selected (highest-scoring BLAST
  hit from de novo contigs) and a reference-based consensus genome must be reconstructed
  from trimmed nanopore reads, including large-indel integration and polishing.
when_to_use_negative:
- 'Large DNA viruses with extensive repetitive regions (e.g. mpox): the README states
  assemblies may contain inaccuracies and should only be regarded as draft genomes'
- No well-fitting reference genome was identified for the sample (BLAST finds no suitable
  viral reference), since this module is inherently reference-guided
- A de novo assembly is required instead of a reference-anchored consensus; this skill
  deliberately projects reads onto a known viral reference
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: Samtools
  role: Maps and sorts/indexes trimmed nanopore reads against the reference genome
    to produce the per-reference BAM, and provides the fallback consensus calling
    (min depth 20, min 70% base proportion, N-masking)
- name: Sniffles2
  role: Calls large insertions/deletions from the alignment BAM and records them in
    a VCF for integration into the reference
- name: BCFtools
  role: Applies the Sniffles2-detected indels to the reference sequence (bcftools
    consensus) after pre-filtering <INV> records, yielding the indel-corrected draft
    reference
- name: Medaka
  role: Polishes the indel-corrected draft into the final consensus using a neural-network
    model ('auto' or a specific medaka_consensus_model)
  repo: https://github.com/nanoporetech/medaka
- name: Nextflow
  role: Workflow engine that executes the ViMOP pipeline steps, including the consensus
    assembly module
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:6df09ae49aa9c97d26d6a0f72dd8c1c65a00c078502b7ab3c2e34fc5169dab77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/medaka
  tools:
  - BCFtools
  - Medaka
  - Nextflow
  - Samtools
  - Sniffles2
  - ViMOP
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# variant-integrated-draft-reference-construction

## Summary

ViMOP's reference-guided consensus assembly module: after BLAST identifies a reference genome, nanopore reads are remapped, large indels called with Sniffles2 are applied to the reference via BCFtools consensus, and the draft is polished to a final consensus FASTA with Medaka (or a Samtools fallback). This turns untargeted nanopore reads into a consensus virus genome anchored on a known reference.

## When to use

Use when a reference genome has been selected (highest-scoring BLAST hit from de novo contigs) and a reference-based consensus genome must be reconstructed from trimmed nanopore reads, including large-indel integration and polishing.

## When NOT to use

- Large DNA viruses with extensive repetitive regions (e.g. mpox): the README states assemblies may contain inaccuracies and should only be regarded as draft genomes
- No well-fitting reference genome was identified for the sample (BLAST finds no suitable viral reference), since this module is inherently reference-guided
- A de novo assembly is required instead of a reference-anchored consensus; this skill deliberately projects reads onto a known viral reference

## Inputs

- Trimmed nanopore reads aligned against the BLAST-selected reference genome (per-reference BAM)
- BLAST-selected reference genome sequence (FASTA)
- Sniffles2 VCF of large insertions/deletions (with <INV> records pre-filtered for BCFtools)
- Optional medaka_consensus_model parameter (or 'auto')

## Outputs

- Final consensus genome FASTA (e.g. AB627954.consensus.fasta)
- Alignment BAM (e.g. AB627954.reads.bam)
- Depth file
- Variant VCF

## How to apply

First map the trimmed nanopore reads against the BLAST-selected reference and sort/index the alignment with Samtools to yield a per-reference BAM. Call large insertions/deletions from the BAM with Sniffles2 into a VCF, then apply these indels to the reference with BCFtools consensus — pre-filtering any <INV> records from the VCF to avoid the known BCFtools consensus error — producing an indel-corrected draft. Polish the draft with Medaka using model 'auto' unless a specific medaka_consensus_model is available; if Medaka cannot fit a model for the data, fall back to Samtools consensus. In the Samtools path, require a minimum depth of 20 and a minimum base proportion of 70%, calling the most abundant base and masking positions below these thresholds with N. Write the final consensus as a FASTA (e.g. AB627954.consensus.fasta) alongside the alignment BAM (AB627954.reads.bam), depth file, and variant VCF in the sample's consensus output directory.

## Related tools

- **Samtools** (Maps and sorts/indexes trimmed nanopore reads against the reference genome to produce the per-reference BAM, and provides the fallback consensus calling (min depth 20, min 70% base proportion, N-masking))
- **Sniffles2** (Calls large insertions/deletions from the alignment BAM and records them in a VCF for integration into the reference)
- **BCFtools** (Applies the Sniffles2-detected indels to the reference sequence (bcftools consensus) after pre-filtering <INV> records, yielding the indel-corrected draft reference)
- **Medaka** (Polishes the indel-corrected draft into the final consensus using a neural-network model ('auto' or a specific medaka_consensus_model)) — https://github.com/nanoporetech/medaka
- **Nextflow** (Workflow engine that executes the ViMOP pipeline steps, including the consensus assembly module) — https://github.com/opr-group-bnitm/vimop

## Evaluation signals

- The consensus FASTA and alignment BAM both exist in the sample's consensus output directory (e.g. AB627954.consensus.fasta and AB627954.reads.bam)
- The consensus contains no ambiguous bases other than N-masked positions
- All positions with coverage below the depth-20 threshold or base proportion below 70% (Samtools path) are represented as N
- All <INV> records were pre-filtered from the VCF before BCFtools consensus to avoid the known error
- The chosen Medaka model either resolved to 'auto' or a supplied medaka_consensus_model; the Samtools fallback is only used when Medaka cannot fit a model for the data

## Limitations

- BCFtools consensus throws an unresolved error when inversions denoted with <INV> are present in the VCF file, so such records must be removed beforehand
- The Samtools consensus fallback requires minimum depth 20 and at least 70% base proportion; positions below these thresholds are masked with N, losing sequence information
- For large DNA viruses with extensive repetitive regions (e.g. mpox), reference-based assemblies may contain inaccuracies and should be regarded as draft genomes; the README recommends reviewing output such as the alignment .bam files

## Evidence

- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [methods] Reads are mapped against the reference genome.: "Reads are mapped against the reference genome."
- [discussion] error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools: "error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools"
- [readme] The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes.: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."

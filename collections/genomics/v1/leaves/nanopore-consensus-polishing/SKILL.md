---
name: nanopore-consensus-polishing
description: Use when reconstructing a consensus genome for a known virus from untargeted
  nanopore reads after a reference genome has been identified, and a polished reference-guided
  consensus (not de novo contigs) is the deliverable.
when_to_use_negative:
- The goal is a finished genome for a large DNA virus with extensive repetitive regions
  (e.g. mpox) — ViMOP output should then only be regarded as a draft genome
- No fitting reference genome can be identified in the virus reference database, since
  this skill is reference-guided consensus assembly, not de novo assembly
- Reads are not Oxford Nanopore data (e.g. Illumina), as the polishing path depends
  on nanopore-specific Medaka models
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0196
tools:
- name: Sniffles2
  role: Detects large insertions/deletions from the reference-aligned BAM and records
    them in a VCF
- name: BCFtools
  role: Applies the Sniffles2 indel VCF to the reference to build an indel-corrected
    draft (bcftools consensus, after filtering <INV> records)
- name: Medaka
  role: Primary polishing tool creating the consensus sequence via neural networks
    on a pileup, with model 'auto' or a configured medaka_consensus_model
  repo: https://github.com/nanoporetech/medaka
- name: Samtools
  role: Maps/sorts/indexes reads against the reference and provides the fallback consensus
    (min depth 20, min 70% base proportion, N-masking)
- name: ViMOP
  role: Pipeline orchestrating the reference-guided assembly module within a Nextflow
    workflow
  repo: https://github.com/opr-group-bnitm/vimop
- name: Nextflow
  role: Workflow engine executing the consensus assembly step with a single command
    or via EPI2ME Desktop
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:de7bf6a3d431de210bad7029c133d3bd0b3433fad591cd27479422e298f5500e
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
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# nanopore-consensus-polishing

## Summary

ViMOP's reference-guided consensus assembly: after a BLAST-selected reference is chosen, nanopore reads are mapped, Sniffles2 large indels are merged into the reference with BCFtools consensus, and Medaka (or a Samtools fallback with depth ≥20 and ≥70% base proportion, N-masking below-threshold positions) produces the final consensus FASTA.

## When to use

Use when reconstructing a consensus genome for a known virus from untargeted nanopore reads after a reference genome has been identified, and a polished reference-guided consensus (not de novo contigs) is the deliverable.

## When NOT to use

- The goal is a finished genome for a large DNA virus with extensive repetitive regions (e.g. mpox) — ViMOP output should then only be regarded as a draft genome
- No fitting reference genome can be identified in the virus reference database, since this skill is reference-guided consensus assembly, not de novo assembly
- Reads are not Oxford Nanopore data (e.g. Illumina), as the polishing path depends on nanopore-specific Medaka models

## Inputs

- Trimmed nanopore FASTQ reads for the sample
- BLAST-selected reference genome (FASTA) identified from de novo contigs
- Reference-aligned, sorted and indexed BAM (per-reference)
- Sniffles2 VCF of large indels (with <INV> records pre-filtered)

## Outputs

- Consensus genome FASTA (e.g. AB627954.consensus.fasta) with N-masked low-confidence positions
- Alignment BAM (e.g. AB627954.reads.bam)
- Depth file
- Variant VCF

## How to apply

Map trimmed nanopore reads against the BLAST-selected reference and sort/index with Samtools to get a per-reference BAM. Call large insertions/deletions from the BAM with Sniffles2 into a VCF, then apply them to the reference with BCFtools consensus, pre-filtering <INV> records to avoid a known BCFtools consensus error, yielding an indel-corrected draft. Polish the draft with Medaka using model 'auto' unless a specific medaka_consensus_model is configured; if Medaka cannot fit a model for the data, fall back to Samtools consensus, which requires a minimum depth of 20 and at least 70% base proportion, calling the most abundant base and masking positions below these thresholds with N. Write the final consensus FASTA (e.g. AB627954.consensus.fasta) with the alignment BAM, depth file and variant VCF into the sample's consensus output directory.

## Related tools

- **Sniffles2** (Detects large insertions/deletions from the reference-aligned BAM and records them in a VCF)
- **BCFtools** (Applies the Sniffles2 indel VCF to the reference to build an indel-corrected draft (bcftools consensus, after filtering <INV> records))
- **Medaka** (Primary polishing tool creating the consensus sequence via neural networks on a pileup, with model 'auto' or a configured medaka_consensus_model) — https://github.com/nanoporetech/medaka
- **Samtools** (Maps/sorts/indexes reads against the reference and provides the fallback consensus (min depth 20, min 70% base proportion, N-masking))
- **ViMOP** (Pipeline orchestrating the reference-guided assembly module within a Nextflow workflow) — https://github.com/opr-group-bnitm/vimop
- **Nextflow** (Workflow engine executing the consensus assembly step with a single command or via EPI2ME Desktop) — https://www.nextflow.io/

## Evaluation signals

- Consensus FASTA and alignment BAM both exist in the sample's consensus output directory
- Consensus contains no ambiguous bases other than N-masked positions
- All positions with coverage below the depth-20 threshold or base proportion below 70% are represented as N (Samtools fallback path)
- The VCF applied by BCFtools consensus contained no <INV> records (pre-filtered), so the indel-corrected draft was built without the known BCFtools consensus error
- Consensus should match the BLAST-selected reference length plus/minus the Sniffles2-called large indels

## Limitations

- Unresolved error when <INV>-denoted inversions are present in the VCF file before consensus calling with BCFtools; such records must be filtered beforehand
- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Medaka requires a matching basecaller model; if it cannot fit a model for the data, the pipeline must fall back to the more conservative Samtools consensus path
- Crash when the sequence contains a non-standard nucleotide (not ACGT and not N) — fixed only from ViMOP 1.0.1

## Evidence

- [methods] The reference-guided assembly is a two-stage process: [evidence span withheld — anchor status: not_found]
- [methods] Samtools fallback thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] BCFtools <INV> workaround: "error when inversions denoted with <INV> are in the vcf-file before consensus calling with bcftools"
- [readme] Medaka consensus purpose: [evidence span withheld — anchor status: external_doc]
- [readme] Reference-guided scope: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes"
- [readme] Draft-genome caveat: "for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes"

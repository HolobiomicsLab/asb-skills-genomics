---
name: fasta-reference-sequence-emission
description: Use when de novo assembled contigs (e.g. no-target.contigs.fasta or re-assembly.contigs.fasta
  from Canu) must be matched to known viral references so each unique top hit can
  be written as a reference FASTA for consensus building.
when_to_use_negative:
- The pipeline run is intended to detect novel/unknown viruses — ViMOP only detects
  known virus species, so no database top hit exists to emit
- Large DNA viruses with extensive repetitive regions (e.g. mpox), where the reference-guided
  assembly may yield inaccurate draft genomes
- Reads rather than contigs need classifying or filtering — Centrifuge-based filtering
  acts on reads, not on contig-to-reference assignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: ViMOP
  role: Pipeline implementing the reference-identification process that runs the BLAST
    search and writes the per-contig reference FASTA files
  repo: https://github.com/opr-group-bnitm/vimop
- name: BLAST
  role: Searches each contig against the ViMOP_DB virus reference nucleotide database;
    default ranking selects the top hit per contig
- name: Nextflow
  role: Workflow engine executing the ViMOP reference-identification process within
    the pipeline
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:6b1023580e82173a91f8670f9c707a986e3119f0291f0e0877e551d9aa5ba46b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - BLAST
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fasta-reference-sequence-emission

## Summary

Emits one per-contig virus reference FASTA (e.g. AB627954.reference.fasta) into the ViMOP consensus output directory by selecting the highest-scoring BLAST hit from the ViMOP_DB virus reference database, enabling downstream reference-guided consensus assembly.

## When to use

Use when de novo assembled contigs (e.g. no-target.contigs.fasta or re-assembly.contigs.fasta from Canu) must be matched to known viral references so each unique top hit can be written as a reference FASTA for consensus building.

## When NOT to use

- The pipeline run is intended to detect novel/unknown viruses — ViMOP only detects known virus species, so no database top hit exists to emit
- Large DNA viruses with extensive repetitive regions (e.g. mpox), where the reference-guided assembly may yield inaccurate draft genomes
- Reads rather than contigs need classifying or filtering — Centrifuge-based filtering acts on reads, not on contig-to-reference assignment

## Inputs

- De novo assembled contigs in FASTA format (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta)
- Pre-built BLAST nucleotide database from ViMOP_DB virus reference directory (ALL.nhr/ALL.nsq built from ALL.fasta)

## Outputs

- One reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) in the consensus output directory
- Per-contig hit information table (contigs.tsv) recording the top-scoring BLAST hit for each contig

## How to apply

Load the de novo assembled contigs produced by the upstream Canu assembly steps of the ViMOP Nextflow pipeline and search each contig against the pre-built BLAST nucleotide database (ALL.nhr/ALL.nsq, built from ALL.fasta) in the ViMOP_DB virus reference directory. Select the highest-scoring hit per contig as its reference genome; where scores tie, resolve deterministically using the database's internal ordering. Write one reference FASTA per identified sequence into the consensus output directory and record per-contig hit information in contigs.tsv. The unique reference FASTAs then serve as mapping targets for the reference-guided assembly step that builds consensus genomes from the nanopore reads.

## Related tools

- **ViMOP** (Pipeline implementing the reference-identification process that runs the BLAST search and writes the per-contig reference FASTA files) — https://github.com/opr-group-bnitm/vimop
- **BLAST** (Searches each contig against the ViMOP_DB virus reference nucleotide database; default ranking selects the top hit per contig)
- **Nextflow** (Workflow engine executing the ViMOP reference-identification process within the pipeline) — https://github.com/opr-group-bnitm/vimop

## Evaluation signals

- Every output *.reference.fasta exists in the consensus output directory
- Each emitted *.reference.fasta is valid FASTA
- The reference ID chosen for each contig corresponds to the top-scoring BLAST hit reported in contigs.tsv
- Each unique database hit is used exactly once as the reference for its group of contigs in the subsequent reference-guided assembly

## Limitations

- Only known virus species present in the reference database can be identified; contigs that match no database entry or only partially are flagged via Centrifuge classification
- Centrifuge is limited in accuracy, so contig context may carry false-positive classifications
- For large DNA viruses with extensive repetitive regions (e.g. mpox), reference-guided assemblies may contain inaccuracies and should be regarded as draft genomes

## Evidence

- [methods] Highest scoring BLAST hit selected as reference genome per contig: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [readme] Pipeline automatically finds fitting virus genomes as references: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [abstract] Contigs matched to known viral references feed consensus assembly: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."

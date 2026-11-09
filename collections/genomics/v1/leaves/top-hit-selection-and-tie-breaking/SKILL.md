---
name: top-hit-selection-and-tie-breaking
description: Use when de novo assembled contigs (e.g. from Canu in ViMOP) must be
  matched to known viral reference genomes to enable reference-guided consensus genome
  reconstruction from untargeted nanopore reads.
when_to_use_negative:
- When reads or contigs should not be matched against a known-reference database —
  e.g. the goal is discovery of novel viruses, since ViMOP targets known virus species
- When the target virus is absent from the ViMOP_DB reference database — contigs with
  no database entry will not obtain a reference (Centrifuge classifies such contigs
  as unmatched or partial)
- For large DNA viruses with extensive repetitive regions (e.g. mpox), where the authors
  warn resulting assemblies may be inaccurate drafts
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0781
tools:
- name: ViMOP
  role: Pipeline implementing the reference-identification process that runs BLAST
    and selects the top hit per contig
  repo: https://github.com/opr-group-bnitm/vimop
- name: BLAST
  role: Searches each contig against the virus reference database to produce ranked
    hits
- name: Nextflow
  role: Workflow engine executing the ViMOP reference-identification and downstream
    reference-guided assembly steps
  repo: https://github.com/opr-group-bnitm/vimop
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:9b94c78259c43c4c27fc9ad6e6e61ca4bdd5e0c76bd50e7a85c87aeaf8ee8360
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/opr-group-bnitm/vimop
  tools:
  - BLAST
  - Nextflow
  - ViMOP
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# top-hit-selection-and-tie-breaking

## Summary

Reference identification step of the ViMOP pipeline: each de novo assembled contig is searched by BLAST against the ViMOP_DB virus reference database, and the highest-scoring hit is assigned as that contig's reference genome for downstream reference-guided consensus assembly.

## When to use

Use when de novo assembled contigs (e.g. from Canu in ViMOP) must be matched to known viral reference genomes to enable reference-guided consensus genome reconstruction from untargeted nanopore reads.

## When NOT to use

- When reads or contigs should not be matched against a known-reference database — e.g. the goal is discovery of novel viruses, since ViMOP targets known virus species
- When the target virus is absent from the ViMOP_DB reference database — contigs with no database entry will not obtain a reference (Centrifuge classifies such contigs as unmatched or partial)
- For large DNA viruses with extensive repetitive regions (e.g. mpox), where the authors warn resulting assemblies may be inaccurate drafts

## Inputs

- De novo assembled contigs in FASTA (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta)
- Pre-built BLAST nucleotide virus reference database from ViMOP_DB (ALL.fasta with ALL.nhr/ALL.nsq index files)

## Outputs

- One reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) in the consensus output directory
- contigs.tsv table recording per-contig top-hit information

## How to apply

Load the de novo assembled contigs (e.g. no-target.contigs.fasta or re-assembly.contigs.fasta from the upstream Canu steps of the ViMOP Nextflow pipeline). BLAST each contig against the pre-built nucleotide virus reference database (ALL.nhr/ALL.nsq built from ALL.fasta, BLAST version pinned in images/general/general.yaml of ViMOP_DB). Take the highest-scoring hit per contig as its reference genome; when scores tie, break ties deterministically using the database's internal ordering so results are reproducible. Write one reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) into the consensus output directory and record per-contig hit information in contigs.tsv for use by the subsequent reference-guided assembly.

## Related tools

- **ViMOP** (Pipeline implementing the reference-identification process that runs BLAST and selects the top hit per contig) — https://github.com/opr-group-bnitm/vimop
- **BLAST** (Searches each contig against the virus reference database to produce ranked hits)
- **Nextflow** (Workflow engine executing the ViMOP reference-identification and downstream reference-guided assembly steps) — https://github.com/opr-group-bnitm/vimop

## Examples

```
nextflow run opr-group-bnitm/vimop -profile docker --fastq <fastq_dir>
```

## Evaluation signals

- Every expected output *.reference.fasta exists in the consensus output directory and is valid FASTA
- The reference ID written for each contig corresponds to the top-scoring BLAST hit reported in contigs.tsv
- Re-running the step yields identical reference assignments, since ties are resolved deterministically by internal database ordering
- Each contig that maps to the ViMOP_DB database obtains exactly one assigned reference genome for reference-guided assembly
- Contigs not matching any database entry are flagged (e.g. via the Centrifuge contig classification) rather than silently assigned a wrong reference

## Limitations

- Centrifuge is limited in accuracy and the related classification/filtering may remove some false positives
- Assemblies for large DNA viruses with extensive repetitive regions (e.g. mpox) may contain inaccuracies and should be regarded as draft genomes
- Only hits present in the configured ViMOP_DB reference database can be selected; custom databases must be built for other targets

## Evidence

- [methods] Highest-scoring hit per contig: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [abstract] Contig-to-reference matching purpose: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] Automatic reference finding in README: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [methods] Read mapping for reference-guided assembly: "Reads are mapped against the reference genome."

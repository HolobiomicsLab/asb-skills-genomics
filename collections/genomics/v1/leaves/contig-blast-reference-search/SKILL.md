---
name: contig-blast-reference-search
description: Use when de novo assembled contigs (e.g. from Canu in the ViMOP Nextflow
  pipeline) must be matched to known viral reference genomes before reference-guided
  consensus assembly.
when_to_use_negative:
- Input is raw nanopore reads rather than assembled contigs — ViMOP runs BLAST only
  on de novo assembled contigs, not reads (reads are classified/filtered with Centrifuge
  and mapping instead)
- The target virus has no close representative in the reference database (e.g. a novel
  virus), so no top BLAST hit exists to serve as a reference genome
- The goal is taxonomic classification of reads rather than selecting reference genomes
  for consensus assembly — use the Centrifuge classification step instead
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_0080
tools:
- name: ViMOP
  role: Implements the reference-identification process that BLASTs each contig against
    the virus reference database and writes per-contig reference FASTA files
  repo: https://github.com/opr-group-bnitm/vimop
- name: BLAST
  role: Performs the nucleotide search of each contig against the pre-built ViMOP_DB
    virus reference database; the default BLAST ranking picks the top hit
- name: Nextflow
  role: Workflow engine running the ViMOP pipeline step that executes the BLAST-based
    reference identification
  repo: https://github.com/OPR-group-BNITM/vimop/blob/main/tutorials/01b_installation_tutorial_command_line.md
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:5cac6e60d45b0cbd87ceeb81d715853448056ec3abeb4b1067e72d8191b4917e
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
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# contig-blast-reference-search

## Summary

Identification of reference genomes for de novo assembled viral contigs by BLAST-searching each contig against the ViMOP_DB virus reference database and selecting the highest-scoring hit per contig as the reference for downstream reference-guided consensus assembly.

## When to use

Use when de novo assembled contigs (e.g. from Canu in the ViMOP Nextflow pipeline) must be matched to known viral reference genomes before reference-guided consensus assembly.

## When NOT to use

- Input is raw nanopore reads rather than assembled contigs — ViMOP runs BLAST only on de novo assembled contigs, not reads (reads are classified/filtered with Centrifuge and mapping instead)
- The target virus has no close representative in the reference database (e.g. a novel virus), so no top BLAST hit exists to serve as a reference genome
- The goal is taxonomic classification of reads rather than selecting reference genomes for consensus assembly — use the Centrifuge classification step instead

## Inputs

- De novo assembled contigs FASTA (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta from Canu)
- ViMOP_DB virus reference BLAST nucleotide database (ALL.fasta with ALL.nhr/ALL.nsq index files)

## Outputs

- One reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) in the consensus output directory
- contigs.tsv table recording per-contig BLAST hit information

## How to apply

Load the de novo assembled contigs (e.g. no-target.contigs.fasta / re-assembly.contigs.fasta) produced by the upstream Canu assembly steps of the ViMOP Nextflow pipeline. Run a BLAST search of each contig against the pre-built BLAST nucleotide database (ALL.nhr/ALL.nsq, built from ALL.fasta in the ViMOP_DB virus reference directory, with the BLAST version pinned in images/general/general.yaml) via the ViMOP reference-identification process. Select the highest-scoring hit for each contig as its reference genome; where scores tie, resolve deterministically by the database's internal ordering so results are reproducible. Write one reference FASTA per identified sequence (e.g. AB627954.reference.fasta) into the consensus output directory and record per-contig hit information in the contigs.tsv table. The rationale is that ViMOP matches contigs to known viral references and uses each unique hit to assemble reference-based consensus genomes from untargeted nanopore reads.

## Related tools

- **ViMOP** (Implements the reference-identification process that BLASTs each contig against the virus reference database and writes per-contig reference FASTA files) — https://github.com/opr-group-bnitm/vimop
- **BLAST** (Performs the nucleotide search of each contig against the pre-built ViMOP_DB virus reference database; the default BLAST ranking picks the top hit)
- **Nextflow** (Workflow engine running the ViMOP pipeline step that executes the BLAST-based reference identification) — https://github.com/OPR-group-BNITM/vimop/blob/main/tutorials/01b_installation_tutorial_command_line.md

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --contamination_filters human
```

## Evaluation signals

- Each expected output *.reference.fasta exists in the consensus output directory and is valid FASTA
- The reference ID chosen for each contig corresponds to the top-scoring BLAST hit reported in contigs.tsv
- Every contig with a database hit is assigned exactly one reference genome; ties are resolved identically across re-runs (deterministic internal database ordering)
- Each unique top hit is used downstream, with reads mapped against the corresponding reference genome for reference-guided assembly

## Limitations

- Assemblies (and hence reference matches) may contain inaccuracies for large DNA viruses with extensive repetitive regions such as mpox; outputs should be regarded as draft genomes and reviewed (e.g. alignment .bam files)
- Only known virus species with entries in the ViMOP_DB reference database can be matched; contigs without a database entry are only flagged by Centrifuge classification
- BLAST ranking ties are broken by the database's internal ordering, so the choice of reference for tied hits depends on the database construction

## Evidence

- [methods] Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome.: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes.: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [readme] Centrifuge also classifies the contigs (see later) to get a rough idea about contigs that do not match any entry in the reference database or only partially.: "Centrifuge also classifies the contigs (see later) to get a rough idea about contigs that do not match any entry in the reference database or only partially."

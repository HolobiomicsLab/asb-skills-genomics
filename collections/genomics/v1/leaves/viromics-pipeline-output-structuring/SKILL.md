---
name: viromics-pipeline-output-structuring
description: Use when de novo assembled contigs (e.g. no-target.contigs.fasta or re-assembly.contigs.fasta
  from Canu) must be assigned to known viral reference genomes to guide consensus
  assembly from untargeted nanopore reads.
when_to_use_negative:
- Input is raw nanopore FASTQ reads rather than assembled contigs — ViMOP's reference
  identification operates on contigs after de novo assembly, not directly on reads
- Viruses with no representative entry in the reference database or novel/novel-ish
  pathogens — contigs that match no entry cannot be assigned a reference
- Large DNA viruses with extensive repetitive regions (e.g. mpox) — ViMOP explicitly
  warns such assemblies may be inaccurate draft genomes
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: ViMOP
  role: pipeline whose reference-identification process runs the BLAST search, selects
    top hits, and writes per-contig reference FASTAs and contigs.tsv
  repo: https://github.com/opr-group-bnitm/vimop
- name: BLAST
  role: searches each contig against the ViMOP_DB virus reference nucleotide database;
    highest-scoring hit becomes the reference genome
- name: Nextflow
  role: workflow engine executing the ViMOP pipeline step containing the reference-identification
    process
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:820c149353e03413d69c52303bbac5563eac20e3d2584e1a61e476e70f2cab80
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
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# viromics-pipeline-output-structuring

## Summary

Reconstructs ViMOP's reference-identification module: each de novo assembled contig is BLAST-searched against the ViMOP_DB virus reference database, the highest-scoring hit is adopted as that contig's reference genome, and per-contig FASTA/TSV outputs are structured for downstream reference-guided assembly.

## When to use

Use when de novo assembled contigs (e.g. no-target.contigs.fasta or re-assembly.contigs.fasta from Canu) must be assigned to known viral reference genomes to guide consensus assembly from untargeted nanopore reads.

## When NOT to use

- Input is raw nanopore FASTQ reads rather than assembled contigs — ViMOP's reference identification operates on contigs after de novo assembly, not directly on reads
- Viruses with no representative entry in the reference database or novel/novel-ish pathogens — contigs that match no entry cannot be assigned a reference
- Large DNA viruses with extensive repetitive regions (e.g. mpox) — ViMOP explicitly warns such assemblies may be inaccurate draft genomes

## Inputs

- de novo assembled contigs in FASTA (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta) from Canu assembly steps
- ViMOP_DB pre-built BLAST nucleotide database (ALL.fasta with ALL.nhr/ALL.nsq index files)

## Outputs

- one reference FASTA per identified contig (e.g. AB627954.reference.fasta, CS272305.reference.fasta) in the consensus output directory
- contigs.tsv table recording per-contig BLAST hit information

## How to apply

Load the contigs produced by the upstream Canu de novo assembly steps of the ViMOP Nextflow pipeline. Run a BLAST search of each contig against the pre-built BLAST nucleotide database in the ViMOP_DB virus reference directory (ALL.nhr/ALL.nsq, built from ALL.fasta with the BLAST version pinned in images/general/general.yaml). Select the highest-scoring hit per contig as its reference genome; when scores tie, resolve deterministically using the database's internal ordering so results are reproducible. Write one reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) into the consensus output directory and record per-contig hit information in the contigs.tsv table, which then feeds the reference-guided assembly step where reads are mapped against the reference genome.

## Related tools

- **ViMOP** (pipeline whose reference-identification process runs the BLAST search, selects top hits, and writes per-contig reference FASTAs and contigs.tsv) — https://github.com/opr-group-bnitm/vimop
- **BLAST** (searches each contig against the ViMOP_DB virus reference nucleotide database; highest-scoring hit becomes the reference genome)
- **Nextflow** (workflow engine executing the ViMOP pipeline step containing the reference-identification process)

## Examples

```
nextflow run opr-group-BNITM/vimop --fastq ./fastq -profile docker
```

## Evaluation signals

- Every expected *.reference.fasta output file exists in the consensus output directory
- Each *.reference.fasta is valid FASTA and its sequence identifier corresponds to the top-scoring BLAST hit reported for that contig in contigs.tsv
- Each contig in contigs.tsv has exactly one assigned reference, and tie-broken assignments are reproducible (same database ordering yields the same reference)
- The BLAST database files (ALL.nhr/ALL.nsq) are present and were built from ALL.fasta with the BLAST version pinned in images/general/general.yaml

## Limitations

- The pipeline automatically finds well-fitting virus genomes but cannot assign contigs lacking a match in the reference database; Centrifuge classifies such contigs only roughly
- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes
- Centrifuge-based classification and filtering is limited in accuracy and may remove some false positives

## Evidence

- [methods] Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome.: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."
- [readme] The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes.: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [readme] However, for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes.: "However, for large DNA viruses with extensive repetitive regions (e.g., mpox), assemblies may contain inaccuracies and should be regarded as draft genomes."

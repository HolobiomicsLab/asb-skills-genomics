---
name: viral-reference-database-querying
description: Use when de novo assembled contigs from untargeted nanopore reads must
  be matched to known viral reference genomes so each contig can be assigned a reference
  for consensus genome reconstruction.
when_to_use_negative:
- The goal is taxonomic classification of raw reads rather than assigning reference
  genomes to assembled contigs — Centrifuge-based read classification is used for
  that in ViMOP.
- The virus of interest is large DNA virus with extensive repetitive regions (e.g.
  mpox), where the reference-guided consensus may be inaccurate and results should
  only be regarded as draft genomes.
- No suitable reference exists in the database, since the skill relies on matching
  contigs to known viral references; contigs with no database entry are only roughly
  flagged via Centrifuge contig classification.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0797
tools:
- name: ViMOP
  role: Pipeline whose reference-identification process runs the BLAST search and
    top-hit selection for each contig
  repo: https://github.com/opr-group-bnitm/vimop
- name: BLAST
  role: Searches each contig against the ViMOP_DB virus reference database; default
    ranking with ties resolved by database internal ordering determines the reference
- name: Nextflow
  role: Workflow engine executing the ViMOP reference-identification process within
    the pipeline
  repo: https://www.nextflow.io/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btaf687
    title: 'ViMOP: a user-friendly and field-applicable pipeline for untargeted viral
      genome nanopore sequencing'
  source_content_hash: sha256:1e00b789b333a57af19c3fa03d8d639617e2bac0aca80f2d5658d0e9d2e8350e
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0797
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# viral-reference-database-querying

## Summary

Identify reference genomes for de novo assembled viral contigs by searching each contig against a BLAST nucleotide virus reference database (ViMOP_DB) and selecting the highest-scoring hit per contig for downstream reference-guided assembly.

## When to use

Use when de novo assembled contigs from untargeted nanopore reads must be matched to known viral reference genomes so each contig can be assigned a reference for consensus genome reconstruction.

## When NOT to use

- The goal is taxonomic classification of raw reads rather than assigning reference genomes to assembled contigs — Centrifuge-based read classification is used for that in ViMOP.
- The virus of interest is large DNA virus with extensive repetitive regions (e.g. mpox), where the reference-guided consensus may be inaccurate and results should only be regarded as draft genomes.
- No suitable reference exists in the database, since the skill relies on matching contigs to known viral references; contigs with no database entry are only roughly flagged via Centrifuge contig classification.

## Inputs

- De novo assembled contigs FASTA from Canu (e.g. no-target.contigs.fasta, re-assembly.contigs.fasta)
- Pre-built BLAST nucleotide virus reference database (ALL.nhr/ALL.nsq etc., from ALL.fasta in ViMOP_DB)

## Outputs

- One reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta)
- Per-contig hit information table (contigs.tsv)

## How to apply

Load the de novo assembled contigs (e.g. no-target.contigs.fasta / re-assembly.contigs.fasta) produced by the upstream Canu assembly steps of the ViMOP Nextflow pipeline. Run a BLAST search of each contig against the pre-built BLAST nucleotide database (ALL.nhr/ALL.nsq, built from ALL.fasta with the BLAST version pinned in images/general/general.yaml) in the ViMOP_DB virus reference directory. Select the highest-scoring hit per contig using the default BLAST ranking, resolving ties deterministically by the database's internal ordering. Write one reference FASTA per identified sequence (e.g. AB627954.reference.fasta, CS272305.reference.fasta) into the consensus output directory and record per-contig hit information in contigs.tsv for downstream reference-guided assembly.

## Related tools

- **ViMOP** (Pipeline whose reference-identification process runs the BLAST search and top-hit selection for each contig) — https://github.com/opr-group-bnitm/vimop
- **BLAST** (Searches each contig against the ViMOP_DB virus reference database; default ranking with ties resolved by database internal ordering determines the reference)
- **Nextflow** (Workflow engine executing the ViMOP reference-identification process within the pipeline) — https://www.nextflow.io/

## Examples

```
nextflow run opr-group-bnitm/vimop --fastq <fastq_dir> --outdir <out_dir>
```

## Evaluation signals

- Each expected *.reference.fasta output exists in the consensus output directory.
- Every *.reference.fasta is valid FASTA.
- The reference ID chosen for each contig corresponds to the top-scoring BLAST hit reported in contigs.tsv.
- Each unique top hit is used as the reference for reference-guided assembly of its contig's read set.
- Tied BLAST scores are resolved consistently by internal database ordering (deterministic re-runs).

## Limitations

- For large DNA viruses with extensive repetitive regions (e.g. mpox), assemblies may contain inaccuracies and should be regarded as draft genomes.
- Only the single highest-scoring BLAST hit per contig is used as the reference genome, so lower-scoring but potentially valid hits are ignored.
- Contigs that do not match any entry in the reference database, or only partially, cannot be assigned a reference and are only roughly characterized by Centrifuge contig classification.

## Evidence

- [methods] Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome.: "Each contig is used to for a BLAST search in the virus reference database. The highest scoring hit is then used as a reference genome."
- [readme] The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes.: "The pipeline automatically finds well fitting virus genomes and uses them as references to build reference based consensus genomes."
- [abstract] The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes.: "The software assembles de novo contigs, matches contigs to known viral references and uses them to assemble consensus genomes."

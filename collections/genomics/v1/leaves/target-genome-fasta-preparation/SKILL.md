---
name: target-genome-fasta-preparation
description: Use when a NanoPipe analysis must align MinION reads against a user-supplied
  (custom or downloaded) target FASTA instead of a precompiled target, requiring lastdb
  index construction.
when_to_use_negative:
- A precompiled target for the species of interest already ships with NanoPipe and
  no custom sequence is needed
- The alignment backend is not LAST/NanoPipe (e.g., metagenomics analysis, which NanoPipe
  does not yet support)
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: NanoPipe
  role: Pipeline whose alignment stage consumes the lastdb-built target database;
    custom targets are built during the analysis run
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: lastdb indexes the target FASTA into the sequence database used by NanoPipe's
    alignment (version 946 used for lastal)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:66984674993c613e4c46c44325c7a1d80ad39816fc9669b3f7288f4240f929a0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# target-genome-fasta-preparation

## Summary

Prepares a custom alignment target for NanoPipe by placing a target genome FASTA in NANOPIPE/targets and building a LAST index with lastdb, so that the pipeline's lastal alignment stage can map MinION reads against it.

## When to use

Use when a NanoPipe analysis must align MinION reads against a user-supplied (custom or downloaded) target FASTA instead of a precompiled target, requiring lastdb index construction.

## When NOT to use

- A precompiled target for the species of interest already ships with NanoPipe and no custom sequence is needed
- The alignment backend is not LAST/NanoPipe (e.g., metagenomics analysis, which NanoPipe does not yet support)

## Inputs

- Target genome FASTA file (named 'target.fasta', e.g., Plasmodium falciparum 3D7 genome from plasmodium.tgz or a custom FASTA)

## Outputs

- LAST sequence database index files named 'target' in the target's folder (consumed by the lastal alignment step)

## How to apply

Place the target genome FASTA (e.g., the Plasmodium falciparum 3D7 target available as plasmodium.tgz from the NanoPipe download site, or any custom FASTA) into a subfolder of NANOPIPE/targets and name the file 'target.fasta'. Run the LAST indexer 'NANOPIPE/tools/bin/lastdb target target.fasta' inside that folder to build the LAST sequence database from the FASTA. Confirm the lastdb index files named 'target' were generated; these are the target-creation output that NanoPipe's alignment stage depends on. Before running the downstream lastal alignment, validate the setup with NANOPIPE/check.sh and, if applicable, the Plasmodium testcase alignment run. NanoPipe constructs these target databases as part of the analysis run, which completed in 12 seconds and 51 minutes respectively for two tested custom-target runs.

## Related tools

- **NanoPipe** (Pipeline whose alignment stage consumes the lastdb-built target database; custom targets are built during the analysis run) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (lastdb indexes the target FASTA into the sequence database used by NanoPipe's alignment (version 946 used for lastal))

## Examples

```
NANOPIPE/tools/bin/lastdb target target.fasta
```

## Evaluation signals

- lastdb index files named 'target' exist in the target's subfolder of NANOPIPE/targets after indexing
- NANOPIPE/check.sh reports no errors for the installed packages and software
- The Plasmodium testcase alignment run executes without error before downstream lastal
- The downstream NanoPipe analysis completes successfully (e.g., the P. falciparum 3D7 case mapped 99.9% of query reads)

## Limitations

- Runtime scales with target size: custom-target runs finished in 12 seconds vs. 51 minutes in the article's tests, so large targets (e.g., human hg38) are slow
- File must be named exactly 'target.fasta' inside the target subfolder; naming or placement errors break the pipeline's alignment stage
- NanoPipe does not yet support metagenomics analysis, limiting target preparation to single-target use cases

## Evidence

- [methods] Place the target genome FASTA into a subfolder of NANOPIPE/targets and name the file 'target.fasta'.: [evidence span withheld — anchor status: not_found]
- [methods] Run lastdb inside the target folder to build the LAST sequence database from the FASTA.: [evidence span withheld — anchor status: not_found]
- [methods] For directly uploaded targets, NanoPipe constructs the target databases as part of the analysis run; both tested runs finished successfully.: [evidence span withheld — anchor status: not_found]
- [readme] NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices and provides alignments to any target of interest.: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest, alignment statistics and information about polymorphisms."
- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"

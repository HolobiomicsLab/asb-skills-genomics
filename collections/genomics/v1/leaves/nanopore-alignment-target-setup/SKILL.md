---
name: nanopore-alignment-target-setup
description: Use when a user supplies a custom target FASTA (not a precompiled NanoPipe
  target) and a LAST sequence database plus MinION-tuned scoring parameters must be
  built before running lastal alignment.
when_to_use_negative:
- A precompiled NanoPipe target already exists — it already ships with best-fitting
  last-train-derived alignment parameters and substitution matrices
- The analysis goal is metagenomics analysis — the paper states metagenomics is not
  yet supported and only planned future work
- The downstream goal is SNP detection polishing rather than target preparation —
  use the filter steps (e.g., 20% alternate coverage, 30% of maximum contig coverage)
  instead
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: LAST
  role: Provides lastdb, which builds the LAST sequence database from the target FASTA,
    and last-train, which derives MinION-adjusted scoring parameters consumed by lastal;
    NanoPipe maps reads using version 946 of the LAST sequence aligner
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: NanoPipe
  role: Pipeline that constructs target databases for directly uploaded targets as
    part of the analysis run and invokes lastal with the trained '-p input.substmatrix'
    parameters
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:9aef30cb70e2d08c150f4828927b1cc1ff1c5ce47485bacb4eb47d8330da459d
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# nanopore-alignment-target-setup

## Summary

Constructs a LAST alignment target database (via lastdb) and derives MinION-adjusted alignment scoring parameters (via last-train) so NanoPipe can align Oxford Nanopore MinION reads to a user-supplied target genome.

## When to use

Use when a user supplies a custom target FASTA (not a precompiled NanoPipe target) and a LAST sequence database plus MinION-tuned scoring parameters must be built before running lastal alignment.

## When NOT to use

- A precompiled NanoPipe target already exists — it already ships with best-fitting last-train-derived alignment parameters and substitution matrices
- The analysis goal is metagenomics analysis — the paper states metagenomics is not yet supported and only planned future work
- The downstream goal is SNP detection polishing rather than target preparation — use the filter steps (e.g., 20% alternate coverage, 30% of maximum contig coverage) instead

## Inputs

- Target genome FASTA (e.g., Plasmodium falciparum 3D7 target FASTA, named 'target.fasta')
- MinION sequencing read sample in FASTA or FASTQ format (for last-train parameter estimation)

## Outputs

- lastdb index files named 'target' in the target's subfolder of NANOPIPE/targets
- Trained LAST scoring parameters file input.substmatrix (substitution matrix / scoring scheme)
- Run metadata parameters file input.lastparams

## How to apply

Place the target genome FASTA (e.g., the Plasmodium falciparum 3D7 target available as plasmodium.tgz, or any custom FASTA) into a subfolder of NANOPIPE/targets named 'target.fasta', then run the LAST indexer 'NANOPIPE/tools/bin/lastdb target target.fasta' inside that folder to generate the lastdb index files named 'target'. For MinION-specific scoring, run last-train (installed at NANOPIPE/tools/bin/last-train) with the lastdb target index and a FASTA/FASTQ sample of MinION reads to estimate insertion, deletion, and substitution rates; save the trained scheme as input.substmatrix (a non-empty plain-text LAST parameters file) and record parameters in input.lastparams. The substitution matrix encodes the mismatch cost between each pair of nucleotides, and NanoPipe's precompiled targets ship with such last-train-derived parameters. The trained matrix is then passed to lastal via '-p input.substmatrix'. Validate by confirming the index files exist and that NANOPIPE/check.sh plus a test-case alignment (e.g., the Plasmodium testcase, ~65 s total) run without error.

## Related tools

- **LAST** (Provides lastdb, which builds the LAST sequence database from the target FASTA, and last-train, which derives MinION-adjusted scoring parameters consumed by lastal; NanoPipe maps reads using version 946 of the LAST sequence aligner) — https://github.com/IOB-Muenster/nanopipe2
- **NanoPipe** (Pipeline that constructs target databases for directly uploaded targets as part of the analysis run and invokes lastal with the trained '-p input.substmatrix' parameters) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
NANOPIPE/tools/bin/lastdb target target.fasta && NANOPIPE/tools/bin/last-train target minion_reads.fasta > input.substmatrix
```

## Evaluation signals

- lastdb index files named 'target' exist in the target's folder after indexing
- input.substmatrix exists as a non-empty plain-text LAST parameters file that lastal accepts via -p without error
- NANOPIPE/check.sh reports all installed packages and software without error
- The Plasmodium testcase alignment completes successfully (test-case pipeline ~65 seconds total; directly uploaded-target runs finished in 12 seconds and 51 minutes including target database construction)
- Downstream lastal maps reads to the target (Plasmodium study case mapped 99.9% of query reads)

## Limitations

- Building a target database manually requires placing 'target.fasta' in a subfolder of NANOPIPE/targets; alternatively a script can be used, so path/naming mistakes break the pipeline
- Precompiled targets are limited to those shipped with NanoPipe; custom targets must be indexed locally, and training requires a representative MinION read sample
- Neither the README nor the article states target database versioning or a changelog ('No changelog found.'), so index provenance must be tracked manually

## Evidence

- [methods] For directly uploaded targets, NanoPipe constructs the target databases as part of the analysis run; both tested runs finished successfully in 12 seconds and in 51 minutes: [evidence span withheld — anchor status: not_found]
- [methods] Run the LAST indexer 'lastdb target target.fasta' inside the target's folder to build the LAST sequence database: [evidence span withheld — anchor status: not_found]
- [methods] the substitution matrix encodes the mismatch cost between each pair of nucleotides, so the training step supplies the MinION-adjusted scoring parameters used for alignment: [evidence span withheld — anchor status: not_found]
- [methods] You can create a target database either with the help of a script or manually.: "You can create a target database either with the help of a script or manually."
- [readme] NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest"
- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"

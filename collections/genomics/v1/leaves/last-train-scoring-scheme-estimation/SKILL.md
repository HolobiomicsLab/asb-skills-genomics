---
name: last-train-scoring-scheme-estimation
description: Use when aligning Oxford Nanopore MinION reads (FASTA/FASTQ) to a target
  sequence and you need a substitution matrix and scoring scheme tuned to MinION error
  rates rather than generic LAST defaults.
when_to_use_negative:
- The aligner is not LAST-based or the target index was not built with lastdb, since
  the -p substitution-matrix input is LAST-specific
- Reads are not nanopore MinION data — the trained parameters encode MinION-specific
  insertion/deletion/substitution rates and may mis-score other platforms
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: last-train
  role: Estimates MinION-specific insertion, deletion, and substitution rates from
    the target index and read sample, emitting the trained scoring scheme saved as
    input.substmatrix
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: LAST
  role: Alignment engine whose lastal step consumes the trained matrix via -p; NanoPipe
    used LAST version 946 for mapping MinION reads to targets
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: NanoPipe
  role: Pipeline that packages last-train-derived parameters with its precompiled
    targets so MinION reads are aligned with technology-adjusted scoring
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:8f4006553213b4629e12b0b730ab743bd13e7a8da10d4e34c17e828cf776a553
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IOB-Muenster/nanopipe2
  tools:
  - LAST
  - NanoPipe
  - last-train
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# last-train-scoring-scheme-estimation

## Summary

Trains LAST alignment scoring parameters and a substitution matrix on MinION sequencing reads against a target index using last-train, so that read-to-target alignments (as used by NanoPipe) are scored with insertion, deletion, and mismatch costs adjusted to nanopore error characteristics.

## When to use

Use when aligning Oxford Nanopore MinION reads (FASTA/FASTQ) to a target sequence and you need a substitution matrix and scoring scheme tuned to MinION error rates rather than generic LAST defaults.

## When NOT to use

- The aligner is not LAST-based or the target index was not built with lastdb, since the -p substitution-matrix input is LAST-specific
- Reads are not nanopore MinION data — the trained parameters encode MinION-specific insertion/deletion/substitution rates and may mis-score other platforms

## Inputs

- Prebuilt LAST target index (lastdb output, e.g., NANOPIPE/targets/<targetname>/target) created from a target FASTA
- FASTA or FASTQ sample of MinION sequencing reads used as training input

## Outputs

- input.substmatrix: non-empty plain-text LAST parameters file containing the trained substitution/scoring scheme
- input.lastparams: run metadata file recording the derived alignment parameters

## How to apply

First build a LAST index of the target with 'lastdb target target.fasta' (stored, e.g., at NANOPIPE/targets/<targetname>/target). Then run last-train (installed at NANOPIPE/tools/bin/last-train per the NanoPipe2 install procedure) with the target index and a sample of the MinION reads as input, so it estimates the insertion, deletion, and substitution rates characteristic of the MinION data. Save the trained scoring scheme to a plain-text parameters file input.substmatrix and record the derived parameters alongside run metadata in input.lastparams. The substitution matrix encodes the mismatch cost between each pair of nucleotides; downstream lastal invocations consume it via the '-p input.substmatrix' flag, which is how NanoPipe ships precompiled targets with MinION-adjusted alignment parameters.

## Related tools

- **last-train** (Estimates MinION-specific insertion, deletion, and substitution rates from the target index and read sample, emitting the trained scoring scheme saved as input.substmatrix) — https://github.com/IOB-Muenster/nanopipe2
- **LAST** (Alignment engine whose lastal step consumes the trained matrix via -p; NanoPipe used LAST version 946 for mapping MinION reads to targets) — https://github.com/IOB-Muenster/nanopipe2
- **NanoPipe** (Pipeline that packages last-train-derived parameters with its precompiled targets so MinION reads are aligned with technology-adjusted scoring) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
NANOPIPE/tools/bin/last-train NANOPIPE/targets/<targetname>/target minion_reads.fastq > input.substmatrix; lastal -p input.substmatrix NANOPIPE/targets/<targetname>/target minion_reads.fastq
```

## Evaluation signals

- input.substmatrix exists as a non-empty plain-text LAST parameters file
- The file can be passed to lastal via -p without error
- The reference test-case pipeline (including lastal with -p input.substmatrix) completes in about 65 seconds total
- input.lastparams records the derived parameters alongside the run metadata
- Downstream NanoPipe mapping achieves expected sensitivity (e.g., 99.9% of query reads mapped in the Plasmodium study case)

## Limitations

- Training requires a representative sample of the MinION reads; a non-representative sample yields parameters not adjusted to the actual sequencing run
- Precompiled NanoPipe targets ship with precomputed best-fitting parameters; custom targets require running this training step manually

## Evidence

- [abstract] NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
- [methods] mapping with LAST 946: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [readme] train on a read sample: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest"
- [methods] input formats: "(1) data uploading in FASTA or FASTQ formats"

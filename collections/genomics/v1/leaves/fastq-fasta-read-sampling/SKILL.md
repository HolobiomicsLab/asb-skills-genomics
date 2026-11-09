---
name: fastq-fasta-read-sampling
description: Use when a MinION nanopore read set in FASTA or FASTQ format needs a
  trained LAST scoring scheme (substitution matrix / parameters) fitted to the specific
  target before mapping with lastal, instead of relying on default or generic scoring.
when_to_use_negative:
- The target is one of NanoPipe's precompiled targets, which already ship with best-fitting
  last-train-derived parameters and substitution matrices, so retraining is redundant
- The reads do not come from Oxford Nanopore MinION sequencing — the estimated insertion/deletion/substitution
  rates would not reflect the sequencing technology in use
- The goal is metagenomics analysis, which NanoPipe explicitly does not yet support
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: last-train
  role: Estimates MinION-specific insertion, deletion, and substitution rates from
    the read sample against the lastdb target index and emits the trained scoring
    scheme
- name: LAST
  role: Alignment suite whose aligner (lastal, version 946 in NanoPipe) consumes the
    trained matrix via '-p input.substmatrix' to map MinION reads to the target
- name: NanoPipe
  role: Pipeline that provides the prebuilt lastdb targets under NANOPIPE/targets/,
    installs last-train at NANOPIPE/tools/bin/, and defines the input.substmatrix
    / input.lastparams file conventions
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:af4e6ee10a7e3bec0b127f23a5839026e4652d048853a0022aa9856107185848
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
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# fastq-fasta-read-sampling

## Summary

Sample a subset of MinION sequencing reads (FASTA or FASTQ) and train LAST's scoring scheme with last-train against a prebuilt lastdb target index, so the alignment parameters reflect the insertion, deletion, and substitution rates characteristic of MinION data rather than a generic matrix.

## When to use

Use when a MinION nanopore read set in FASTA or FASTQ format needs a trained LAST scoring scheme (substitution matrix / parameters) fitted to the specific target before mapping with lastal, instead of relying on default or generic scoring.

## When NOT to use

- The target is one of NanoPipe's precompiled targets, which already ship with best-fitting last-train-derived parameters and substitution matrices, so retraining is redundant
- The reads do not come from Oxford Nanopore MinION sequencing — the estimated insertion/deletion/substitution rates would not reflect the sequencing technology in use
- The goal is metagenomics analysis, which NanoPipe explicitly does not yet support

## Inputs

- FASTA or FASTQ file containing a sample of MinION sequencing reads
- Prebuilt lastdb target index (e.g., NANOPIPE/targets/<targetname>/target created with 'lastdb target target.fasta')

## Outputs

- input.substmatrix — non-empty plain-text LAST parameters file holding the trained substitution/scoring scheme, passable to lastal via -p
- input.lastparams — run metadata recording the derived parameters

## How to apply

Load the prebuilt lastdb target index (e.g., NANOPIPE/targets/<targetname>/target, built with 'lastdb target target.fasta') and a FASTA or FASTQ sample of the MinION reads. Execute last-train (installed at NANOPIPE/tools/bin/last-train per the NanoPipe2 install procedure) with the target index and the read sample as input, so it estimates the insertion, deletion, and substitution rates characteristic of the MinION data for that target. Save the resulting trained scoring scheme to the parameters file input.substmatrix — its substitution matrix encodes the mismatch cost between each pair of nucleotides — and record the run metadata alongside it in input.lastparams. Validate that input.substmatrix exists as a non-empty plain-text LAST parameters file and that lastal accepts it via the '-p input.substmatrix' option without error.

## Related tools

- **last-train** (Estimates MinION-specific insertion, deletion, and substitution rates from the read sample against the lastdb target index and emits the trained scoring scheme)
- **LAST** (Alignment suite whose aligner (lastal, version 946 in NanoPipe) consumes the trained matrix via '-p input.substmatrix' to map MinION reads to the target)
- **NanoPipe** (Pipeline that provides the prebuilt lastdb targets under NANOPIPE/targets/, installs last-train at NANOPIPE/tools/bin/, and defines the input.substmatrix / input.lastparams file conventions) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
last-train NANOPIPE/targets/<targetname>/target reads_sample.fastq > input.substmatrix
```

## Evaluation signals

- input.substmatrix exists and is a non-empty plain-text LAST parameters file
- lastal runs without error when invoked with '-p input.substmatrix' against the target index
- The full NanoPipe2 test-case pipeline (which uses the trained matrix) completes in about 65 seconds total
- The trained substitution matrix encodes per-nucleotide-pair mismatch costs rather than LAST defaults, reflecting MinION substitution rates
- input.lastparams is present alongside input.substmatrix so run metadata records the derived parameters

## Limitations

- Training is target-specific: the derived scoring scheme fits the given lastdb target and the sampled MinION read set, so it may not transfer to other targets or sequencing runs
- The substitution matrix only encodes mismatch costs between nucleotide pairs; insertion and deletion behavior comes from the rates estimated by last-train and depends on the representativeness of the read sample
- NanoPipe's SNP detection statistics and scope are limited (no metagenomics option yet), so the trained alignment is intended for targeted MinION resequencing workflows

## Evidence

- [methods] mapping of sequencing reads to the target using version 946 of the LAST sequence aligner: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [methods] data uploading in FASTA or FASTQ formats: "NanoPipe can be conceptually divided into four stages: (1) data uploading in FASTA or FASTQ formats"
- [readme] NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices."
- [abstract] NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"

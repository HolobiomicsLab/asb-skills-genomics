---
name: minion-error-model-calibration
description: Use when building or updating a NanoPipe target whose alignments against
  MinION reads should use MinION-adjusted scoring, e.g. before running lastal with
  a trained parameters file via the -p option.
when_to_use_negative:
- Input reads are not MinION nanopore data — the trained parameters are specifically
  adjusted to MinION sequencing specifics and would mis-score other platforms
- A precompiled NanoPipe target already ships with last-train-derived parameters and
  the target/reference has not changed
- The task is alignment analysis or consensus/SNP calling (e.g. 20% alternate coverage
  SNP candidates, majority-rule consensus) rather than producing the scoring parameters
  themselves
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LAST
  role: Alignment engine whose trained scoring parameters (passed to lastal via -p
    input.substmatrix) enable MinION-adjusted mapping of reads to the target; NanoPipe
    uses version 946 of the LAST sequence aligner
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: last-train
  role: Estimates insertion, deletion, and substitution rates characteristic of MinION
    data from a target index and read sample; installed at NANOPIPE/tools/bin/last-train
  repo: https://github.com/IOB-Muenster/nanopipe2
- name: NanoPipe
  role: Pipeline whose precompiled targets carry best-fitting alignment parameters
    and substitution matrices derived from last-train calculations; the trained matrix
    feeds its alignment stage
  repo: https://github.com/IOB-Muenster/nanopipe2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giy169
    title: NanoPipe—a web server for nanopore MinION sequencing data analysis
  source_content_hash: sha256:6031c5bc35befe34220c587b4bb4e9cb289002c40210115f84944cb5c10b8b54
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# minion-error-model-calibration

## Summary

Trains LAST alignment scoring parameters (substitution matrix with per-nucleotide-pair mismatch costs plus insertion/deletion rates) on a sample of MinION reads so that downstream alignment in NanoPipe reflects the error profile of Oxford Nanopore sequencing rather than a generic scoring scheme.

## When to use

Use when building or updating a NanoPipe target whose alignments against MinION reads should use MinION-adjusted scoring, e.g. before running lastal with a trained parameters file via the -p option.

## When NOT to use

- Input reads are not MinION nanopore data — the trained parameters are specifically adjusted to MinION sequencing specifics and would mis-score other platforms
- A precompiled NanoPipe target already ships with last-train-derived parameters and the target/reference has not changed
- The task is alignment analysis or consensus/SNP calling (e.g. 20% alternate coverage SNP candidates, majority-rule consensus) rather than producing the scoring parameters themselves

## Inputs

- Prebuilt lastdb target index (NANOPIPE/targets/<targetname>/target built with lastdb from target.fasta)
- Sample of MinION sequencing reads in FASTA or FASTQ format

## Outputs

- input.substmatrix — non-empty plain-text LAST parameters file containing the trained substitution matrix/scoring scheme
- input.lastparams — run metadata recording the derived parameters

## How to apply

Load the prebuilt lastdb target index (e.g. NANOPIPE/targets/<targetname>/target created with 'lastdb target target.fasta') and a FASTA or FASTQ sample of the MinION reads. Run last-train (installed at NANOPIPE/tools/bin/last-train per the NanoPipe2 install procedure) with the target index and the read sample as input so it estimates MinION-characteristic insertion, deletion, and substitution rates. Save the resulting trained scoring scheme as a plain-text LAST parameters file input.substmatrix, which the substitution matrix component encodes as the mismatch cost between each pair of nucleotides, and record the run metadata in input.lastparams. At alignment time, pass the trained file to lastal via '-p input.substmatrix'; NanoPipe's precompiled targets already ship with such best-fitting last-train-derived parameters.

## Related tools

- **LAST** (Alignment engine whose trained scoring parameters (passed to lastal via -p input.substmatrix) enable MinION-adjusted mapping of reads to the target; NanoPipe uses version 946 of the LAST sequence aligner) — https://github.com/IOB-Muenster/nanopipe2
- **last-train** (Estimates insertion, deletion, and substitution rates characteristic of MinION data from a target index and read sample; installed at NANOPIPE/tools/bin/last-train) — https://github.com/IOB-Muenster/nanopipe2
- **NanoPipe** (Pipeline whose precompiled targets carry best-fitting alignment parameters and substitution matrices derived from last-train calculations; the trained matrix feeds its alignment stage) — https://github.com/IOB-Muenster/nanopipe2

## Examples

```
NANOPIPE/tools/bin/last-train NANOPIPE/targets/<targetname>/target reads_sample.fastq > input.substmatrix; lastal -p input.substmatrix NANOPIPE/targets/<targetname>/target reads_sample.fastq
```

## Evaluation signals

- input.substmatrix exists as a non-empty plain-text LAST parameters file that can be passed to lastal via -p without error
- The trained matrix encodes a mismatch cost for each pair of nucleotides (complete substitution matrix)
- input.lastparams is produced alongside input.substmatrix as run metadata
- Downstream lastal alignment with the trained parameters completes successfully (the reference test-case pipeline runs in about 65 seconds total)
- Mapped reads against the target behave consistently with MinION error characteristics, e.g. NanoPipe succeeding in mapping ~99.9% of query reads in the Plasmodium study case

## Limitations

- Parameters are MinION-specific: the article's premise is alignment parameters 'adjusted for MinION sequencing technology specifics', so the trained matrix is not transferable to other sequencing platforms
- Trained parameters depend on the target: precompiled targets ship with their own best-fitting parameters, so a new or altered target requires retraining
- NanoPipe itself does not yet implement more sophisticated statistics for SNP detection and evaluation, so trained-parameter alignment quality must be interpreted through its simpler p-error / coverage heuristics

## Evidence

- [other] precompiled targets ship with best-fitting alignment parameters: [evidence span withheld — anchor status: not_found]
- [other] last-train estimates MinION rates: [evidence span withheld — anchor status: not_found]
- [abstract] adjusted alignment parameters: "NanoPipe was developed in consideration of the specifics of the MinION sequencing technologies, providing accordingly adjusted alignment parameters"
- [methods] LAST aligner version 946: "mapping of sequencing reads to the target using version 946 of the LAST sequence aligner"
- [readme] README scope statement: "NanoPipe analyzes reads generated by the Oxford Nanopore sequencing devices. As a result, it provides alignments to any target of interest"

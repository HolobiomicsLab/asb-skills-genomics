---
name: nanopore-reference-alignment-parameter-tuning
description: Use when raw multi-fast5 ONT direct RNA reads must be converted to single-read
  fast5 and raw current signal aligned (resquiggled) to a reference FASTA before per-position
  modification detection, especially when tuning alignment/event-detection parameters
  for rRNA or similar references.
when_to_use_negative:
- If reads are already single-read fast5 and resquiggled, re-running conversion/resquiggle
  is unnecessary and risks overwriting with --overwrite
- When the target requires a trained kmer model for modification identification without
  an IVT control — the paper notes no such training sets exist and detection here
  depends on comparison with an IVT control
- When the goal is basecalling or alignment of cDNA/DNA nanopore reads rather than
  raw RNA signal-to-sequence alignment for modification detection
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0099
tools:
- name: multi_to_single_fast5
  role: Converts multi-read fast5 files to single-read fast5 files with --recursive,
    --fast5_out, --flowcell and --kit so tombo can process individual reads
  repo: https://github.com/physnano/rRNA_nanoSHAPE
- name: tombo
  role: Performs raw-signal-to-sequence alignment (resquiggle) of nanopore ionic current
    events to the reference FASTA with the paper's flag set
  repo: https://github.com/physnano/rRNA_nanoSHAPE
- name: tombo detect_modifications model_sample_compare
  role: Downstream consumer of resquiggled fast5 directories; computes per-position
    KS statistics between sample and IVT control signal distributions
  repo: https://github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:d13ff8c990961778a020c3c275cc7959143f93070126a66a434e10b8c183dc71
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  repo_url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - multi_to_single_fast5
  - tombo
  - tombo detect_modifications model_sample_compare
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nanopore-reference-alignment-parameter-tuning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A resquiggling/alignment-parameterization recipe for ONT direct RNA nanopore data: converting multi-fast5 files to single-read fast5 and aligning raw ionic-current signal to a reference sequence with Tombo's resquiggle using the paper's validated flag set, so that subsequent modification detection (KS-based sample comparison) operates on correctly signal-to-sequence-aligned reads.

## When to use

Use when raw multi-fast5 ONT direct RNA reads must be converted to single-read fast5 and raw current signal aligned (resquiggled) to a reference FASTA before per-position modification detection, especially when tuning alignment/event-detection parameters for rRNA or similar references.

## When NOT to use

- If reads are already single-read fast5 and resquiggled, re-running conversion/resquiggle is unnecessary and risks overwriting with --overwrite
- When the target requires a trained kmer model for modification identification without an IVT control — the paper notes no such training sets exist and detection here depends on comparison with an IVT control
- When the goal is basecalling or alignment of cDNA/DNA nanopore reads rather than raw RNA signal-to-sequence alignment for modification detection

## Inputs

- Multi-read ONT fast5 files from native (modified) direct RNA sequencing runs
- Multi-read ONT fast5 files from IVT (unmodified control) direct RNA sequencing runs
- Reference FASTA of the target RNA (e.g., E. coli or S. cerevisiae rRNA sequence)
- Flowcell and kit identifiers for the sequenced run (required by multi_to_single_fast5)

## Outputs

- Single-read fast5 files (--fast5_out) organized per flowcell/kit
- Resquiggled fast5 files with raw-signal-to-sequence alignment, including per-event standard deviation from --include-event-stdev
- Aligned fast5 directories ready as --fast5-basedirs / --control-fast5-basedirs inputs for tombo detect_modifications

## How to apply

First convert multi-read fast5 files to per-read format with multi_to_single_fast5, passing --recursive, --fast5_out, and the correct --flowcell and --kit for the sequenced run. Then run 'tombo resquiggle' against the rRNA FASTA reference with the validated flag set: --include-event-stdev (so event standard deviations are retained for downstream testing), --num-most-common-errors 5, --ignore-read-locks, --overwrite, and multiple --processes for parallelism. Resquiggled fast5s become the --fast5-basedirs (modified sample) and --control-fast5-basedirs (IVT control) inputs for 'tombo detect_modifications model_sample_compare' with --minimum-test-reads 100 and --multiprocess-region-size 250. Correct alignment is prerequisite to localization: the paper shows this parameterization yields KS-statistic peaks within ±2 nt of known 2'-O-methyl and base modification sites in E. coli and S. cerevisiae rRNA.

## Related tools

- **multi_to_single_fast5** (Converts multi-read fast5 files to single-read fast5 files with --recursive, --fast5_out, --flowcell and --kit so tombo can process individual reads) — https://github.com/physnano/rRNA_nanoSHAPE
- **tombo** (Performs raw-signal-to-sequence alignment (resquiggle) of nanopore ionic current events to the reference FASTA with the paper's flag set) — https://github.com/physnano/rRNA_nanoSHAPE
- **tombo detect_modifications model_sample_compare** (Downstream consumer of resquiggled fast5 directories; computes per-position KS statistics between sample and IVT control signal distributions) — https://github.com/physnano/rRNA_nanoSHAPE

## Examples

```
tombo resquiggle <single-fast5-dir> <rRNA.fasta> --include-event-stdev --num-most-common-errors 5 --ignore-read-locks --overwrite --processes 8
```

## Evaluation signals

- Resquiggle completes over the full fast5 directory without errors when run with --num-most-common-errors 5 and --overwrite; error counts reported for the 5 most common failure classes are reviewed
- Resquiggled fast5 files retain event standard deviation (--include-event-stdev) so downstream model_sample_compare can use signal distributions
- Downstream KS-test peaks fall within ±2 nt of known rRNA modification positions (the paper's success criterion for E. coli and S. cerevisiae)
- --minimum-test-reads of 100 is met at tested positions (per the paper's typical setting for in vitro/control comparisons)
- Alignment accuracy is consistent with the paper's finding that both Tombo and Nanopolish raw-signal alignment identified rRNA modification positions to within ±2 nt of known modified sites

## Limitations

- Signal discrimination remains an outstanding challenge: it is a function of kmer sequence context and the ability to align raw current
- Modification identification without an IVT control requires training sets of known modifications in all possible kmer contexts, which do not yet exist
- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates
- Poor resolution of reactivity profiles at the 5' ends of longer RNA molecules

## Evidence

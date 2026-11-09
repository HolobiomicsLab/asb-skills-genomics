---
name: rna-modification-ks-sample-compare-testing
description: Use when raw ionic-current signal from native and IVT-control direct
  RNA nanopore reads (multi-fast5 converted to per-read fast5) must be compared to
  localize endogenous 2'-O-methyl or base modifications per nucleotide.
when_to_use_negative:
- When no IVT (unmodified) control sample exists for the target RNA — Tombo's sample-compare
  framework requires a control signal distribution; de novo identification without
  an IVT control needs training sets of known modifications in all kmer contexts,
  which do not yet exist
- When the target is low-concentration RNA or lacks a free 3'-hydroxyl for adaptor
  ligation — the method requires high concentrations of target RNA and a ligate-able
  3'-OH end
- When the question is RNA structure (SHAPE reactivity) rather than modification localization
  — use the nanoSHAPE/extract_array.py workflow instead
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3524
tools:
- name: multi_to_single_fast5
  role: Converts multi-read fast5 files to per-read fast5 files required by Tombo,
    with --recursive and --fast5_out and flowcell/kit specs for the sequenced run
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo resquiggle
  role: Aligns raw ionic-current signal to the rRNA FASTA reference sequence, with
    --include-event-stdev to retain event standard deviations used in KS testing
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo detect_modifications model_sample_compare
  role: 'Core KS-test step: computes per-position KS statistics between the native
    sample''s fast5-basedirs and the IVT control-fast5-basedirs, with --minimum-test-reads
    100 and --multiprocess-region-size 250; re-run with --per-read-statistics-basename
    for per-read output'
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo plot per_read
  role: Visualizes per-read modification statistics at target genomic locations from
    the per-read statistics file, as a PDF
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:c0ee99b510b68839c6d7c74b7b4e015203fdfb53c2cfd0164bde33c4447123c4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - multi_to_single_fast5
  - tombo
  - tombo detect_modifications model_sample_compare
  - tombo plot per_read
  - tombo resquiggle
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7222222222222222
n_claims: 18
---

# rna-modification-ks-sample-compare-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects RNA modifications in direct RNA nanopore sequencing by applying Tombo's model_sample_compare, which computes per-position Kolmogorov-Smirnov statistics between the ionic-current signal distributions of a native sample and an IVT (unmodified) control, localizing modifications to within ±2 nt of known sites in E. coli and S. cerevisiae rRNAs.

## When to use

Use when raw ionic-current signal from native and IVT-control direct RNA nanopore reads (multi-fast5 converted to per-read fast5) must be compared to localize endogenous 2'-O-methyl or base modifications per nucleotide.

## When NOT to use

- When no IVT (unmodified) control sample exists for the target RNA — Tombo's sample-compare framework requires a control signal distribution; de novo identification without an IVT control needs training sets of known modifications in all kmer contexts, which do not yet exist
- When the target is low-concentration RNA or lacks a free 3'-hydroxyl for adaptor ligation — the method requires high concentrations of target RNA and a ligate-able 3'-OH end
- When the question is RNA structure (SHAPE reactivity) rather than modification localization — use the nanoSHAPE/extract_array.py workflow instead

## Inputs

- Native (modified) direct RNA nanopore multi-fast5 files (e.g. E. coli or S. cerevisiae rRNA)
- IVT unmodified control direct RNA nanopore multi-fast5 files
- rRNA FASTA reference sequence
- Known rRNA modification annotation (for validating ±2 nt peak localization)

## Outputs

- Per-position KS statistics file comparing native vs. IVT ionic-current distributions
- Per-read statistics file (via --per-read-statistics-basename)
- List of significant KS-peak positions corresponding to candidate modification sites
- Optional tombo per-read PDF plots at genomic locations of target sites

## How to apply

Convert multi-fast5 files to single-read fast5 with multi_to_single_fast5 (--recursive, --fast5_out, specifying --flowcell and --kit), then resquiggle raw signal to the rRNA FASTA reference with 'tombo resquiggle' using --include-event-stdev, --num-most-common-errors 5, --ignore-read-locks, and --overwrite. Run 'tombo detect_modifications model_sample_compare' with the native fast5 directory as --fast5-basedirs and the IVT fast5 directory as --control-fast5-basedirs, setting --minimum-test-reads to 100 (the recommended in vitro minimum) and --multiprocess-region-size 250; this computes a per-position KS statistic comparing the sample's signal distribution against the control's. Re-run the same command adding --per-read-statistics-basename to emit a per-read statistics file usable for 'tombo plot per_read' visualization. Extract significant KS-peak positions from the statistics output and tabulate them against the known rRNA modification map: a call is validated if the KS-test peak falls within ±2 nt of a known modified site.

## Related tools

- **multi_to_single_fast5** (Converts multi-read fast5 files to per-read fast5 files required by Tombo, with --recursive and --fast5_out and flowcell/kit specs for the sequenced run) — github.com/physnano/rRNA_nanoSHAPE
- **tombo resquiggle** (Aligns raw ionic-current signal to the rRNA FASTA reference sequence, with --include-event-stdev to retain event standard deviations used in KS testing) — github.com/physnano/rRNA_nanoSHAPE
- **tombo detect_modifications model_sample_compare** (Core KS-test step: computes per-position KS statistics between the native sample's fast5-basedirs and the IVT control-fast5-basedirs, with --minimum-test-reads 100 and --multiprocess-region-size 250; re-run with --per-read-statistics-basename for per-read output) — github.com/physnano/rRNA_nanoSHAPE
- **tombo plot per_read** (Visualizes per-read modification statistics at target genomic locations from the per-read statistics file, as a PDF) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
tombo detect_modifications model_sample_compare --fast5-basedirs <MODIFIED fast5 dir> --control-fast5-basedirs <IVT fast5 dir> --statistics-file-basename ks_stats --minimum-test-reads 100 --multiprocess-region-size 250 --per-read-statistics-basename per_read_stats
```

## Evaluation signals

- KS statistic peaks fall within ±2 nt of known modified sites in the E. coli and S. cerevisiae rRNA modification annotation — the paper's stated success criterion
- Each tested position has at least --minimum-test-reads (100, the recommended in vitro minimum) reads in both sample and control; positions below this threshold are not called
- The statistics file and per-read statistics file are produced with the expected basenames from --statistics-file-basename and --per-read-statistics-basename
- Resquiggling completes with --num-most-common-errors 5 without excessive alignment errors; reads retain event stdev (--include-event-stdev) needed for testing
- Per-read plots (tombo plot per_read) show signal-level differences at called sites relative to the IVT control

## Limitations

- Signal discrimination depends on kmer sequence context and the ability to align raw current; high kmer-complexity regions may fail
- No training sets of known modifications in all possible kmer sequence contexts exist, so identification without an IVT control is not possible with this approach
- The method requires high concentrations of target RNA and a free 3'-hydroxyl for ligation to the first sequencing adaptor
- Signal-level calls may be offset from the true modified base (hence the ±2 nt tolerance rather than exact nt resolution)

## Evidence

- [methods] KS statistic peaks within ±2 nt of known modification sites: [evidence span withheld — anchor status: not_found]

---
name: rrna-modification-site-annotation
description: Use when you have native (modified) and IVT (unmodified) direct RNA nanopore
  fast5 datasets for a transcript with known modification annotations (e.g. rRNA Nm,
  Ψ, base sites) and need to call/localize modification positions from raw-signal
  KS statistics.
when_to_use_negative:
- You have no IVT/unmodified control sample — the paper notes identification without
  an IVT comparison would require training sets of known modifications in all possible
  kmer sequence contexts, which do not yet exist.
- Your goal is de novo modification classification independent of prior annotation
  rather than localizing sites to a known rRNA modification map.
- Input is already a per-position statistics table requiring no raw-signal processing
  (use downstream profiling/correlation skills instead).
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: multi_to_single_fast5
  role: Converts multi-read FAST5 files to per-read single FAST5 files (with --recursive,
    --fast5_out, --flowcell and --kit of the sequenced run) as prerequisite for tombo
    processing.
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo resquiggle
  role: Aligns raw nanopore signal to the rRNA FASTA reference per read (--include-event-stdev,
    --num-most-common-errors 5, --ignore-read-locks, --overwrite, multiple --processes)
    to enable per-position signal comparison.
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo detect_modifications model_sample_compare
  role: Computes per-position KS statistics between native and IVT fast5 signal distributions
    (--minimum-test-reads 100, --multiprocess-region-size 250); re-run with --per-read-statistics-basename
    for per-read stats.
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: tombo plot per_read
  role: Plots per-read modification statistics at target genomic locations using the
    step-4 per-read statistics file.
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: extract_array.py
  role: Custom script that extracts per-read signal arrays (current, dwell, stdev)
    from single fast5 files into sorted .npy arrays for python-side KS profiling against
    annotated sites.
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: Nanopolish
  role: Alternative raw-signal-to-sequence eventalign tool used for concordance checks;
    Tombo and Nanopolish KS profiles agree at r ≈ 0.75–0.79 (current) and 0.45–0.55
    (dwell time).
provenance:
  source_task_ids:
  - task_003
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:17030b17c724cde3a78c72d20835121c68068ed1fb5b95da7070a30afdf13c12
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - Nanopolish
  - extract_array.py
  - multi_to_single_fast5
  - python
  - tombo
  - tombo detect_modifications model_sample_compare
  - tombo plot per_read
  - tombo resquiggle
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 21
---

# rrna-modification-site-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Localizes RNA modification sites by comparing per-position Kolmogorov–Smirnov statistics of nanopore ionic-current and dwell-time distributions between native and IVT-control direct RNA reads, then tabulating called KS-peak positions against annotated E. coli / S. cerevisiae rRNA modification maps.

## When to use

Use when you have native (modified) and IVT (unmodified) direct RNA nanopore fast5 datasets for a transcript with known modification annotations (e.g. rRNA Nm, Ψ, base sites) and need to call/localize modification positions from raw-signal KS statistics.

## When NOT to use

- You have no IVT/unmodified control sample — the paper notes identification without an IVT comparison would require training sets of known modifications in all possible kmer sequence contexts, which do not yet exist.
- Your goal is de novo modification classification independent of prior annotation rather than localizing sites to a known rRNA modification map.
- Input is already a per-position statistics table requiring no raw-signal processing (use downstream profiling/correlation skills instead).

## Inputs

- Native (modified) direct RNA sequencing multi-fast5 files from E. coli or S. cerevisiae rRNA (e.g. from NCBI SRA)
- Matching IVT (unmodified) control direct RNA multi-fast5 files
- rRNA FASTA reference sequence for resquiggling
- Annotated rRNA modification map (positions and classes: Nm, Ψ, base modifications) for the sequenced organism
- tombo per-position KS statistics file from model_sample_compare (with --statistics-file-basename)
- Per-read statistics file generated with --per-read-statistics-basename

## Outputs

- Per-position KS statistics (current and dwell-time) across the rRNA reference from tombo model_sample_compare
- Per-read statistics file for tombo plot per_read visualization
- Per-read signal .npy arrays from extract_array.py (--current, --dwell, --stdev)
- Tabulation of called modification positions against known rRNA modification sites, with a ±2 nt correctness criterion
- Optional median KS profiles grouped by modification class (Nm, Ψ, base)

## How to apply

Convert multi-read FAST5s to per-read files with multi_to_single_fast5 (--recursive, --fast5_out, matching --flowcell and --kit), then resquiggle each single-fast5 directory with 'tombo resquiggle' against the rRNA FASTA reference using --include-event-stdev, --num-most-common-errors 5, --ignore-read-locks, and --overwrite. Run 'tombo detect_modifications model_sample_compare' with the native fast5 directory as --fast5-basedirs and the IVT directory as --control-fast5-basedirs, setting --minimum-test-reads 100 and --multiprocess-region-size 250, and re-run with --per-read-statistics-basename to obtain per-read statistics for visualization via 'tombo plot per_read'. Extract per-read signal arrays with extract_array.py (--fa FASTA, --f5 fast5 dir, --sort-reads, and --current or --dwell/--stdev) to obtain .npy arrays for downstream KS profiling. Extract significant KS-peak positions from the model_sample_compare statistics output and tabulate them against the annotated rRNA modification map; a call is considered correct if the KS peak falls within ±2 nt of a known modified site. Optionally group called positions by modification class (Nm, Ψ, base) and compute median KS profiles centered on each site, expecting a single constriction peak at the site for current and a secondary dwell-time peak displaced ~10 nt in the 3' direction for Nm/Ψ.

## Related tools

- **multi_to_single_fast5** (Converts multi-read FAST5 files to per-read single FAST5 files (with --recursive, --fast5_out, --flowcell and --kit of the sequenced run) as prerequisite for tombo processing.) — github.com/physnano/rRNA_nanoSHAPE
- **tombo resquiggle** (Aligns raw nanopore signal to the rRNA FASTA reference per read (--include-event-stdev, --num-most-common-errors 5, --ignore-read-locks, --overwrite, multiple --processes) to enable per-position signal comparison.) — github.com/physnano/rRNA_nanoSHAPE
- **tombo detect_modifications model_sample_compare** (Computes per-position KS statistics between native and IVT fast5 signal distributions (--minimum-test-reads 100, --multiprocess-region-size 250); re-run with --per-read-statistics-basename for per-read stats.) — github.com/physnano/rRNA_nanoSHAPE
- **tombo plot per_read** (Plots per-read modification statistics at target genomic locations using the step-4 per-read statistics file.) — github.com/physnano/rRNA_nanoSHAPE
- **extract_array.py** (Custom script that extracts per-read signal arrays (current, dwell, stdev) from single fast5 files into sorted .npy arrays for python-side KS profiling against annotated sites.) — github.com/physnano/rRNA_nanoSHAPE
- **Nanopolish** (Alternative raw-signal-to-sequence eventalign tool used for concordance checks; Tombo and Nanopolish KS profiles agree at r ≈ 0.75–0.79 (current) and 0.45–0.55 (dwell time).)

## Examples

```
tombo detect_modifications model_sample_compare --fast5-basedirs <native single-fast5 dir> --control-fast5-basedirs <IVT single-fast5 dir> --statistics-file-basename rRNA_stats --minimum-test-reads 100 --multiprocess-region-size 250
```

## Evaluation signals

- Called modification positions are correct if KS-test peaks fall within ±2 nt of known modified sites in the E. coli and S. cerevisiae rRNA annotation.
- Median current profile shows a single peak centered at the modification site (nanopore constriction, relative position 0).
- Median dwell-time profile for Nm and Ψ classes shows a main peak plus a secondary peak ~10 nt downstream (3' direction); base modifications lack this secondary peak.
- tombo model_sample_compare ran with --minimum-test-reads 100 and --multiprocess-region-size 250, and a per-read statistics file was produced via --per-read-statistics-basename.
- Cross-tool check: Tombo vs Nanopolish Pearson r falls in 0.75–0.79 (current) and 0.45–0.55 (dwell time); Nm median KS exceeds Ψ.

## Limitations

- Requires comparison with an IVT control: no training sets of known modifications in all possible kmer sequence contexts exist for identification without such a control.
- Signal discrimination depends on kmer sequence context and the ability to align raw current signal.
- The method requires high concentrations of target RNA and a free 3'-hydroxyl for adaptor ligation.
- Electrophilic SHAPE reagents (e.g. AcIm) covalently modify the 3'-hydroxyl, preventing ligation; chemical/phosphatase treatment is needed to restore ligatable ends.
- --minimum-test-reads 100 (typical for in vitro samples) gates per-position KS testing, so low-coverage positions are not called.

## Evidence

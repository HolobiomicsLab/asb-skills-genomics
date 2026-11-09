---
name: modification-peak-pattern-analysis-current-vs-dwell
description: Use when you have Tombo per-position KS statistics from modified-vs-IVT-control
  ONT direct RNA sequencing plus annotated modification positions, and want to test
  whether current/dwell-time shifts form class-specific, position-resolved peak patterns.
when_to_use_negative:
- You lack an IVT (unmodified) control sample — Tombo model_sample_compare requires
  a comparison control and the paper notes training sets for identification without
  an IVT control do not yet exist.
- You have too few reads per position — the workflow specifies --minimum-test-reads
  100 (typical for in vitro samples), so sparse coverage makes median KS profiles
  unreliable.
- Your target is a short-read-based or RT-based modification assay output, where modifications
  are dissociated from their native single-molecule context.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: tombo detect_modifications model_sample_compare
  role: Produces the per-position KS statistics (modified vs. IVT control) that are
    aggregated into the median current and dwell-time profiles
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Custom script extracting per-read signal arrays (--fa, --f5, --sort-reads,
    --current/--dwell/--stdev) as .npy files used for profile computation
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: python
  role: Used to process the extracted numpy arrays, group KS statistics by modification
    class, compute and plot the median profiles
- name: tombo
  role: Resquiggles single-fast5 reads to raw-signal-to-sequence alignment before
    modification detection
  repo: https://nanoporetech.github.io/tombo/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:a4edd6e89db5faccc7bd86a21f7f28e5a0532eecb45dbf3bb5587b4599f194d9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/physnano/rRNA_nanoSHAPE
  tools:
  - extract_array.py
  - python
  - tombo
  - tombo detect_modifications model_sample_compare
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# modification-peak-pattern-analysis-current-vs-dwell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes aggregate median profiles of per-position KS statistics from Tombo model_sample_compare, centered on annotated rRNA modification sites and grouped by modification class (Nm, Ψ, base), to detect class-specific signatures in nanopore current and dwell-time signals. The expected signature is a single current peak at the modification and a dual dwell-time peak with a secondary peak ~10 nt in the 3' direction for Nm and Ψ.

## When to use

Use when you have Tombo per-position KS statistics from modified-vs-IVT-control ONT direct RNA sequencing plus annotated modification positions, and want to test whether current/dwell-time shifts form class-specific, position-resolved peak patterns.

## When NOT to use

- You lack an IVT (unmodified) control sample — Tombo model_sample_compare requires a comparison control and the paper notes training sets for identification without an IVT control do not yet exist.
- You have too few reads per position — the workflow specifies --minimum-test-reads 100 (typical for in vitro samples), so sparse coverage makes median KS profiles unreliable.
- Your target is a short-read-based or RT-based modification assay output, where modifications are dissociated from their native single-molecule context.

## Inputs

- Resquiggled single-fast5 directories for modified and IVT control samples
- FASTA reference of the target rRNA sequence
- Annotated rRNA modification site list with modification class labels (Nm, Ψ, base)
- Tombo per-position KS statistics output from detect_modifications model_sample_compare
- Per-read signal .npy arrays produced by extract_array.py (--current, --dwell, or --stdev)

## Outputs

- Median KS statistic profiles for current and dwell time, centered on modified nucleotides, per modification class
- Plot of median profiles showing peak positions relative to the modification site
- Confirmation/quantification of secondary dwell-time peak displacement (~10 nt, 3' direction) for Nm and Ψ

## How to apply

Run tombo detect_modifications model_sample_compare on resquiggled single-fast5 directories (modified vs. IVT control) with --minimum-test-reads 100 (typical for in vitro samples) and --multiprocess-region-size 250 to obtain per-position KS statistics, then extract per-read signal arrays with extract_array.py using --fa, --f5, --sort-reads, and --current/--dwell/--stdev to write .npy arrays. Align the per-position KS statistics to annotated rRNA modification sites in E. coli or S. cerevisiae rRNA, group them by modification class (Nm, Ψ, base), and compute the median KS statistic profile over relative positions centered on each modified nucleotide for each signal type. The rationale is that the helicase motor produces modification-dependent signals distally, so Nm and Ψ are expected to show a constriction peak at position 0 plus a secondary dwell-time peak ~10 nt in the 3' direction, whereas base modifications lack this secondary dwell-time peak and Nm shows a larger median KS statistic than Ψ. Validate the resulting profiles against these expected shapes before interpreting any new modification site.

## Related tools

- **tombo detect_modifications model_sample_compare** (Produces the per-position KS statistics (modified vs. IVT control) that are aggregated into the median current and dwell-time profiles) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Custom script extracting per-read signal arrays (--fa, --f5, --sort-reads, --current/--dwell/--stdev) as .npy files used for profile computation) — github.com/physnano/rRNA_nanoSHAPE
- **python** (Used to process the extracted numpy arrays, group KS statistics by modification class, compute and plot the median profiles)
- **tombo** (Resquiggles single-fast5 reads to raw-signal-to-sequence alignment before modification detection) — https://nanoporetech.github.io/tombo/

## Examples

```
tombo detect_modifications model_sample_compare --fast5-basedirs <MODIFIED single-fast5 dir> --control-fast5-basedirs <CONTROL dir> --statistics-file-basename <basename> --minimum-test-reads 100 --multiprocess-region-size 250
```

## Evaluation signals

- Median current profile shows a single peak centered at relative position 0 (the nanopore constriction at the modification site).
- Median dwell-time profile for Nm and Ψ classes shows the primary constriction peak plus a secondary peak approximately 10 nt displaced in the 3' direction.
- Base modification class lacks the secondary dwell-time peak seen for Nm and Ψ.
- Nm modifications yield a larger median KS statistic than Ψ in the dwell-time profile.
- Tombo-reported modification positions agree with known rRNA sites within ±2 nt (as reported for both Tombo and Nanopolish alignment).

## Limitations

- Signal discrimination for modification detection depends on kmer sequence context and the ability to align raw current.
- No training sets of known modifications in all possible kmer sequence contexts exist, so identification requires comparison with an IVT control.
- At least --minimum-test-reads 100 reads per position are required for the sample-compare test, limiting analysis to well-covered sites.
- Secondary dwell-time peak (~10 nt 3') is observed for Nm and Ψ classes but not for base modifications, so the expected pattern is class-dependent and its absence for base modifications is itself the expected result, not a failure.

## Evidence

- [abstract] Endogenous 2'-O-methyl and base modification dwell-time signature: "We detect endogenous 2'-O-methyl and base modifications across E. coli and S. cerevisiae ribosomal RNAs as shifts in current signal and dwell times distally through interactions with the helicase"
- [discussion] Tombo/Nanopolish localization accuracy: "we performed raw-signal-to-sequence alignment with both Tombo31 and Nanopolish32 and identified rRNA modification positions to within ±2 nt of known modified sites."
- [readme] Extract arrays for per-read visualization: "Finally, the extract_array.py script is used to generate an output numpy array which can be easily processed using python for per-read modification visualization/plotting."
- [readme] Minimum test reads threshold: "--minimum-test-reads <num min test reads, typically 100 for in vitro>"

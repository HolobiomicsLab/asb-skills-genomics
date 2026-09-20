---
name: nanoshape-reactivity-profiling
description: Use when validating nanoSHAPE per-read reactivity (from current or dwell
  signal) against SHAPE-MaP on an AcIm-modified RNA such as pri-miR-17~92, or when
  determining how few reads are needed for a stable reactivity profile.
when_to_use_negative:
- The goal is de novo modification identification without an IVT/unmodified control
  — the method relies on comparison with unmodified reads
- The RNA is short or the target region is 5'-proximal — nanoSHAPE has poor resolution
  at the 5' ends of longer molecules and short-read SHAPE-MaP may suffice
- Read depth is far below ~200 reads — the subsampling analysis shows 95% of maximum
  correlation requires approximately 200 reads
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_0659
tools:
- name: tombo detect_modifications model_sample_compare
  role: Compares AcIm-modified vs control single fast5s to detect modifications and
    generate per-read statistics used for downstream single-molecule analysis
  repo: https://nanoporetech.github.io/tombo/
- name: tombo
  role: Resquiggles single-read fast5s to raw-signal-to-sequence alignment before
    model_sample_compare
  repo: https://nanoporetech.github.io/tombo/
- name: extract_array.py
  role: Custom script extracting per-read current/dwell/stdev signal arrays from single
    fast5s into a .npy array for per-position KS statistics
  repo: github.com/physnano/rRNA_nanoSHAPE
- name: multi_to_single_fast5
  role: Converts multi fast5 sequencing files to single-read fast5 files required
    by tombo
- name: python
  role: Runs extract_array.py and downstream per-read statistics / correlation analysis
  repo: github.com/physnano/rRNA_nanoSHAPE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xgen.2022.100097
    title: Direct detection of RNA modifications and structure using single-molecule
      nanopore sequencing
  source_content_hash: sha256:bdf70f4472ea7c1fea6c10f7b77fdd7a946a7706762c211a8ce4d05d7af56580
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
  - multi_to_single_fast5
  - python
  - tombo
  - tombo detect_modifications model_sample_compare
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# nanoshape-reactivity-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates per-read nanoSHAPE reactivity profiles from AcIm-modified and control direct RNA nanopore fast5s using tombo model_sample_compare plus per-read signal extraction, then correlates the KS-derived profile against an independent SHAPE-MaP reactivity profile to validate single-molecule structural probing.

## When to use

Use when validating nanoSHAPE per-read reactivity (from current or dwell signal) against SHAPE-MaP on an AcIm-modified RNA such as pri-miR-17~92, or when determining how few reads are needed for a stable reactivity profile.

## When NOT to use

- The goal is de novo modification identification without an IVT/unmodified control — the method relies on comparison with unmodified reads
- The RNA is short or the target region is 5'-proximal — nanoSHAPE has poor resolution at the 5' ends of longer molecules and short-read SHAPE-MaP may suffice
- Read depth is far below ~200 reads — the subsampling analysis shows 95% of maximum correlation requires approximately 200 reads

## Inputs

- Single-read fast5 directories for AcIm-modified and unmodified control RNA (same target, e.g. pri-miR-17~92)
- FASTA reference for the target RNA
- Per-read statistics file from tombo detect_modifications model_sample_compare
- Independent AcIm SHAPE-MaP reactivity profile for the same RNA

## Outputs

- Per-read statistics file from tombo (--per-read-statistics-basename)
- NumPy .npy array of per-read signal values (current, dwell, or stdev) from extract_array.py
- nanoSHAPE per-position reactivity profile derived from KS statistics
- Correlation (rho) of nanoSHAPE vs SHAPE-MaP per AcIm concentration and a reads-vs-correlation subsampling curve

## How to apply

Convert multi-fast5 files to single fast5s and resquiggle both the AcIm-modified and control/unmodified reads with tombo. Run 'tombo detect_modifications model_sample_compare' twice: once with --control-fast5-basedirs and --minimum-test-reads 100 (typical in vitro value) for aggregate statistics, and again with --per-read-statistics-basename to emit the per-read statistics file. Extract per-read signal arrays (current, dwell, or stdev) from single fast5s using extract_array.py with --fa, --f5, --rna-name, --sort-reads, and --current/--dwell/--stdev, producing a .npy array. Compute per-position Kolmogorov-Smirnov statistics between modified and control per-read signal distributions to obtain the nanoSHAPE reactivity profile, then Spearman-correlate it against the AcIm SHAPE-MaP reactivity profile. Subsample reads from 1 to the full count and plot correlation versus read number to find the reads needed to reach 95% of maximum correlation. Success for pri-miR-17~92 is rho = 0.51 (current) and rho = 0.28 (dwell) at 150 mM AcIm, with 95% of maximum correlation at ~200 reads.

## Related tools

- **tombo detect_modifications model_sample_compare** (Compares AcIm-modified vs control single fast5s to detect modifications and generate per-read statistics used for downstream single-molecule analysis) — https://nanoporetech.github.io/tombo/
- **tombo** (Resquiggles single-read fast5s to raw-signal-to-sequence alignment before model_sample_compare) — https://nanoporetech.github.io/tombo/
- **extract_array.py** (Custom script extracting per-read current/dwell/stdev signal arrays from single fast5s into a .npy array for per-position KS statistics) — github.com/physnano/rRNA_nanoSHAPE
- **multi_to_single_fast5** (Converts multi fast5 sequencing files to single-read fast5 files required by tombo)
- **python** (Runs extract_array.py and downstream per-read statistics / correlation analysis) — github.com/physnano/rRNA_nanoSHAPE

## Examples

```
tombo detect_modifications model_sample_compare --fast5-basedirs <modified_f5> --control-fast5-basedirs <control_f5> --per-read-statistics-basename <out> --minimum-test-reads 100 --multiprocess-region-size 250
```

## Evaluation signals

- Spearman rho between nanoSHAPE current-derived KS profile and AcIm SHAPE-MaP profile reaches 0.51 at 150 mM AcIm for pri-miR-17~92
- Dwell-time-derived profile reaches rho = 0.28 at 150 mM AcIm (current outperforms dwell)
- Subsampling curve reaches 95% of maximum correlation at approximately 200 reads
- model_sample_compare runs with --minimum-test-reads set to the typical in vitro value of 100
- extract_array.py emits a .npy array with --sort-reads and a single signal channel (--current, --dwell, or --stdev) specified

## Limitations

- High rates of AcIm modification lower the number of full-length reads, overall yield, and alignment rates
- nanoSHAPE has poor resolution of reactivity profiles at the 5' ends of longer RNA molecules
- Electrophilic SHAPE reagents including AcIm covalently modify the 3'-hydroxyl, preventing ligation, and the method requires high concentrations of target RNA and a free 3'-hydroxyl
- Signal discrimination remains an outstanding challenge, being a function of kmer sequence context and raw-current alignment quality

## Evidence

- [methods] tombo detect_modifications model_sample_compare 
		--fast5-basedirs <path to single fast5 files MODIFIED> 
		--control-fast5-basedirs: "tombo detect_modifications model_sample_compare 
		--fast5-basedirs <path to single fast5 files MODIFIED> 
		--control-fast5-basedirs <path to single fast5 files CONTROL/UNMODIFIED>"
- [methods] python extract_array.py 
		--fa <FASTA reference file> 
		--f5 <path to single fast5 files to process, either MODIFIED or UNMODIFIED>: "python extract_array.py 
		--fa <FASTA reference file> 
		--f5 <path to single fast5 files to process, either MODIFIED or UNMODIFIED>"

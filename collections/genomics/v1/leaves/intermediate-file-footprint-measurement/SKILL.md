---
name: intermediate-file-footprint-measurement
description: Use when reproducing or extending published claims about intermediate
  file size or parse-time performance of a read-depth CNV caller, e.g. verifying CNVpytor's
  2-20x parsing speedup and 20-50x smaller intermediate files versus CNVnator.
when_to_use_negative:
- The goal is accuracy of CNV calls rather than I/O performance — call-level agreement
  (>99% overlap between CNVnator and CNVpytor calls) is a separate evaluation and
  does not follow from footprint benchmarks.
- The comparison target is not a CNVnator-derived intermediate format, so the reported
  20-50x size and 2-20x speed ratios are not applicable baselines.
- Only final results files (calls, plots, VCF/TSV exports) matter for the analysis
  and intermediate artifacts are immediately discarded.
edam_operation: http://edamontology.org/operation_3211
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Read-depth parser under benchmark; produces compressed HDF5 .pytor intermediate
    files from BAM using a pysam-based parallelized engine
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Baseline read-depth parser whose wall-time and intermediate file size are
    recorded for the ratio comparison
- name: Python
  role: Scripting environment used to time parsing runs, compute per-sample ratios,
    and assemble the metrics table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:4c3df7e3e5ef4cadffd4d444a72607cdc444bba5f22cd86aa7ad4fd9afa5c05d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3211
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# intermediate-file-footprint-measurement

## Summary

Benchmarks the disk footprint and parse speed of intermediate files produced by CNV callers, comparing CNVpytor's compressed HDF5 (.pytor) files against CNVnator's intermediate output on the same BAM inputs. Motivated by the reported 20-50x smaller .pytor files and 2-20x faster BAM parsing of CNVpytor.

## When to use

Use when reproducing or extending published claims about intermediate file size or parse-time performance of a read-depth CNV caller, e.g. verifying CNVpytor's 2-20x parsing speedup and 20-50x smaller intermediate files versus CNVnator.

## When NOT to use

- The goal is accuracy of CNV calls rather than I/O performance — call-level agreement (>99% overlap between CNVnator and CNVpytor calls) is a separate evaluation and does not follow from footprint benchmarks.
- The comparison target is not a CNVnator-derived intermediate format, so the reported 20-50x size and 2-20x speed ratios are not applicable baselines.
- Only final results files (calls, plots, VCF/TSV exports) matter for the analysis and intermediate artifacts are immediately discarded.

## Inputs

- Whole-genome alignment BAM files (e.g. CEPH/Platinum Genomes trio samples)
- Installed CNVpytor (with pysam, h5py dependencies) and CNVnator

## Outputs

- Per-sample .pytor intermediate files (HDF5 with compressed 100-bp RD bins)
- CNVnator intermediate output files
- Metrics table (e.g. parsing_benchmark.tsv) with per-sample parse wall-times, intermediate file sizes, and speedup/size ratios

## How to apply

Obtain the same alignment inputs for all tools under comparison, e.g. the CEPH (Platinum Genomes) trio example BAM files used in the CNVpytor paper. For each sample, parse the BAM with CNVpytor via 'cnvpytor -root file.pytor -rd file.bam', recording parse wall-time (CNVpytor uses a pysam-based, parallelized read-depth engine) and the size of the resulting .pytor file, an HDF5 container that stores RD signal compressed in 100-bp bins. Parse the identical BAM with CNVnator's read-depth parser and record its wall-time and intermediate file size. Compute per-sample speedup ratios (CNVnator wall-time / CNVpytor wall-time) and intermediate-size ratios (CNVnator file size / CNVpytor .pytor size), then assemble a metrics table such as parsing_benchmark.tsv. Accept the benchmark when the speedup falls within the paper-reported 2-20x range and the size ratio falls within the reported 20-50x range.

## Related tools

- **CNVpytor** (Read-depth parser under benchmark; produces compressed HDF5 .pytor intermediate files from BAM using a pysam-based parallelized engine) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Baseline read-depth parser whose wall-time and intermediate file size are recorded for the ratio comparison)
- **Python** (Scripting environment used to time parsing runs, compute per-sample ratios, and assemble the metrics table)

## Examples

```
cnvpytor -root sample.pytor -rd sample.bam && du -h sample.pytor
```

## Evaluation signals

- Computed CNVpytor-over-CNVnator parse speedup falls within the paper-reported 2-20x range for BAM parsing
- Computed intermediate-file size ratio (CNVnator / CNVpytor) falls within the reported 20-50x range
- Each CNVpytor run yields a valid .pytor HDF5 file whose RD signal is stored as 100-bp compressed bins
- Metrics table (parsing_benchmark.tsv) contains wall-time, file size, and both ratios for all three trio samples
- Both tools parsed identical BAM inputs so ratios are apples-to-apples

## Limitations

- The 2-20x speedup and 20-50x size ranges are tool- and dataset-specific (CNVpytor vs CNVnator on example WGS BAMs); they may not transfer to other callers, genomes, or read types.
- Version drift affects the size comparison: CNVpytor v1.3.1 further reduced .pytor size by compressing the BAF likelihood matrix and offering the -nolh option (files under 50 MB), changing the footprint relative to the original benchmark.
- Wall-time measurements are hardware- and parallelization-dependent since CNVpytor's read-depth engine is pysam-based and parallelized.

## Evidence

- [abstract] Parsing speed and intermediate file size claim: "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [abstract] Intermediate file size claim: "and has (20–50 times) smaller intermediate files"
- [discussion] Compression via 100-bp bins in .pytor files: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [methods] Canonical CNVpytor parse invocation: "### Call CNVs using read depth:
> cnvpytor -root file.pytor -rd file.bam"
- [readme] README: further size reductions in v1.3.1: "Reduced Pytor file size by compressing the BAF likelihood matrix"

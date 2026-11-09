---
name: hdf5-storage-format-handling
description: Use when a CNV analysis pipeline needs a compact, compressed intermediate
  store for per-sample read-depth bins and SNP BAF likelihoods, or when benchmarking
  parse wall-time and file footprint against CNVnator.
when_to_use_negative:
- The pipeline already relies on CNVnator ROOT files and requires bit-identical legacy
  output rather than a compressed HDF5 store
- You need to share intermediate files with tools that cannot read HDF5 and have no
  CNVpytor import path
- Downstream analysis needs raw per-base read counts rather than 100-bp binned, compressed
  RD signal
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Writes and reads the HDF5 .pytor container; its pysam-based parallelized read-depth
    engine parses BAMs 2-20x faster than CNVnator while compressing 100-bp RD bins
    into the .pytor file
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Baseline predecessor whose parse wall-time and intermediate file size are
    compared against CNVpytor's .pytor footprint
- name: Python
  role: Implementation language of CNVpytor; h5py dependency provides the HDF5 backend
    for .pytor files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:bad75428891123f6ddbe6f464484b1e5b5109aabc78a928d46aeb740b93c0de1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# hdf5-storage-format-handling

## Summary

Use CNVpytor's HDF5-backed .pytor file format to store compressed read-depth signal (100-bp bins) and B-allele frequency likelihood data, yielding intermediate files 20-50 times smaller than CNVnator's while supporting parallelized parsing.

## When to use

Use when a CNV analysis pipeline needs a compact, compressed intermediate store for per-sample read-depth bins and SNP BAF likelihoods, or when benchmarking parse wall-time and file footprint against CNVnator.

## When NOT to use

- The pipeline already relies on CNVnator ROOT files and requires bit-identical legacy output rather than a compressed HDF5 store
- You need to share intermediate files with tools that cannot read HDF5 and have no CNVpytor import path
- Downstream analysis needs raw per-base read counts rather than 100-bp binned, compressed RD signal

## Inputs

- Whole-genome alignment file (BAM) per sample
- Optional VCF of SNPs and single-letter indels for BAF data

## Outputs

- Per-sample .pytor file (HDF5 format with compressed 100-bp RD bins)
- Benchmark metrics table (e.g. parsing_benchmark.tsv) with parse wall-times, file sizes, and speedup/size ratios

## How to apply

Create a .pytor container per sample with 'cnvpytor -root file.pytor -rd file.bam'; CNVpytor writes the RD signal into an HDF5 file using compressed 100-bp bins, which is the mechanism behind the reported 20-50x smaller intermediate files versus CNVnator's ROOT output. Downstream steps (-his, -partition, -call, -snp, -baf) read and append results to the same .pytor file, so it acts as the single per-sample data store. In v1.3.1+ the BAF likelihood matrix is itself compressed, and the '-nolh' option skips storing it entirely, shrinking the .pytor file to under 50 MB with likelihoods recomputed at the -call step. When benchmarking, time the -rd parse step per sample (e.g. on the CEPH Platinum Genomes trio BAMs), record the .pytor file size, and compare against CNVnator's parse time and intermediate file size.

## Related tools

- **CNVpytor** (Writes and reads the HDF5 .pytor container; its pysam-based parallelized read-depth engine parses BAMs 2-20x faster than CNVnator while compressing 100-bp RD bins into the .pytor file) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Baseline predecessor whose parse wall-time and intermediate file size are compared against CNVpytor's .pytor footprint)
- **Python** (Implementation language of CNVpytor; h5py dependency provides the HDF5 backend for .pytor files)

## Examples

```
cnvpytor -root file.pytor -rd file.bam
```

## Evaluation signals

- The computed CNVpytor-over-CNVnator parse speedup falls within the paper-reported 2-20x range for BAM parsing
- The CNVnator-over-CNVpytor intermediate-size ratio falls within the reported 20-50x range
- Each sample yields a single .pytor file that subsequent -his/-partition/-call/-baf commands can reopen without errors
- With -nolh (v1.3.1+), the final .pytor file is less than 50 MB and -call still completes by computing likelihood at runtime
- RD signal in the .pytor file is stored as 100-bp bins, and >99% of calls overlap CNVnator's calls when the same BAMs are analyzed

## Limitations

- Somatic CNA caller is only a prototype and is functional but not fully mature
- Some compression choices (e.g. -nolh) trade file size for runtime, since likelihood must be recalculated during the -call step
- Missing signals noted in the article index: no changelog, no DOI/Zenodo record for archived code/data version

## Evidence

- [abstract] Speedup and compression claims: "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [abstract] Smaller intermediate files: "and has (20–50 times) smaller intermediate files"
- [discussion] HDF5 100-bp bin compression: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [methods] Parse command: "### Call CNVs using read depth:
> cnvpytor -root file.pytor -rd file.bam"
- [readme] v1.3.1 likelihood compression and -nolh: "Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB"

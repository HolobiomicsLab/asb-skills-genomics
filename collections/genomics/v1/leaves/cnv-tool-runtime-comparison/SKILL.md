---
name: cnv-tool-runtime-comparison
description: Use when benchmarking alignment-file parsing performance of read-depth
  CNV callers, or when quantifying runtime and intermediate-file footprint differences
  between CNVpytor and CNVnator on the same BAM inputs.
when_to_use_negative:
- The goal is comparing CNV call concordance rather than parsing performance — for
  that, use the finding that called regions by CNVnator and CNVpytor overlap by >99%.
- Input is not a whole-genome alignment (BAM) file, e.g. already-parsed .pytor files
  or VCF-only variant data, since the benchmark targets the alignment parsing stage.
- Allele-imbalance/BAF functionality (e.g. -snp, -baf, likelihood matrices) is the
  analysis focus — this skill benchmarks only the -rd parsing step.
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: Subject tool; parses BAM alignment files into compressed .pytor HDF5 intermediates
    via a pysam-based parallelized read-depth engine
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Baseline tool whose read-depth parser wall-time and intermediate file size
    are measured for ratio computation
- name: Python
  role: Implementation language of CNVpytor and used to compute per-sample speedup
    and size ratios and assemble parsing_benchmark.tsv
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:c182e89ad991ed24195b11a48f4c80a7fae1dad729c05c3c6ad25bff7b318acc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3197
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cnv-tool-runtime-comparison

## Summary

Benchmark protocol for comparing read-depth CNV callers at the alignment-file parsing stage, specifically CNVpytor versus CNVnator, measuring wall-time speedup (reported 2-20x) and intermediate file size reduction (reported 20-50x via HDF5 .pytor files with compressed 100-bp RD bins).

## When to use

Use when benchmarking alignment-file parsing performance of read-depth CNV callers, or when quantifying runtime and intermediate-file footprint differences between CNVpytor and CNVnator on the same BAM inputs.

## When NOT to use

- The goal is comparing CNV call concordance rather than parsing performance — for that, use the finding that called regions by CNVnator and CNVpytor overlap by >99%.
- Input is not a whole-genome alignment (BAM) file, e.g. already-parsed .pytor files or VCF-only variant data, since the benchmark targets the alignment parsing stage.
- Allele-imbalance/BAF functionality (e.g. -snp, -baf, likelihood matrices) is the analysis focus — this skill benchmarks only the -rd parsing step.

## Inputs

- CEPH (Platinum Genomes) trio whole-genome alignment BAM files (three samples, genomics-public-data/platinum-genomes)
- Installed CNVpytor (pip install cnvpytor; cnvpytor -download)
- Installed CNVnator read-depth parser

## Outputs

- Per-sample parsing_benchmark.tsv with wall-times, intermediate file sizes, and speedup/size ratios
- Per-sample .pytor HDF5 intermediate files with compressed 100-bp RD bins

## How to apply

Obtain the CEPH (Platinum Genomes) trio example BAM files from the genomics-public-data/platinum-genomes bucket. For each sample, parse the BAM with CNVpytor using `cnvpytor -root file.pytor -rd file.bam`, which relies on its pysam-based parallelized read-depth engine, and record the wall-time. Parse the identical BAM with CNVnator's read-depth parser and record its wall-time plus the size of its intermediate output. Record the size of each CNVpytor .pytor file (HDF5 format storing RD signal with 100-bp bins and additional compression). Compute the per-sample CNVpytor-over-CNVnator speedup ratio and the CNVnator-over-CNvpytor intermediate-size ratio, and assemble a per-sample metrics table (parsing_benchmark.tsv).

## Related tools

- **CNVpytor** (Subject tool; parses BAM alignment files into compressed .pytor HDF5 intermediates via a pysam-based parallelized read-depth engine) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Baseline tool whose read-depth parser wall-time and intermediate file size are measured for ratio computation)
- **Python** (Implementation language of CNVpytor and used to compute per-sample speedup and size ratios and assemble parsing_benchmark.tsv)

## Examples

```
cnvpytor -root file.pytor -rd file.bam
```

## Evaluation signals

- Computed CNVpytor-over-CNVnator parsing speedup falls within the paper-reported 2-20x range for BAM parsing.
- Computed intermediate-file size ratio falls within the reported 20-50x range.
- Each of the three CEPH trio samples yields a .pytor file that opens with `cnvpytor -root file.pytor -view` without errors.
- parsing_benchmark.tsv contains wall-time, intermediate size, and both ratios for all three samples.
- CNVpytor .pytor files store RD signal at 100-bp bin resolution in HDF5 format (verifiable with h5py).

## Limitations

- Reported speedup (2-20x) and size reduction (20-50x) ranges are hardware- and input-dependent; benchmark on the same machine and identical BAMs for a fair comparison.
- The paper's benchmark covers only the alignment parsing stage, not the full CNV calling pipeline (-his, -partition, -call).
- CNVpytor and CNVnator implementations differ slightly; called regions overlap by >99% but are not identical, so the benchmark does not imply equivalence of downstream call sets.

## Evidence

- [abstract] Parsing speedup: "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [abstract] Intermediate file size: "and has (20–50 times) smaller intermediate files"
- [discussion] HDF5 storage with 100-bp bins: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [methods] Parsing command: "### Call CNVs using read depth:
> cnvpytor -root file.pytor -rd file.bam"
- [results] Example dataset: "Example dataset: https://console.cloud.google.com/storage/browser/genomics-public-data/platinum-genomes"
- [discussion] Call concordance: "In spite of slightly different implementation, called regions by CNVnator and CNVpytor overlap by >99%."

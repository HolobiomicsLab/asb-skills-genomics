---
name: read-depth-benchmarking
description: Use when you need to empirically validate performance claims for read-depth
  CNV callers by measuring parse wall-time and intermediate file sizes per sample
  on the same BAM inputs.
when_to_use_negative:
- The goal is CNV calling accuracy or concordance of called regions rather than parse-time
  and file-size benchmarking (use CNVpytor's -call workflow instead; note CNVnator
  and CNVpytor calls overlap by >99%)
- The inputs are not whole-genome alignment (BAM) files, e.g. only VCF SNP data is
  available — the speedup claim applies specifically to parsing alignment files
- You need the somatic CNA caller for cancer/mosaic CNAs, which exists only as a prototype
  and is not what this benchmark exercises
edam_operation: http://edamontology.org/operation_3197
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: 'Tested tool: parses BAM read depth with a pysam-based parallelized engine
    and writes compressed HDF5 .pytor intermediates'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: 'Baseline tool: its read-depth parser and ROOT intermediate files provide
    the wall-time and file-size reference against which CNVpytor speedup is measured'
- name: Python
  role: Used to compute per-sample speedup and size ratios and assemble the parsing_benchmark.tsv
    metrics table
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:41ce007de9e4ea464d4afd6e546802c50f33623f38702f05b56cc155e2c12981
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
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# read-depth-benchmarking

## Summary

Benchmarks CNVpytor against its predecessor CNVnator on read-depth parsing of whole-genome alignment files, quantifying the reported 2–20x parse speedup and 20–50x smaller intermediate (.pytor vs ROOT) file footprint on the CEPH trio example dataset.

## When to use

Use when you need to empirically validate performance claims for read-depth CNV callers by measuring parse wall-time and intermediate file sizes per sample on the same BAM inputs.

## When NOT to use

- The goal is CNV calling accuracy or concordance of called regions rather than parse-time and file-size benchmarking (use CNVpytor's -call workflow instead; note CNVnator and CNVpytor calls overlap by >99%)
- The inputs are not whole-genome alignment (BAM) files, e.g. only VCF SNP data is available — the speedup claim applies specifically to parsing alignment files
- You need the somatic CNA caller for cancer/mosaic CNAs, which exists only as a prototype and is not what this benchmark exercises

## Inputs

- CEPH (Platinum Genomes) trio example BAM alignment files for three samples (genomics-public-data/platinum-genomes)
- Installed CNVpytor (Python, pysam, h5py dependencies) and CNVnator binaries
- Reference genome context (GRCh37/hg19 or GRCh38) for CNVpytor resource data

## Outputs

- Per-sample .pytor (HDF5) intermediate files from CNVpytor parsing
- CNVnator intermediate (ROOT) files from its read-depth parser
- parsing_benchmark.tsv metrics table listing per-sample parse wall-times, intermediate file sizes, speedup ratios, and size ratios

## How to apply

Obtain the CEPH (Platinum Genomes) trio BAM files from the genomics-public-data/platinum-genomes bucket and parse each sample twice: once with CNVpytor using 'cnvpytor -root file.pytor -rd file.bam', which relies on its pysam-based parallelized read-depth engine and stores RD signal as compressed 100-bp bins in HDF5 (.pytor) format, and once with CNVnator's read-depth parser. Record parse wall-time for both tools and the on-disk size of both intermediate outputs. Compute the per-sample CNVpytor-over-CNVnator speedup ratio and the inverse intermediate-size ratio, then assemble a per-sample metrics table (parsing_benchmark.tsv). The benchmark is accepted only if the speedup falls within the paper-reported 2–20x range for BAM parsing and the size ratio falls within the reported 20–50x range; this works because CNVpytor's HDF5 storage with 100-bp bin compression is intrinsically more compact than CNVnator's ROOT files.

## Related tools

- **CNVpytor** (Tested tool: parses BAM read depth with a pysam-based parallelized engine and writes compressed HDF5 .pytor intermediates) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Baseline tool: its read-depth parser and ROOT intermediate files provide the wall-time and file-size reference against which CNVpytor speedup is measured)
- **Python** (Used to compute per-sample speedup and size ratios and assemble the parsing_benchmark.tsv metrics table)

## Examples

```
time cnvpytor -root sample.pytor -rd sample.bam
```

## Evaluation signals

- Computed CNVpytor-over-CNVnator parse speedup falls within the paper-reported 2–20x range for BAM parsing
- Computed intermediate file size ratio falls within the paper-reported 20–50x range
- Each trio sample yields a valid .pytor file readable by subsequent CNVpytor steps (-his, -partition, -call)
- parsing_benchmark.tsv contains wall-time, intermediate size, and both ratios for all three CEPH trio samples
- Both tools are run on identical BAM inputs so the per-sample ratios are directly comparable

## Limitations

- Reported speedup (2–20x) and size reduction (20–50x) are ranges whose realized values depend on the alignment file and hardware; the validation gate is range-based, not point-based
- Speed and file-size gains do not imply accuracy differences: called regions by CNVnator and CNVpytor overlap by >99%
- CNVpytor versions change file-size behavior (e.g. v1.3.1 compresses the BAF likelihood matrix and offers -nolh), so size ratios may shift across versions

## Evidence

- [abstract] 2-20x parse speedup claim: "CNVpytor is significantly faster than CNVnator—particularly for parsing alignment files (2–20 times faster)"
- [abstract] 20-50x smaller intermediates claim: "and has (20–50 times) smaller intermediate files"
- [discussion] pysam-based RD storage: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [methods] CNVpytor parse command: "> cnvpytor -root file.pytor -rd file.bam"
- [results] CEPH trio example dataset source: "Example dataset: https://console.cloud.google.com/storage/browser/genomics-public-data/platinum-genomes"

---
name: gc-content-resource-mapping
description: Use when correcting GC-correlated technical bias in binned read depth
  signal during CNVpytor RD analysis and a reference FASTA is unavailable but the
  genome is GRCh37 or GRCh38.
when_to_use_negative:
- The sample is aligned to a reference genome other than GRCh37 (hg19) or GRCh38 —
  no pre-calculated GC/AT resource exists in the package for other builds
- The goal is read-depth binning or CNV calling itself rather than GC-bias correction
  of already-binned RD signal (use the -rd/-his/-partition/-call workflow instead)
- The task requires GC content for a custom bin size derived from the reference sequence,
  which the shipped 100-bp resource does not directly provide
edam_operation: http://edamontology.org/operation_3216
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: Provides the rd_use_gc_corr viewer parameter (defaults to True) that models
    and corrects GC-correlated RD bias, and ships the pre-calculated GC/AT resource
    data
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: Python library used to read the binned RD signal from the .pytor HDF5 file
    for joining with GC content
  repo: https://github.com/abyzovlab/CNVpytor
- name: matplotlib
  role: Plots the fitted GC-bias curve used for RD normalization
- name: numpy
  role: Numerical modeling and normalization of RD by the fitted GC-bias curve
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:20129b7fc1e23dfc10bdbf72ac4ce801bded4d93a39554c3dbd74ebf8638f0e8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  - h5py
  - matplotlib
  - numpy
  edam_operation: http://edamontology.org/operation_3216
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# gc-content-resource-mapping

## Summary

Maps read-depth bins to per-bin GC/AT content using the pre-calculated GC content resource shipped with the CNVpytor package for GRCh37 (hg19) and GRCh38, enabling GC-correlated RD bias correction without accessing a reference FASTA.

## When to use

Use when correcting GC-correlated technical bias in binned read depth signal during CNVpytor RD analysis and a reference FASTA is unavailable but the genome is GRCh37 or GRCh38.

## When NOT to use

- The sample is aligned to a reference genome other than GRCh37 (hg19) or GRCh38 — no pre-calculated GC/AT resource exists in the package for other builds
- The goal is read-depth binning or CNV calling itself rather than GC-bias correction of already-binned RD signal (use the -rd/-his/-partition/-call workflow instead)
- The task requires GC content for a custom bin size derived from the reference sequence, which the shipped 100-bp resource does not directly provide

## Inputs

- CNVpytor .pytor file (HDF5 format) containing binned read depth signal for a chosen bin size
- GC/AT content resource shipped with the CNVpytor package (per 100-bp bin GC and AT content for GRCh37 hg19 and GRCh38)

## Outputs

- GC-corrected per-bin read depth signal (e.g. gc_corrected_rd.tsv, one row per genomic bin)
- Fitted GC-bias curve plot (e.g. gc_bias_curve.png) showing the RD-vs-GC relationship used for normalization

## How to apply

Load the binned read depth signal from the .pytor HDF5 file using CNVpytor or h5py for the selected bin size. Instead of computing GC content from a reference FASTA, join the RD bins with the resource data shipped inside the CNVpytor package, which tabulates GC and AT content for each 100-bp bin for GRCh37 (hg19) and GRCh38. Model RD as a function of GC content to estimate the GC-correlated bias (equivalent to the rd_use_gc_corr viewer parameter, which defaults to True), then normalize each bin's RD by the fitted GC-bias curve. Rationale: shipping the GC/AT resource avoids a per-run reference scan and guarantees consistent GC values across analyses on these two reference builds.

## Related tools

- **CNVpytor** (Provides the rd_use_gc_corr viewer parameter (defaults to True) that models and corrects GC-correlated RD bias, and ships the pre-calculated GC/AT resource data) — https://github.com/abyzovlab/CNVpytor
- **h5py** (Python library used to read the binned RD signal from the .pytor HDF5 file for joining with GC content) — https://github.com/abyzovlab/CNVpytor
- **matplotlib** (Plots the fitted GC-bias curve used for RD normalization)
- **numpy** (Numerical modeling and normalization of RD by the fitted GC-bias curve)

## Examples

```
import h5py; rd = h5py.File('file.pytor','r')['RD/10000/bins'][:]  # join bins with CNVpytor shipped GC/AT resource, fit RD~GC, normalize
```

## Evaluation signals

- gc_corrected_rd.tsv exists and contains exactly one row per genomic bin from the .pytor RD signal
- The fitted curve plot shows the expected RD-vs-GC relationship and matches rd_use_gc_corr=True behavior in CNVpytor
- GC/AT values joined to bins come from the shipped per-100-bp resource for GRCh37 (hg19) or GRCh38, not from a separately computed reference scan
- Corrected RD values are normalized by the fitted GC-bias curve (mean-corrected RD near the unbiased level in GC regions with adequate coverage)

## Limitations

- Pre-calculated GC/AT content is only provided for GRCh37 (hg19) and GRCh38 at 100-bp resolution; other assemblies or custom bin sizes require computing GC from a reference
- The correction removes only GC-correlated bias; other coverage artifacts (e.g. mappability) need separate handling such as the 1000 Genomes Project P-mask via rd_use_mask

## Evidence

- [discussion] GC correction using resource data instead of reference FASTA: "information about GC and AT content for each 100-bp bin is provided as resource data within the CNVpytor package"
- [discussion] GC resource coverage of reference builds: "For the 2 versions GRCh37 (hg19) and GRCh38 of the human reference genome, information about GC and AT content for each 100-bp bin is provided"
- [readme] RD parsing from pytor files: "CNVpytor is a Python package and command line tool for CNV/CNA analysis from depth-of-coverage by mapped reads"
- [discussion] HDF5 storage of RD signal: "Additional compression is obtained by storing RD signal using 100-bp bins."

---
name: read-depth-gc-bias-correction
description: Use when correcting binned read depth signal for GC-content-dependent
  bias during CNV/CNA discovery with CNVpytor, especially on GRCh37/GRCh38 where GC
  content per 100-bp bin is pre-calculated as packaged resource data.
when_to_use_negative:
- Working with a non-human reference genome for which CNVpytor does not ship a pre-calculated
  GC/AT per-100-bp-bin resource — the packaged-resource approach described here is
  specific to GRCh37 (hg19) and GRCh38
- The goal is detecting CNVs from allele imbalance/B-allele frequency rather than
  read depth — this skill only corrects the RD signal, not SNP/BAF likelihood data
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Main tool whose RD workflow includes GC correction; ships the GRCh37/GRCh38
    GC and AT content resource data used per 100-bp bin
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor method whose RD pipeline (binning, mean-shift, partitioning, calling)
    defines the RD signal being GC-corrected
- name: Python
  role: Language used to implement CNVpytor and any scripted RD loading, GC-bias curve
    fitting, and normalization
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: Python library for reading the binned RD signal stored in the HDF5 .pytor
    file
- name: matplotlib
  role: Plotting the fitted RD-vs-GC bias curve and corrected signal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:874a8a1a9721a95363b66a258f5c8e45e7cc1a47b9d7ec779ca582bcaa4f4da6
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# read-depth-gc-bias-correction

## Summary

Removes GC-content-correlated technical bias from binned read depth (RD) signal in CNVpytor's RD analysis workflow, using the per-bin GC/AT content resource shipped with the CNVpytor package for GRCh37 (hg19) and GRCh38 instead of recomputing GC from a reference FASTA.

## When to use

Use when correcting binned read depth signal for GC-content-dependent bias during CNV/CNA discovery with CNVpytor, especially on GRCh37/GRCh38 where GC content per 100-bp bin is pre-calculated as packaged resource data.

## When NOT to use

- Working with a non-human reference genome for which CNVpytor does not ship a pre-calculated GC/AT per-100-bp-bin resource — the packaged-resource approach described here is specific to GRCh37 (hg19) and GRCh38
- The goal is detecting CNVs from allele imbalance/B-allele frequency rather than read depth — this skill only corrects the RD signal, not SNP/BAF likelihood data

## Inputs

- file.pytor (HDF5) containing binned read depth signal for the selected bin size
- GC/AT content resource data bundled with the CNVpytor package (per 100-bp bin, GRCh37/hg19 or GRCh38)

## Outputs

- GC-corrected per-bin read depth table (one row per genomic bin, e.g. gc_corrected_rd.tsv)
- Fitted GC-bias curve plot (e.g. gc_bias_curve.png) showing the RD-vs-GC relationship used for normalization

## How to apply

Load the binned RD signal from the HDF5-formatted .pytor file for the selected bin size using CNVpytor/h5py, then join each RD bin with the GC/AT content resource tabulated for each 100-bp bin of GRCh37 (hg19) or GRCh38 that ships inside the CNVpytor package. Model RD as a function of GC content to estimate the GC-correlated bias (equivalent to the viewer parameter rd_use_gc_corr, which defaults to True), then normalize each bin's RD by the fitted GC-bias curve. Rationale: CNVpytor deliberately stores GC and AT content as resource data per 100-bp bin for the two human reference builds so the correction step does not need to recompute GC from a FASTA. Validate by confirming one corrected value per genomic bin and that the fitted RD-vs-GC curve visually matches the normalization applied to the signal.

## Related tools

- **CNVpytor** (Main tool whose RD workflow includes GC correction; ships the GRCh37/GRCh38 GC and AT content resource data used per 100-bp bin) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor method whose RD pipeline (binning, mean-shift, partitioning, calling) defines the RD signal being GC-corrected)
- **Python** (Language used to implement CNVpytor and any scripted RD loading, GC-bias curve fitting, and normalization) — https://github.com/abyzovlab/CNVpytor
- **h5py** (Python library for reading the binned RD signal stored in the HDF5 .pytor file)
- **matplotlib** (Plotting the fitted RD-vs-GC bias curve and corrected signal)

## Examples

```
cnvpytor -root file.pytor -rd file.bam
```

## Evaluation signals

- gc_corrected_rd.tsv exists and contains exactly one row per genomic bin at the chosen bin size
- The fitted GC-bias curve plot shows the RD-vs-GC relationship used for normalization, matching rd_use_gc_corr behavior (default True in the viewer)
- Corrected RD bins align with the GC/AT resource binning: one GC/AT value per 100-bp bin of GRCh37/GRCh38 joined to each RD bin
- Corrected RD distribution is flatter across GC content strata than the raw RD distribution (residual RD-GC correlation near zero)
- .pytor file integrity verified via h5py (RD signal stored in 100-bp bins per CNVpytor's compression scheme)

## Limitations

- Pre-calculated GC and AT content is provided only for GRCh37 (hg19) and GRCh38; other assemblies require an alternative GC source
- GC correction addresses only GC-correlated technical bias in RD signal; other biases (e.g. mappability) require different steps such as the -rd_use_mask 1000 Genomes P-mask option
- RD binning, mean-shift, partitioning, and calling details are inherited from the CNVnator article rather than re-derived in CNVpytor

## Evidence

- [discussion] GC correction using resource data instead of reference FASTA: "information about GC and AT content for each 100-bp bin is provided as resource data within the CNVpytor package"
- [discussion] GC/AT resource coverage of both human reference builds: "For the 2 versions GRCh37 (hg19) and GRCh38 of the human reference genome, information about GC and AT content for each 100-bp bin is provided"
- [discussion] RD signal is stored in 100-bp bins in the .pytor HDF5 file: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [readme] CNVpytor command line import of read depth from BAM: "cnvpytor -root file.pytor -rd file.bam"

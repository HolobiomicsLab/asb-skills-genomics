---
name: binned-genomic-signal-normalization
description: Use when binned read-depth signal shows coverage bias correlated with
  GC content and per-bin RD must be GC-normalized for CNVpytor RD analysis on GRCh37/GRCh38.
when_to_use_negative:
- Reference assembly is not GRCh37 (hg19) or GRCh38 — the pre-calculated GC/AT resource
  is provided only for these two human reference versions.
- The analysis target is SNP/BAF evidence rather than read depth — use the VCF import
  and -baf likelihood pipeline instead of GC-correcting RD bins.
- The dominant bias is mappability-related rather than GC-correlated — filter RD with
  the 1000 Genomes Project strict P-mask (rd_use_mask) instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: computes the binned RD signal, ships the per-100-bp GC/AT resource for GRCh37/GRCh38,
    and applies GC correction during RD analysis (rd_use_gc_corr defaults to True)
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: reads the RD bins from the HDF5-format .pytor file so the GC-bias fit can
    be reconstructed in Python
- name: numpy
  role: fits the RD-vs-GC bias model and normalizes per-bin RD by the fitted curve
- name: matplotlib
  role: plots the fitted GC-bias curve used for normalization and saves it as gc_bias_curve.png
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:291f63b3917b58f61316d528da2c2d2051bda19318b3cca87b04e2e06ef4f0f9
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# binned-genomic-signal-normalization

## Summary

Reconstructs CNVpytor's removal of GC-correlated technical bias from binned read depth (RD) signal: RD bins from the HDF5-format .pytor file are joined with the package's pre-calculated per-100-bp GC/AT content resource for GRCh37/GRCh38, an RD-vs-GC bias curve is fitted, and each bin is normalized by the fitted curve.

## When to use

Use when binned read-depth signal shows coverage bias correlated with GC content and per-bin RD must be GC-normalized for CNVpytor RD analysis on GRCh37/GRCh38.

## When NOT to use

- Reference assembly is not GRCh37 (hg19) or GRCh38 — the pre-calculated GC/AT resource is provided only for these two human reference versions.
- The analysis target is SNP/BAF evidence rather than read depth — use the VCF import and -baf likelihood pipeline instead of GC-correcting RD bins.
- The dominant bias is mappability-related rather than GC-correlated — filter RD with the 1000 Genomes Project strict P-mask (rd_use_mask) instead.

## Inputs

- Binned read depth signal stored in a CNVpytor .pytor file (HDF5 format) for a chosen bin size, produced by 'cnvpytor -rd file.bam'
- Pre-calculated GC/AT content resource shipped with the CNVpytor package (GC and AT content per 100-bp bin, for GRCh37/hg19 and GRCh38)
- BAM alignment file used to generate the binned RD signal

## Outputs

- GC-corrected per-bin RD table (gc_corrected_rd.tsv, one row per genomic bin)
- Fitted RD-vs-GC bias curve plot (gc_bias_curve.png)

## How to apply

Load the binned RD signal from the file.pytor HDF5 store (via CNVpytor or h5py) for the selected bin size, then join each RD bin with the GC/AT content resource shipped inside the CNVpytor package, which tabulates GC and AT content for each 100-bp bin for GRCh37 (hg19) and GRCh38 — no reference FASTA access is needed. Model RD as a function of GC content to estimate the GC-correlated bias; this reproduces the behavior of the rd_use_gc_corr viewer parameter, which defaults to True in CNVpytor. Normalize every bin's RD by the fitted GC-bias curve and write the corrected per-bin signal to gc_corrected_rd.tsv, and plot the fitted curve to gc_bias_curve.png with matplotlib. Rationale: GC content systematically distorts binned coverage, so normalization must occur on the binned signal before partitioning and CNV calling, which follow the CNVnator-style RD binning → mean-shift → partition → call pipeline.

## Related tools

- **CNVpytor** (computes the binned RD signal, ships the per-100-bp GC/AT resource for GRCh37/GRCh38, and applies GC correction during RD analysis (rd_use_gc_corr defaults to True)) — https://github.com/abyzovlab/CNVpytor
- **h5py** (reads the RD bins from the HDF5-format .pytor file so the GC-bias fit can be reconstructed in Python)
- **numpy** (fits the RD-vs-GC bias model and normalizes per-bin RD by the fitted curve)
- **matplotlib** (plots the fitted GC-bias curve used for normalization and saves it as gc_bias_curve.png)

## Examples

```
cnvpytor -root file.pytor -rd file.bam
```

## Evaluation signals

- gc_corrected_rd.tsv exists and contains exactly one row per genomic bin at the selected bin size.
- gc_bias_curve.png shows the fitted RD-vs-GC relationship that CNVpytor uses for normalization when rd_use_gc_corr is True (the viewer default).
- After correction, the correlation between per-bin RD and GC content is markedly reduced relative to the raw binned RD signal.
- Per-bin GC values used in the fit match the shipped resource tabulation (GC and AT content per 100-bp bin) for the GRCh37/GRCh38 assembly in use.
- Copy numbers derived from the corrected signal (mean RD divided by mean autosomal RD scaled by 2) remain consistent for known CNV regions.

## Limitations

- GC and AT content are pre-calculated only for GRCh37 (hg19) and GRCh38; other reference assemblies require computing GC content externally.
- The resource is tabulated at a fixed 100-bp resolution, so analyses at other bin sizes must aggregate the 100-bp GC/AT values.
- This correction removes only GC-correlated bias; genotyping additionally supports P-mask filtering (1000 Genomes Project strict mask via rd_use_mask) for mappability, which this skill does not cover.

## Evidence

- [discussion] GC resource coverage: "For the 2 versions GRCh37 (hg19) and GRCh38 of the human reference genome, information about GC and AT content for each 100-bp bin is provided"
- [discussion] GC correction via resource data: "information about GC and AT content for each 100-bp bin is provided as resource data within the CNVpytor package"
- [discussion] binned RD storage: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [readme] CNVpytor scope: "CNVpytor is a Python package and command line tool for CNV/CNA analysis from depth-of-coverage by mapped reads developed in Abyzov Lab, Mayo Clinic."

---
name: hdf5-genomic-data-extraction
description: Use when the input is a CNVpytor .pytor (HDF5) file and you need programmatic
  access to the underlying RD signal in 100-bp bins, GC/AT bin content, or BAF likelihood
  data for downstream analysis or track export.
when_to_use_negative:
- Input is a raw BAM/CRAM alignment file rather than an existing .pytor file — the
  alignment must first be processed with 'cnvpytor -root file.pytor -rd file.bam'
- Variant-level detail is needed (e.g. individual SNP calls) — .pytor stores binned
  RD signal and BAF likelihoods, not a variant catalogue; import from VCF is a separate
  workflow step
- The goal is only to view calls interactively — the CNVpytor '-view' interactive
  mode with filtering (Q0_range, size_range, p_range) is the appropriate interface
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: Creates the .pytor HDF5 container via '-rd' and '-his' steps; its '-baf' step
    computes the BAF tracks at 10000 and 100000 bp that are then extracted
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: Python HDF5 library used to open the .pytor file and read the RD signal and
    BAF likelihood datasets
  repo: https://github.com/abyzovlab/CNVpytor
- name: pyBigWig
  role: Optional dependency installed specifically for JBrowse export functionality;
    converts per-bin RD/BAF values into coordinate-indexed track files
  repo: https://github.com/abyzovlab/CNVpytor
- name: JBrowse
  role: Genome browser that consumes the exported RD and BAF tracks at 1-, 10-, and
    100-kb resolutions, auto-selecting resolution by visualized region size
  repo: https://github.com/igvteam/igv.js
- name: Python
  role: Implementation language of CNVpytor; used for scripted HDF5 access and bin
    aggregation
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:d7be7803d8a56ff2fbfaaaee347b9c53dfd17d2406b0c5ed9f24298ab9c0452c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - JBrowse
  - Python
  - h5py
  - pyBigWig
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# hdf5-genomic-data-extraction

## Summary

Extracts read-depth (RD) signal and B-allele frequency (BAF) likelihood data from CNVpytor's HDF5-based .pytor container using h5py and Python, optionally converting per-bin values into genome-browser tracks with pyBigWig.

## When to use

Use when the input is a CNVpytor .pytor (HDF5) file and you need programmatic access to the underlying RD signal in 100-bp bins, GC/AT bin content, or BAF likelihood data for downstream analysis or track export.

## When NOT to use

- Input is a raw BAM/CRAM alignment file rather than an existing .pytor file — the alignment must first be processed with 'cnvpytor -root file.pytor -rd file.bam'
- Variant-level detail is needed (e.g. individual SNP calls) — .pytor stores binned RD signal and BAF likelihoods, not a variant catalogue; import from VCF is a separate workflow step
- The goal is only to view calls interactively — the CNVpytor '-view' interactive mode with filtering (Q0_range, size_range, p_range) is the appropriate interface

## Inputs

- CNVpytor .pytor file (HDF5 container holding RD signal in 100-bp bins and BAF likelihood data from heterozygous SNPs)
- Genome reference build selection (GRCh37/hg19 or GRCh38 resource data for GC/AT content per 100-bp bin)

## Outputs

- GC-corrected RD signal arrays per chromosome at 1000-, 10000-, and 100000-bp bin resolutions
- BAF track values per chromosome at 10000- and 100000-bp bin resolutions
- Coordinate-indexed genome-browser track files (bedGraph/JSON-compatible bigWig-style exports via pyBigWig), one per bin size

## How to apply

Open the .pytor file with h5py (an HDF5 container) and locate the RD signal stored with 100-bp bin compression; extract the GC-corrected RD signal since rd_use_gc_corr is True by default, drawing GC and AT content per 100-bp bin from the resource data bundled in the CNVpytor package. To reproduce the '-his 1000 10000 100000' convention, aggregate the 100-bp RD bins to 1000-, 10000-, and 100000-bp bin sizes. Extract the BAF track computed by the '-baf 10000 100000' step from heterozygous SNP likelihood information (symmetrized beta distribution) stored in the same file; note that newer versions (>=1.3.1) compress the BAF likelihood matrix or omit it with -nolh, recalculating likelihood at runtime if needed. Use pyBigWig, the optional dependency for JBrowse export functionality, to convert per-bin RD and BAF values into coordinate-indexed track files, one per bin size; JBrowse then auto-selects resolution based on the visualized region size.

## Related tools

- **CNVpytor** (Creates the .pytor HDF5 container via '-rd' and '-his' steps; its '-baf' step computes the BAF tracks at 10000 and 100000 bp that are then extracted) — https://github.com/abyzovlab/CNVpytor
- **h5py** (Python HDF5 library used to open the .pytor file and read the RD signal and BAF likelihood datasets) — https://github.com/abyzovlab/CNVpytor
- **pyBigWig** (Optional dependency installed specifically for JBrowse export functionality; converts per-bin RD/BAF values into coordinate-indexed track files) — https://github.com/abyzovlab/CNVpytor
- **JBrowse** (Genome browser that consumes the exported RD and BAF tracks at 1-, 10-, and 100-kb resolutions, auto-selecting resolution by visualized region size) — https://github.com/igvteam/igv.js
- **Python** (Implementation language of CNVpytor; used for scripted HDF5 access and bin aggregation) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -rd file.bam; cnvpytor -root file.pytor -his 1000 10000 100000; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- Each exported track file exists and parses without error (valid bigWig/bedGraph-compatible structure)
- Exported tracks contain expected per-chromosome bin coordinates matching the 1000/10000/100000-bp bin sizes
- Track coverage matches the set of chromosomes present in the source .pytor file
- Aggregated RD values at coarser bin sizes are consistent with means of the underlying 100-bp GC-corrected RD bins
- BAF values are present only at the 10000- and 100000-bp bin sizes, matching the '-baf 10000 100000' convention

## Limitations

- RD signal in .pytor files is stored with 100-bp bin compression, so no sub-100-bp resolution is recoverable from the container
- Versions >=1.3.1 compress or omit the BAF likelihood matrix (-nolh); with -nolh the likelihood is recalculated at runtime and the 'lh_lite' plotting parameter is used
- pyBigWig is an optional dependency; JBrowse export functionality is unavailable if it is not installed
- GC/AT bin content is only pre-packaged for GRCh37 (hg19) and GRCh38 human reference genomes

## Evidence

- [discussion] Additional compression is obtained by storing RD signal using 100-bp bins.: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [readme] pyBigWig for JBrowse export: "Optional:

* pyBigWig - for JBrowse export functionality"
- [other] JBrowse auto-selects resolution: [evidence span withheld — anchor status: not_found]
- [discussion] GC/AT resource data: "information about GC and AT content for each 100-bp bin is provided as resource data within the CNVpytor package"
- [discussion] symmetrized beta distribution: "we have to use the symmetrized beta distribution for likelihood"
- [readme] nolh option v1.3.1: "Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB"

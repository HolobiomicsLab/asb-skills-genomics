---
name: cnvpytor-pytor-file-export
description: Use when a CNVpytor .pytor file's read-depth and BAF tracks need to be
  rendered in JBrowse (or igv.js), requiring conversion of per-bin RD/BAF values into
  browser track files at 1-, 10-, and 100-kb resolutions.
when_to_use_negative:
- The input is raw BAM/VCF data that has not yet been processed through the -rd/-his/-baf
  steps into a .pytor file
- The goal is interactive CNV calling or filtering in CNVpytor's -view mode rather
  than browser visualization of signal tracks
- pyBigWig is unavailable and the ROOT/other export paths (e.g. CNVnator root import/export)
  are not applicable to the JBrowse target
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Produces and reads the .pytor HDF5 file containing RD signal in 100-bp bins
    and BAF likelihood data; its export functionality writes JBrowse track data
  repo: https://github.com/abyzovlab/CNVpytor
- name: pyBigWig
  role: Optional dependency installed for JBrowse export functionality; converts per-bin
    RD and BAF values into coordinate-indexed track files
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: Python HDF5 library used to access RD and BAF data inside the .pytor container
  repo: https://github.com/abyzovlab/CNVpytor
- name: JBrowse
  role: Target genome browser that consumes the exported RD and BAF tracks and automatically
    selects resolution based on the visualized region size
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:ecd105444ec926cbca0dad2877345289b306aa46b0240461ba821be6fbbad688
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
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cnvpytor-pytor-file-export

## Summary

Exports CNVpytor's RD and BAF signal stored in the HDF5-based .pytor file into JBrowse-consumable coordinate-indexed track files at multiple bin resolutions, enabling genome-browser visualization of copy number evidence.

## When to use

Use when a CNVpytor .pytor file's read-depth and BAF tracks need to be rendered in JBrowse (or igv.js), requiring conversion of per-bin RD/BAF values into browser track files at 1-, 10-, and 100-kb resolutions.

## When NOT to use

- The input is raw BAM/VCF data that has not yet been processed through the -rd/-his/-baf steps into a .pytor file
- The goal is interactive CNV calling or filtering in CNVpytor's -view mode rather than browser visualization of signal tracks
- pyBigWig is unavailable and the ROOT/other export paths (e.g. CNVnator root import/export) are not applicable to the JBrowse target

## Inputs

- CNVpytor .pytor file (HDF5 container with 100-bp RD bins and BAF likelihood data)
- GC-corrected RD signal (rd_use_gc_corr=True by default)
- BAF track data computed with '-baf 10000 100000' from heterozygous SNP information

## Outputs

- JBrowse track files in JSON/bedGraph-compatible form via pyBigWig, one file per bin size (1 kb, 10 kb, 100 kb)
- RD and BAF tracks at three resolutions loadable by JBrowse/igv.js

## How to apply

Load the sample's .pytor file (an HDF5 container built with h5py) that stores GC-corrected RD signal in 100-bp bins plus BAF likelihood data from heterozygous SNPs. Aggregate the rd_use_gc_corr RD signal to the three exported bin sizes 1000, 10000, and 100000, matching the '-his 1000 10000 100000' convention, and extract the BAF track produced by the '-baf 10000 100000' step. Use pyBigWig (the optional dependency installed specifically for JBrowse export) to convert per-bin RD and BAF values into coordinate-indexed track files, one file per bin size. JBrowse then automatically selects the appropriate resolution based on the size of the visualized genomic region.

## Related tools

- **CNVpytor** (Produces and reads the .pytor HDF5 file containing RD signal in 100-bp bins and BAF likelihood data; its export functionality writes JBrowse track data) — https://github.com/abyzovlab/CNVpytor
- **pyBigWig** (Optional dependency installed for JBrowse export functionality; converts per-bin RD and BAF values into coordinate-indexed track files) — https://github.com/abyzovlab/CNVpytor
- **h5py** (Python HDF5 library used to access RD and BAF data inside the .pytor container) — https://github.com/abyzovlab/CNVpytor
- **JBrowse** (Target genome browser that consumes the exported RD and BAF tracks and automatically selects resolution based on the visualized region size)

## Examples

```
cnvpytor -root file.pytor -his 1000 10000 100000
```

## Evaluation signals

- Each exported track file exists and parses without error (e.g. loadable via pyBigWig or JSON parsing)
- Track files contain the expected per-chromosome bin coordinates matching the 1000, 10000, and 100000 bin sizes
- Exported tracks cover the chromosomes present in the source .pytor file
- JBrowse/igv.js loads the tracks and swaps resolution appropriately when zooming (1 kb at fine zoom, 100 kb at genome-wide view)
- RD track values are consistent with the GC-corrected signal (rd_use_gc_corr=True) stored in the .pytor file

## Limitations

- JBrowse export requires the optional pyBigWig dependency, which is not installed with the default CNVpytor dependency set
- RD signal is compressed/stored at 100-bp bins in the .pytor file, so export resolution is limited to aggregations of that base binning
- BAF tracks are only available at 10 kb and 100 kb resolutions (the '-baf 10000 100000' convention), not at 1 kb

## Evidence

- [abstract] Data can be exported into JBrowse, while a lightweight plugin version of CNVpytor for JBrowse enables nearly instant and GUI-assisted analysis: "Data can be exported into JBrowse, while a lightweight plugin version of CNVpytor for JBrowse enables nearly instant and GUI-assisted analysis"
- [readme] pyBigWig - for JBrowse export functionality: "Optional:

* pyBigWig - for JBrowse export functionality"
- [discussion] Additional compression is obtained by storing RD signal using 100-bp bins.: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [methods] cnvpytor -root file.pytor -his 1000 10000 100000: "> cnvpytor -root file.pytor -his 1000 10000 100000"
- [other] Load the sample's .pytor file (an HDF5 container built with h5py) that holds RD signal in 100-bp bins plus BAF likelihood data.: [evidence span withheld — anchor status: not_found]

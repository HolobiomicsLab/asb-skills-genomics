---
name: read-depth-track-conversion
description: Use when a .pytor file's RD or BAF data must be surfaced as genome-browser
  track files (e.g. for JBrowse/igv.js), or when multi-resolution coverage/allele-imbalance
  tracks need to be rebuilt from CNVpytor HDF5 output.
when_to_use_negative:
- The goal is calling CNVs (e-value/Q0/size-filtered segment lists) rather than visualization
  tracks — use CNVpytor's -call/-view workflow instead
- Input is a raw BAM/VCF rather than a processed .pytor file — the RD must first be
  binned (-rd, -his) and BAF computed (-snp, -baf) before export
- pyBigWig is unavailable and the export must use a different track backend — pyBigWig
  is the designated optional dependency for this functionality
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: CNVpytor
  role: Produces and reads the .pytor HDF5 file with RD (100-bp bins) and BAF likelihood
    data that is exported to JBrowse format
  repo: https://github.com/abyzovlab/CNVpytor
- name: pyBigWig
  role: Optional dependency used to convert per-bin RD and BAF values into coordinate-indexed
    JBrowse track files
  repo: https://github.com/abyzovlab/CNVpytor
- name: h5py
  role: Python HDF5 library used to load and read the RD and BAF data inside the .pytor
    file
  repo: https://github.com/abyzovlab/CNVpytor
- name: JBrowse
  role: Genome browser into which the exported multi-resolution RD and BAF tracks
    are embedded
- name: Python
  role: Implementation language of the CNVpytor export workflow
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:f059294a95b84ed56ebbb794b4f93e8c749f5ae333421b8445d6fd1ee3991ce6
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
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# read-depth-track-conversion

## Summary

Reconstructs CNVpytor's JBrowse export: GC-corrected RD signal from a .pytor (HDF5/h5py) file and BAF tracks are converted into coordinate-indexed, JBrowse/bedGraph-compatible track files at 1-, 10-, and 100-kb bin resolutions using the optional pyBigWig dependency.

## When to use

Use when a .pytor file's RD or BAF data must be surfaced as genome-browser track files (e.g. for JBrowse/igv.js), or when multi-resolution coverage/allele-imbalance tracks need to be rebuilt from CNVpytor HDF5 output.

## When NOT to use

- The goal is calling CNVs (e-value/Q0/size-filtered segment lists) rather than visualization tracks — use CNVpytor's -call/-view workflow instead
- Input is a raw BAM/VCF rather than a processed .pytor file — the RD must first be binned (-rd, -his) and BAF computed (-snp, -baf) before export
- pyBigWig is unavailable and the export must use a different track backend — pyBigWig is the designated optional dependency for this functionality

## Inputs

- CNVpytor .pytor file (HDF5 container built with h5py) holding 100-bp RD bins and BAF likelihood data
- GC-corrected RD signal (rd_use_gc_corr=True)
- BAF track data from the -baf 10000 100000 step (heterozygous SNP-derived likelihoods)

## Outputs

- Coordinate-indexed RD track files in JSON/bedGraph-compatible form, one per bin size (1 kb, 10 kb, 100 kb)
- Coordinate-indexed BAF track files, one per bin size (10 kb, 100 kb)
- JBrowse-embeddable tracks at 3 resolutions with automatic zoom-level selection

## How to apply

Load the sample's .pytor file with h5py; it stores RD signal in 100-bp bins plus BAF likelihood data from heterozygous SNPs. Extract the GC-corrected RD signal (rd_use_gc_corr is True by default, using GC/AT content provided as resource data for GRCh37/hg19 and GRCh38) and aggregate it to 1000, 10000, and 100000 bp bins, matching the '-his 1000 10000 100000' convention. Extract the BAF track computed by the '-baf 10000 100000' step (symmetrized beta distribution likelihood over heterozygous SNP data). Use pyBigWig — the optional dependency installed specifically for JBrowse export functionality — to convert per-bin RD and BAF values into coordinate-indexed track files, one file per bin size; JBrowse then automatically selects the appropriate resolution based on the size of the visualized genomic region.

## Related tools

- **CNVpytor** (Produces and reads the .pytor HDF5 file with RD (100-bp bins) and BAF likelihood data that is exported to JBrowse format) — https://github.com/abyzovlab/CNVpytor
- **pyBigWig** (Optional dependency used to convert per-bin RD and BAF values into coordinate-indexed JBrowse track files) — https://github.com/abyzovlab/CNVpytor
- **h5py** (Python HDF5 library used to load and read the RD and BAF data inside the .pytor file) — https://github.com/abyzovlab/CNVpytor
- **JBrowse** (Genome browser into which the exported multi-resolution RD and BAF tracks are embedded)
- **Python** (Implementation language of the CNVpytor export workflow) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -his 1000 10000 100000; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- Each exported track file exists and parses without error
- Track files contain the expected per-chromosome bin coordinates at each of the 1000/10000/100000 bp bin sizes
- Track chromosome coverage matches the chromosomes present in the source .pytor file
- RD values reflect GC correction (rd_use_gc_corr=True default) and correct aggregation from 100-bp to the coarser bins
- JBrowse loads the tracks and automatically selects the appropriate resolution when zooming

## Limitations

- pyBigWig is an optional dependency; the export will fail if it is not installed
- JBrowse export covers only the three pre-computed bin resolutions (RD at 1/10/100 kb; BAF at 10/100 kb), not arbitrary bin sizes
- BAF tracks require the -baf step to have been run on heterozygous SNP data imported from a VCF; SNPs outside 1000 Genomes P-mask regions may inflate false-positive heterozygous statistics

## Evidence

- [full_text] Exported JBrowse data provide 3 resolutions of RD and BAF tracks (1, 10, and 100 kb bins): [evidence span withheld — anchor status: not_found]
- [readme] pyBigWig optional dependency for JBrowse export: "Optional:

* pyBigWig - for JBrowse export functionality"
- [abstract] JBrowse export capability in abstract: "Data can be exported into JBrowse, while a lightweight plugin version of CNVpytor for JBrowse enables nearly instant and GUI-assisted analysis"
- [discussion] HDF5 storage with 100-bp bins: "Additional compression is obtained by storing RD signal using 100-bp bins."
- [discussion] GC/AT resource data per 100-bp bin: "For the 2 versions GRCh37 (hg19) and GRCh38 of the human reference genome, information about GC and AT content for each 100-bp bin is provided"

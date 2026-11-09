---
name: baf-track-conversion
description: Use when a .pytor file contains BAF likelihood data from heterozygous
  SNPs and it must be exported into JBrowse/igv.js-compatible track files at 10-kb
  and 100-kb bin resolutions.
when_to_use_negative:
- The goal is exporting read-depth (RD) signal rather than BAF — RD is exported at
  1, 10, and 100 kb bins, not the 10/100 kb BAF resolutions
- The .pytor file lacks SNP data because no '-snp file.vcf' and '-baf 10000 100000'
  steps were run; there is no BAF likelihood data to convert
- Interactive plotting or call filtering (Q0_range, size_range, p_range) is the goal
  rather than static genome-browser track export
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Computes the BAF track from heterozygous SNP data via '-baf 10000 100000'
    and stores it in the .pytor HDF5 container that feeds the export
  repo: https://github.com/abyzovlab/CNVpytor
- name: pyBigWig
  role: Optional dependency used to convert per-bin BAF values into coordinate-indexed
    track files for JBrowse export
- name: h5py
  role: Provides access to the .pytor HDF5 container holding the BAF likelihood data
- name: JBrowse
  role: Genome browser that consumes the exported BAF tracks and auto-selects resolution
    based on visualized region size
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:71d512ca331ac28b7e2a551d6a9eeffee49d07bce52ffd241eb0082ae3fe832a
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
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# baf-track-conversion

## Summary

Converts CNVpytor's B-allele frequency (BAF) likelihood data from the .pytor HDF5 container into coordinate-indexed genome-browser track files at multiple bin resolutions for JBrowse/igv.js visualization.

## When to use

Use when a .pytor file contains BAF likelihood data from heterozygous SNPs and it must be exported into JBrowse/igv.js-compatible track files at 10-kb and 100-kb bin resolutions.

## When NOT to use

- The goal is exporting read-depth (RD) signal rather than BAF — RD is exported at 1, 10, and 100 kb bins, not the 10/100 kb BAF resolutions
- The .pytor file lacks SNP data because no '-snp file.vcf' and '-baf 10000 100000' steps were run; there is no BAF likelihood data to convert
- Interactive plotting or call filtering (Q0_range, size_range, p_range) is the goal rather than static genome-browser track export

## Inputs

- .pytor file (HDF5 via h5py) containing BAF likelihood data from heterozygous SNPs imported from a VCF
- Heterozygous SNP calls from a VCF file (SNPs and single-letter indels; all other variants ignored)
- pyBigWig library (optional dependency for JBrowse export)

## Outputs

- Coordinate-indexed BAF track files in JSON/bedGraph-compatible form at 10-kb bin resolution
- Coordinate-indexed BAF track files in JSON/bedGraph-compatible form at 100-kb bin resolution
- JBrowse-embeddable tracks where the browser auto-selects resolution by zoom level

## How to apply

Load the sample's .pytor file (an HDF5 container built with h5py) and extract the BAF track computed by the '-baf 10000 100000' step, which derives allele-imbalance signal from heterozygous SNP information stored via the '-snp file.vcf' import. BAF is exported at two bin resolutions (10 kb and 100 kb bins), unlike RD which is exported at 1, 10, and 100 kb. Use pyBigWig, the optional dependency installed specifically for JBrowse export functionality, to convert per-bin BAF values into coordinate-indexed track files in JSON/bedGraph-compatible form, one file per bin size. JBrowse automatically selects the appropriate resolution based on the size of the visualized genomic region, so all resolutions must be exported for seamless zoom behavior. Note that v1.3.1 compresses the BAF likelihood matrix, and the '-nolh' option avoids storing it entirely, in which case likelihood is calculated at run time.

## Related tools

- **CNVpytor** (Computes the BAF track from heterozygous SNP data via '-baf 10000 100000' and stores it in the .pytor HDF5 container that feeds the export) — https://github.com/abyzovlab/CNVpytor
- **pyBigWig** (Optional dependency used to convert per-bin BAF values into coordinate-indexed track files for JBrowse export)
- **h5py** (Provides access to the .pytor HDF5 container holding the BAF likelihood data)
- **JBrowse** (Genome browser that consumes the exported BAF tracks and auto-selects resolution based on visualized region size)

## Examples

```
cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- Each exported BAF track file exists and parses without error
- Each track file contains the expected per-chromosome bin coordinates at 10-kb and 100-kb bin sizes
- Track files cover all chromosomes present in the source .pytor file
- BAF values fall in the expected 0–1 allele-frequency range derived from heterozygous SNPs
- JBrowse selects the appropriate resolution automatically when zooming across the visualized region

## Limitations

- pyBigWig is only an optional dependency; if not installed, JBrowse export functionality is unavailable
- BAF is exported only at 10-kb and 100-kb resolutions, so fine-scale BAF inspection at 1-kb zoom is not supported
- Only SNPs and single-letter indels are imported from the VCF; all other variants are ignored, limiting BAF input data
- The BAF likelihood matrix increases .pytor file size; the '-nolh' option omits it and calculates likelihood at run time instead

## Evidence

- [results] Exported JBrowse data provide 3 resolutions of RD and BAF tracks (1, 10, and 100 kb bins), and JBrowse automatically selects the appropriate resolution based on the size of the visualized genomic region.: [evidence span withheld — anchor status: not_found]
- [methods] Use pyBigWig (the optional dependency installed for JBrowse export functionality) to convert the per-bin RD and BAF values into coordinate-indexed track files: [evidence span withheld — anchor status: not_found]
- [readme] pyBigWig - for JBrowse export functionality: "pyBigWig - for JBrowse export functionality"
- [abstract] CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional"

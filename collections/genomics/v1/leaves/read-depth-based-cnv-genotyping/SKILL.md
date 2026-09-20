---
name: read-depth-based-cnv-genotyping
description: Use when you need a per-region copy-number estimate from WGS read depth
  stored in a .pytor archive, e.g. to genotype a specific interval like 12:11396601-11436500
  in CNVpytor.
when_to_use_negative:
- You want de novo CNV discovery across the genome — use the -rd/-his/-partition/-call
  workflow instead of region genotyping.
- No .pytor archive exists yet; RD must first be binned into the archive with -rd
  file.bam before genotyping.
- The somatic/mosaic CNA setting calling for the joint caller prototype, which the
  article describes as still under development rather than a genotyping substitute.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3673
tools:
- name: CNVpytor
  role: Runs the -genotype command on .pytor files to compute region copy number from
    read depth, with optional BAF-based extended statistics
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor whose read-depth binning and calling calculations CNVpytor inherits
    and reimplements
- name: Python
  role: Implementation language of CNVpytor; the genotyping can also be driven from
    Python/Jupyter notebooks
  repo: https://github.com/abyzovlab/CNVpytor
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:92a1088f4f2036099edf65b61f9b388a9704530240b562244af85902e9a390c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Jupyter notebook
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# read-depth-based-cnv-genotyping

## Summary

Genotype a user-supplied genomic region with CNVpytor by computing copy number as the mean read depth within the region divided by the mean autosomal read depth, scaled by 2, optionally enriched with BAF likelihood evidence (e-value, q0, pN, maximum-likelihood BAF level) from imported SNP VCF data.

## When to use

Use when you need a per-region copy-number estimate from WGS read depth stored in a .pytor archive, e.g. to genotype a specific interval like 12:11396601-11436500 in CNVpytor.

## When NOT to use

- You want de novo CNV discovery across the genome — use the -rd/-his/-partition/-call workflow instead of region genotyping.
- No .pytor archive exists yet; RD must first be binned into the archive with -rd file.bam before genotyping.
- The somatic/mosaic CNA setting calling for the joint caller prototype, which the article describes as still under development rather than a genotyping substitute.

## Inputs

- CNVpytor .pytor (HDF5) archive containing binned RD signal
- One or more genomic region strings in chr:start-end format (e.g. 12:11396601-11436500)
- Bin sizes (e.g. 10000 and 100000)
- Optional SNP VCF imported via -snp for BAF-based extended statistics
- Optional 1000 Genomes Project strict genome mask for -rd_use_mask

## Outputs

- Per-region copy-number estimates at each requested bin size (e.g. 1.933261 and 1.937531 for 12:11396601-11436500)
- With -a: extended genotype record containing e-value, q0, pN, BAF likelihood level, homozygous count, heterozygous count, and BAF P-value

## How to apply

Load the read-depth data from the existing .pytor archive and run the CNVpytor genotype command on the requested region using bin sizes 10000 and 100000. Add the -a flag to extend the record with the binned-RD Gaussian-tail e-value, q0, pN, maximum-likelihood BAF level, homozygous/heterozygous variant counts, and the BAF-based P-value. Optionally repeat with -rd_use_mask so genotyping uses the P-filtered (1000 Genome Project strict mask) RD signal, which removes ~22% of variants but improves heterozygous SNP statistics. The copy number is computed as the mean RD within the region divided by the mean autosomal RD scaled by 2, with first/last-bin overlap weighting, so a diploid region should yield a value near 2.

## Related tools

- **CNVpytor** (Runs the -genotype command on .pytor files to compute region copy number from read depth, with optional BAF-based extended statistics) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor whose read-depth binning and calling calculations CNVpytor inherits and reimplements)
- **Python** (Implementation language of CNVpytor; the genotyping can also be driven from Python/Jupyter notebooks) — https://github.com/abyzovlab/CNVpytor

## Examples

```
cnvpytor -root file.pytor -genotype 10000 100000 -a
```

## Evaluation signals

- Diploid regions should return copy-number estimates close to 2 (e.g. 1.933261 / 1.937531 for the README example region).
- Output rows pair each queried coordinate string with one copy-number value per requested bin size (two values for '10000 100000').
- With -a, each record must contain 11 extended fields including e-value, q0, pN, BAF level, homozygous/heterozygous counts, and a BAF P-value matching the CNVpytor printout.
- Re-running with -rd_use_mask should reproduce the P-mask-filtered record (e.g. 1:800000-900000 -> 2.3012 with q0 0.0021).
- Genotype values are stable across both bin sizes for a consistent region, as in the README examples.

## Limitations

- Mask filtering (-rd_use_mask) eliminates ~22% of variants, which reduces usable signal even though heterozygous SNP statistics improve.
- The copy-number estimate derives only from mean RD ratio; accuracy depends on GC/AT correction resources and bin size choice.
- For cancer/mosaic CNAs, the article notes a more accurate dedicated approach is still under development; only a prototype somatic CNA caller is available.

## Evidence

- [discussion] Genotyping via mean RD normalized to autosomal RD scaled by 2: "The copy number of a provided genomic region is calculated as a mean RD within the region divided by mean autosomal RD scaled by 2."
- [methods] Mask-filtered genotyping option: "Genotyping using P filtered (1000 Genome Project strict mask) RD signal:"
- [readme] CLI genotype invocation with extended stats: "> cnvpytor -root file.pytor -genotype 10000 100000 -a"
- [readme] Worked example output for a region: "12:11396601-11436500	1.933261	1.937531"

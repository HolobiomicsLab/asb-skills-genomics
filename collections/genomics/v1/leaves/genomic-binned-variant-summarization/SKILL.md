---
name: genomic-binned-variant-summarization
description: Use when you have heterozygous SNP/small-indel genotype data in a VCF
  and need per-bin summaries (e.g., BAF likelihood profiles at 10 kb or 100 kb bin
  sizes) to support CNV/CNA or copy number-neutral LOH analysis.
when_to_use_negative:
- Input contains no SNP/small-indel genotype data (e.g., BAM-only, RD-only analysis)
  — BAF summarization requires VCF variant calls
- You only need read-depth CNV calling without allele imbalance evidence — the RD
  pipeline (-rd/-his/-partition/-call) suffices
- 'Variants are structural or multi-nucleotide: CNVpytor ignores all variants other
  than SNPs and single-letter indels'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: Imports VCF SNPs/small indels, computes symmetrized-beta BAF likelihoods per
    bin, and extracts per-bin maximum-likelihood BAF and variant counts
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor read-depth engine whose core (RD binning, mean-shift partitioning)
    CNVpytor inherits; context for binning framework
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Language in which CNVpytor and the BAF likelihood calculation are implemented
  repo: https://github.com/abyzovlab/CNVpytor
- name: NumPy
  role: Numerical dependency used for the discretized likelihood grid and per-bin
    likelihood multiplication
- name: SciPy
  role: Dependency supplying the beta distribution used for the symmetrized BAF likelihood
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:dab157158a006849fef2cef2e7f0094cef81f06c9590cb574ba4052a524677db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - NumPy
  - Python
  - SciPy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# genomic-binned-variant-summarization

## Summary

Summarize variant-level evidence (B-allele frequencies from SNPs and small indels imported from a VCF) into per-bin genomic profiles by multiplying per-variant likelihoods within each bin, as implemented in CNVpytor's BAF likelihood calculation.

## When to use

Use when you have heterozygous SNP/small-indel genotype data in a VCF and need per-bin summaries (e.g., BAF likelihood profiles at 10 kb or 100 kb bin sizes) to support CNV/CNA or copy number-neutral LOH analysis.

## When NOT to use

- Input contains no SNP/small-indel genotype data (e.g., BAM-only, RD-only analysis) — BAF summarization requires VCF variant calls
- You only need read-depth CNV calling without allele imbalance evidence — the RD pipeline (-rd/-his/-partition/-call) suffices
- Variants are structural or multi-nucleotide: CNVpytor ignores all variants other than SNPs and single-letter indels

## Inputs

- VCF file with SNPs and single-letter indels (optionally with a sample name)
- Sorted, indexed BAM alignment file (for optional -pileup refinement)
- CNVpytor .pytor (HDF5) root file
- Optional 1000 Genomes Project genome mask for -mask_snps filtering

## Outputs

- Per-bin BAF likelihood profiles stored in the .pytor HDF5 file (compressed since v1.3.1)
- baf_bin_table.tsv with bin coordinates, per-bin variant counts, and maximum-likelihood BAF values in [0, 0.5]

## How to apply

First import SNPs and single-letter indels from the VCF into the .pytor HDF5 file using CNVpytor's '-snp file.vcf -sample sample_name' command; all other variant types are ignored. Optionally refine variants with '-pileup file.bam' or filter them with '-mask_snps' (which removes variants in non-P regions of the 1000 Genomes Project mask, eliminating ~22% of variants while improving heterozygous SNP statistics). For each requested bin size (e.g., 10000 and 100000), compute per-variant B-allele frequency likelihoods using the symmetrized beta distribution over the discretized [0, 0.5] interval (default 101 grid points), then multiply the likelihoods of all variants falling within each bin to obtain the bin-level likelihood profile. Extract the maximum-likelihood BAF as the argmax of each bin's profile and record the number of variants per bin. Note that in CNVpytor v1.3.1+ the BAF likelihood matrix is compressed, or can be omitted entirely with '-nolh', in which case the likelihood is recomputed at run time during the '-call' step.

## Related tools

- **CNVpytor** (Imports VCF SNPs/small indels, computes symmetrized-beta BAF likelihoods per bin, and extracts per-bin maximum-likelihood BAF and variant counts) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor read-depth engine whose core (RD binning, mean-shift partitioning) CNVpytor inherits; context for binning framework) — https://github.com/abyzovlab/CNVpytor
- **Python** (Language in which CNVpytor and the BAF likelihood calculation are implemented) — https://github.com/abyzovlab/CNVpytor
- **NumPy** (Numerical dependency used for the discretized likelihood grid and per-bin likelihood multiplication)
- **SciPy** (Dependency supplying the beta distribution used for the symmetrized BAF likelihood)

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- baf_bin_table.tsv exists in TSV format and contains bin-coordinate and variant-count fields
- Every maximum-likelihood BAF value lies within the [0, 0.5] interval
- The .pytor file stores the BAF likelihood matrix compressed (or, with -nolh, is recomputed at run time during -call)
- Variant counts per bin are consistent with the number of heterozygous SNPs/small indels imported from the VCF within each bin
- Plotting the likelihood panel ('set panels rd likelihood') renders a per-bin BAF/likelihood profile without errors

## Limitations

- Only SNPs and single-letter indels from the VCF are used; all other variant types are ignored
- Mask filtering (-mask_snps) eliminates ~22% of variants, reducing the number of markers contributing to per-bin summaries (a trade-off for fewer false-positive heterozygous SNPs)
- Without -nolh, the full BAF likelihood matrix can inflate .pytor file size; since v1.3.1 it is compressed, and with -nolh it must be recomputed at run time
- The BAF signal is primary information only for copy number-neutral LOH and additional evidence for CNVs/CNAs — it is not a standalone copy-number caller

## Evidence

- [discussion] Variant import from VCF and per-bin BAF likelihood: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."
- [discussion] Symmetrized beta distribution rationale: "we have to use the symmetrized beta distribution for likelihood"
- [readme] CLI workflow from README: "> cnvpytor -root file.pytor -snp file.vcf -sample sample_name
> cnvpytor -root file.pytor -pileup file.bam                   # OPTIONAL
> cnvpytor -root file.pytor -mask_snps"
- [readme] BAF likelihood storage compression (v1.3.1): "Reduced Pytor file size by compressing the BAF likelihood matrix"
- [abstract] BAF as evidence for CNVs and primary for LOH: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs"

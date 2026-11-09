---
name: symmetrized-beta-distribution-maximum-likelihood
description: Use when WGS SNP/indel genotype data from a VCF must be converted into
  per-bin B-allele frequency likelihoods to support CNV/CNA calling or copy number-neutral
  loss of heterozygosity detection.
when_to_use_negative:
- The variant file contains no heterozygous SNP/indel data (BAF likelihoods cannot
  be computed from structural or other ignored variant types)
- The goal is read-depth-only CNV detection with no allele-imbalance evidence or LOH
  assessment
- Variants were not imported from a VCF (e.g. only -rd BAM binning has been performed)
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: Implements the -snp, -pileup, -mask_snps, and -baf commands that import variants
    and compute symmetrized beta maximum-likelihood BAF profiles per bin
  repo: https://github.com/abyzovlab/CNVpytor
- name: SciPy
  role: Provides the beta distribution used to evaluate the symmetrized beta likelihood
    over the discretized [0, 0.5] grid
- name: NumPy
  role: Numerical computation of the likelihood grid and argmax extraction of the
    maximum-likelihood BAF
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:8e9494206845b2dd398d6ddbc0d5ffebb8662f1274788b98f99523b315e3ee7d
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
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# symmetrized-beta-distribution-maximum-likelihood

## Summary

Computes per-bin B-allele frequency (BAF) likelihood profiles in CNVpytor by multiplying symmetrized beta distribution likelihoods of heterozygous SNP/indel variants within each bin, then extracting the maximum-likelihood BAF as evidence for CNVs/CNAs and copy number-neutral LOH.

## When to use

Use when WGS SNP/indel genotype data from a VCF must be converted into per-bin B-allele frequency likelihoods to support CNV/CNA calling or copy number-neutral loss of heterozygosity detection.

## When NOT to use

- The variant file contains no heterozygous SNP/indel data (BAF likelihoods cannot be computed from structural or other ignored variant types)
- The goal is read-depth-only CNV detection with no allele-imbalance evidence or LOH assessment
- Variants were not imported from a VCF (e.g. only -rd BAM binning has been performed)

## Inputs

- VCF file with SNPs and single-letter indels (optionally with a -sample name)
- Optional BAM file for -pileup variant refinement
- Optional 1000 Genomes Project P-mask via -mask_snps for variant filtering
- Bin sizes for BAF computation (e.g. 10000, 100000)

## Outputs

- Per-bin BAF likelihood profiles stored in the .pytor HDF5 file (compressed in v1.3.1+)
- Maximum-likelihood BAF per bin within the [0, 0.5] interval
- Variant counts per bin (e.g. in baf_bin_table.tsv with bin coordinates, variant counts, and max-likelihood BAF)

## How to apply

First import SNPs and single-letter indels from the VCF into the .pytor HDF5 file with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name'; all other variant types are ignored. Optionally refine variants with '-pileup file.bam' or filter them with '-mask_snps' before likelihood computation. Then run '-baf' at the target bin sizes (e.g. 10000 and 100000): for each bin, the likelihood of haplotype imbalance is evaluated using the symmetrized beta distribution over the discretized [0, 0.5] interval (default 101 grid points), and the per-variant likelihoods of all variants in the bin are multiplied to form the bin-level profile. The maximum-likelihood BAF is the argmax of the profile; the variant count per bin is also recorded. The rationale is that allelic imbalance from heterozygous sites provides evidence independent of read depth, serving as primary information for copy number-neutral LOH. From CNVpytor v1.3.1, the BAF likelihood matrix can be compressed or skipped entirely with '-nolh', in which case likelihoods are calculated at run time during the -call step.

## Related tools

- **CNVpytor** (Implements the -snp, -pileup, -mask_snps, and -baf commands that import variants and compute symmetrized beta maximum-likelihood BAF profiles per bin) — https://github.com/abyzovlab/CNVpytor
- **SciPy** (Provides the beta distribution used to evaluate the symmetrized beta likelihood over the discretized [0, 0.5] grid)
- **NumPy** (Numerical computation of the likelihood grid and argmax extraction of the maximum-likelihood BAF)

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- Every maximum-likelihood BAF value lies within the [0, 0.5] interval
- baf_bin_table.tsv exists, is valid TSV, and contains bin-coordinate and variant-count fields
- Bins with zero heterozygous variants carry no informative likelihood (variant count field is 0 and BAF is uninformative)
- The .pytor file either contains the (compressed) BAF likelihood matrix or, if -nolh was used, likelihoods are recomputed at run time during -call
- Called CNV regions that combine RD and BAF evidence should be consistent with CNVnator calls (known >99% overlap between CNVnator and CNVpytor RD-based calls)

## Limitations

- Only SNPs and single-letter indels are imported from the VCF; all other variant types are ignored
- Mask filtering (1000 Genomes P-mask) eliminates ~22% of variants, trading variant count for improved heterozygous SNP statistics
- Full BAF likelihood matrices can inflate .pytor file size; the -nolh option avoids storage but requires run-time recomputation during -call
- More accurate approaches for discovering cancer and mosaic CNAs are still under development; the somatic CNA caller is only a prototype

## Evidence

- [discussion] symmetrized beta distribution: "we have to use the symmetrized beta distribution for likelihood"
- [discussion] VCF variant import scope: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."
- [abstract] BAF as additional/primary evidence: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs"
- [readme] BAF command workflow: "> cnvpytor -root file.pytor -baf 10000 100000"
- [readme] v1.3.1 likelihood storage: "If the full BAF likelihood matrix is not stored in Pytor file, during -call step likelihood will be calculated during run time"
- [discussion] mask filtering trade-off: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"

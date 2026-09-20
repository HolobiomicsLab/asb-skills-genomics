---
name: baf-likelihood-computation
description: Use when a WGS analysis needs B-allele frequency information from VCF
  SNPs/small indels converted into per-bin likelihood profiles to support CNV/CNA
  calling or detect copy number-neutral loss of heterozygosity.
when_to_use_negative:
- The variant file contains no SNPs or single-letter indels — CNVpytor ignores all
  other variant types, so BAF likelihoods cannot be computed
- The goal is read-depth-only CNV calling with no allele-imbalance or copy number-neutral
  LOH evidence
- Strict file size constraints preclude storing the likelihood matrix and run-time
  recomputation during '-call' is not acceptable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0625
tools:
- name: CNVpytor
  role: Computes per-bin BAF likelihood via symmetrized beta distribution from VCF-imported
    SNP/small-indel data; '-baf' command with bin sizes 10000/100000; supports -pileup,
    -mask_snps, and -nolh options
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor read-depth method whose core engine CNVpytor inherits; provides
    the RD context into which BAF likelihood is integrated
- name: Python
  role: Implementation language of CNVpytor; likelihood calculation uses NumPy and
    SciPy
  repo: https://github.com/abyzovlab/CNVpytor
- name: SciPy
  role: Dependency used for the beta-distribution-based likelihood computation
- name: NumPy
  role: Dependency used for array-wise multiplication of per-variant likelihoods into
    bin-level profiles
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:c818aeac484415b06d1a5d098040a02b9d46feeca4a8aa9e4d17e9c4164991c2
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
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# baf-likelihood-computation

## Summary

Computes per-bin B-allele frequency (BAF) likelihood profiles in CNVpytor by multiplying symmetrized beta distribution likelihoods of all SNPs/small indels within each bin, using BAF as additional evidence for CNVs/CNAs and primary evidence for copy number-neutral LOH.

## When to use

Use when a WGS analysis needs B-allele frequency information from VCF SNPs/small indels converted into per-bin likelihood profiles to support CNV/CNA calling or detect copy number-neutral loss of heterozygosity.

## When NOT to use

- The variant file contains no SNPs or single-letter indels — CNVpytor ignores all other variant types, so BAF likelihoods cannot be computed
- The goal is read-depth-only CNV calling with no allele-imbalance or copy number-neutral LOH evidence
- Strict file size constraints preclude storing the likelihood matrix and run-time recomputation during '-call' is not acceptable

## Inputs

- VCF file with SNPs and single-letter indels (e.g. from matched sample, with -sample sample_name selection)
- Optional BAM alignment file for '-pileup' variant refinement
- Optional 1000 Genomes Project P-mask regions for '-mask_snps' variant filtering
- Requested bin sizes (e.g. 10000 and 100000 bp)

## Outputs

- Per-bin BAF likelihood profiles stored in the .pytor HDF5 file (symmetrized beta likelihood matrix)
- Bin-level maximum-likelihood BAF values within [0, 0.5] and per-bin variant counts (e.g. written to baf_bin_table.tsv with bin coordinates, variant counts, and maximum-likelihood BAF)
- Likelihood tracks usable as plot panels ('set panels rd likelihood') and as input to the -call step

## How to apply

Import SNPs and single-letter indels from the VCF into a .pytor HDF5 file with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name'; all other variant types are ignored. Optionally refine variants with '-pileup file.bam' or filter them with '-mask_snps' before computing likelihoods. For each requested bin size (e.g. 'cnvpytor -root file.pytor -baf 10000 100000'), calculate the likelihood of every variant's B-allele frequency using the symmetrized beta distribution over the discretized [0, 0.5] interval (default 101 grid points), then multiply the per-variant likelihoods of all variants falling within each bin to obtain the bin-level likelihood profile. Extract the maximum-likelihood BAF as the argmax of each bin's profile and record the variant count per bin. From version 1.3.1, the full BAF likelihood matrix is compressed in the .pytor file, can be skipped entirely with '-nolh', and is recalculated at run time during '-call' if not stored.

## Related tools

- **CNVpytor** (Computes per-bin BAF likelihood via symmetrized beta distribution from VCF-imported SNP/small-indel data; '-baf' command with bin sizes 10000/100000; supports -pileup, -mask_snps, and -nolh options) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor read-depth method whose core engine CNVpytor inherits; provides the RD context into which BAF likelihood is integrated)
- **Python** (Implementation language of CNVpytor; likelihood calculation uses NumPy and SciPy) — https://github.com/abyzovlab/CNVpytor
- **SciPy** (Dependency used for the beta-distribution-based likelihood computation)
- **NumPy** (Dependency used for array-wise multiplication of per-variant likelihoods into bin-level profiles)

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- baf_bin_table.tsv exists in TSV format with bin-coordinate and variant-count fields
- Every maximum-likelihood BAF value lies within the [0, 0.5] interval
- Per-bin variant counts match the number of SNPs/small indels imported from the VCF that fall in each bin
- The .pytor file stores the (compressed) BAF likelihood matrix, or '-nolh' was set and likelihood is recomputed during '-call' without error
- Bins in copy number-neutral LOH regions show maximum-likelihood BAF shifted away from ~0.5, consistent with allele imbalance

## Limitations

- Only SNPs and single-letter indels from the VCF are used; all other variant types are ignored
- Mask filtering eliminates ~22% of variants (1000 Genomes P-mask), reducing variant density but improving heterozygous SNP statistics
- The full BAF likelihood matrix can inflate the .pytor file; from v1.3.1 it is compressed or omitted with '-nolh', in which case likelihood is recalculated at run time during '-call'
- BAF likelihood serves as primary information only for copy number-neutral LOH; for CNVs/CNAs it is auxiliary to read depth

## Evidence

- [discussion] CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored.: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."
- [discussion] we have to use the symmetrized beta distribution for likelihood: "we have to use the symmetrized beta distribution for likelihood"
- [abstract] CNVpytor uses B-allele frequency likelihood information from SNPs and small indels as additional evidence for CNVs/CNAs and primary information for copy number-neutral losses of heterozygosity: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs and as primary information for copy"
- [readme] > cnvpytor -root file.pytor -baf 10000 100000: "> cnvpytor -root file.pytor -baf 10000 100000"
- [readme] Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size: "Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB"

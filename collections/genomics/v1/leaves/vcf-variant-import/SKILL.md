---
name: vcf-variant-import
description: 'Use when starting a CNVpytor analysis from VCF variant calls: you have
  a VCF with SNPs and small indels for a sample and need them stored in a .pytor file
  before -baf, -mask_snps, or -genotype steps.'
when_to_use_negative:
- The variants of interest are multi-base substitutions, large indels, or structural
  variants — CNVpytor ignores all VCF entries other than SNPs and single-letter indels.
- The analysis goal is read-depth-only CNV calling with no allele-imbalance evidence;
  use the -rd/-his/-partition/-call pipeline instead.
- You need variant filtering against a different mask than the 1000 Genomes Project
  strict (P) mask — -mask_snps is specifically tied to that resource.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3577
tools:
- name: CNVpytor
  role: Command-line tool that parses the VCF and stores SNP/single-letter-indel variant
    data in the .pytor HDF5 root file via the -snp command.
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Implementation language of CNVpytor; the import can also be driven through
    the Python library / Jupyter notebook interface.
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor tool whose core engine CNVpytor inherits; provides context for
    the read-depth workflow into which imported SNPs are integrated.
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:5b11285e62682417722b27a0f48d4adad3f6d00e5e2d148c20f8d9a11de73f1a
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
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# vcf-variant-import

## Summary

Import SNP and single-letter indel variant data from a VCF file into a CNVpytor .pytor (HDF5) root file, enabling downstream BAF likelihood and copy-number analysis. This is the prerequisite step for using allele imbalance as evidence for CNVs/CNAs.

## When to use

Use when starting a CNVpytor analysis from VCF variant calls: you have a VCF with SNPs and small indels for a sample and need them stored in a .pytor file before -baf, -mask_snps, or -genotype steps.

## When NOT to use

- The variants of interest are multi-base substitutions, large indels, or structural variants — CNVpytor ignores all VCF entries other than SNPs and single-letter indels.
- The analysis goal is read-depth-only CNV calling with no allele-imbalance evidence; use the -rd/-his/-partition/-call pipeline instead.
- You need variant filtering against a different mask than the 1000 Genomes Project strict (P) mask — -mask_snps is specifically tied to that resource.

## Inputs

- VCF file containing SNPs and single-letter indels (other variant types are ignored)
- Sample name specified via -sample
- Optional: BAM alignment file for the optional -pileup refinement step
- Target .pytor root file (HDF5), created if absent

## Outputs

- .pytor (HDF5) root file with stored per-variant SNP/indel data (position, B-allele, depths, genotype fields)
- Variant records available for downstream -baf, -mask_snps, and -genotype commands

## How to apply

Run `cnvpytor -root file.pytor -snp file.vcf -sample sample_name` to load variant data into the .pytor HDF5 root file. CNVpytor imports information about SNPs and single-letter indels from the VCF; all other variant types are ignored. Optionally refine the imported variants with `cnvpytor -root file.pytor -pileup file.bam` (recomputes variant depth from the BAM) or filter them with `cnvpytor -root file.pytor -mask_snps`, which removes variants located in non-P regions of the 1000 Genomes Project strict mask (masking eliminates ~22% of variants but improves heterozygous SNP statistics by removing false-positive heterozygous SNPs caused by systematic misalignment, incomplete reference, or site-specific sequencing noise). The imported variants then serve as input for per-bin BAF likelihood computation (e.g. `-baf 10000 100000`) using the symmetrized beta distribution. The -sample flag is required to associate variants with the correct sample in multi-sample .pytor files.

## Related tools

- **CNVpytor** (Command-line tool that parses the VCF and stores SNP/single-letter-indel variant data in the .pytor HDF5 root file via the -snp command.) — https://github.com/abyzovlab/CNVpytor
- **Python** (Implementation language of CNVpytor; the import can also be driven through the Python library / Jupyter notebook interface.) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor tool whose core engine CNVpytor inherits; provides context for the read-depth workflow into which imported SNPs are integrated.)

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name
```

## Evaluation signals

- The .pytor file exists and contains stored heterozygous variant counts queryable before and after masking.
- After running -mask_snps, the fraction of removed variants is approximately 0.22, consistent with the paper's report of ~22% eliminated.
- Only SNPs and single-letter indels from the VCF are present in the .pytor file; all other variant types are absent (CNVpytor ignores them by design).
- Downstream -baf computation on imported variants yields per-bin maximum-likelihood BAF values that all lie within the [0, 0.5] interval.
- Retained (P-mask) variants' BAF values closely follow the expected binomial distribution, indicating removal of outlier heterozygous variants.

## Limitations

- All VCF variant types other than SNPs and single-letter indels are silently ignored during import.
- The -mask_snps filter removes ~22% of variants, which reduces the number of available SNPs; the trade-off is fewer false-positive heterozygous SNPs and improved statistics.
- For version 1.3.1+ the full BAF likelihood matrix may be omitted from the .pytor file with -nolh, in which case likelihoods are recalculated at the -call step (and the 'lh_lite' plotting parameter is used).

## Evidence

- [discussion] CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored.: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."
- [readme] ### Importing and using single nucleotide polymorphism data: "### Importing and using single nucleotide polymorphism data:
> cnvpytor -root file.pytor -snp file.vcf -sample sample_name"
- [discussion] Mask filtering eliminates ~22% of variants but improves statistics of heterozygous SNPs: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [discussion] Calculating BAF likelihood function using symmetrized beta distribution: "we have to use the symmetrized beta distribution for likelihood"
- [readme] Option to avoid storing the full BAF likelihood matrix (-nolh): "Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB"

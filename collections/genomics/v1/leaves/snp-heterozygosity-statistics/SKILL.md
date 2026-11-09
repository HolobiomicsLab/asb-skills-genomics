---
name: snp-heterozygosity-statistics
description: Use when heterozygous variant BAF values show outlier dispersion from
  systematic misalignment, incomplete reference, or site-specific sequencing noise,
  and you need to quantify how P-mask filtering improves SNP statistics.
when_to_use_negative:
- Input variants are not from a human WGS VCF aligned to GRCh37 (hg19) or GRCh38 —
  the mask and BAF likelihood machinery assume the human reference genome context
- The goal is to filter CNV calls (by size, e-val1, Q0, pN, dG) rather than to remove
  SNP variants in non-P regions — use the view-mode call filters instead
- You need copy number genotyping from read depth; the related but separate -rd_use_mask
  option applies the P mask to RD signal, not to SNP variant counts
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2885
tools:
- name: CNVpytor
  role: CLI/Python tool that imports SNP data from VCF into a .pytor file and applies
    the -mask_snps P-mask variant filter
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Language used to query heterozygous variant counts from the .pytor file and
    compute/write the removed-fraction summary CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:e566c4330c0f32a8231c8914090084afac4ee269af85c2a25d7fe7c17ae012a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# snp-heterozygosity-statistics

## Summary

Quantify how CNVpytor's 1000 Genomes Project strict-mask (P-mask) filtering of SNPs removes outlier BAF heterozygous variants, reproducing the paper's ~22% removal statistic and verifying retained variants follow expected BAF dispersion.

## When to use

Use when heterozygous variant BAF values show outlier dispersion from systematic misalignment, incomplete reference, or site-specific sequencing noise, and you need to quantify how P-mask filtering improves SNP statistics.

## When NOT to use

- Input variants are not from a human WGS VCF aligned to GRCh37 (hg19) or GRCh38 — the mask and BAF likelihood machinery assume the human reference genome context
- The goal is to filter CNV calls (by size, e-val1, Q0, pN, dG) rather than to remove SNP variants in non-P regions — use the view-mode call filters instead
- You need copy number genotyping from read depth; the related but separate -rd_use_mask option applies the P mask to RD signal, not to SNP variant counts

## Inputs

- VCF file with SNPs and single-letter indels (heterozygous variant calls with B-allele frequencies)
- CNVpytor .pytor (HDF5) root file with imported SNP data
- 1000 Genomes Project strict mask (P regions) bundled with CNVpytor

## Outputs

- mask_filtering_summary.csv with total, retained, and removed heterozygous variant counts and the removed fraction (~0.22)
- P-mask-filtered SNP set stored in the .pytor file after -mask_snps

## How to apply

Import SNP data from a VCF into a .pytor root file with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name'. Apply the P-mask variant filter by running 'cnvpytor -root file.pytor -mask_snps', which removes variants located in non-P regions of the 1000 Genomes Project strict mask (~74% of mask bases are marked passed). Query stored heterozygous variant counts before and after masking and compute the removed fraction, which should be approximately 0.22 per the paper. The rationale is that although ~22% of variants are discarded, fewer false-positive heterozygous SNPs improves the statistics: retained variants' BAF values closely follow the expected binomial distribution, so downstream BAF likelihood calculations (based on the symmetrized beta distribution) are not corrupted by systematic misalignment or site-specific noise. Write total, retained, and removed counts plus the removed fraction to a summary CSV and validate the fraction against ~0.22.

## Related tools

- **CNVpytor** (CLI/Python tool that imports SNP data from VCF into a .pytor file and applies the -mask_snps P-mask variant filter) — https://github.com/abyzovlab/CNVpytor
- **Python** (Language used to query heterozygous variant counts from the .pytor file and compute/write the removed-fraction summary CSV)

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name && cnvpytor -root file.pytor -mask_snps
```

## Evaluation signals

- Removed fraction of heterozygous variants after -mask_snps is approximately 0.22, matching the paper's reported ~22%
- BAF values of retained heterozygous variants closely follow the expected binomial distribution (no outlier dispersion)
- Total = retained + removed variant counts hold in mask_filtering_summary.csv
- All variants in the filtered set lie within P (passed) regions of the 1000 Genomes Project strict mask (~74% of mask bases marked passed)

## Limitations

- Mask filtering eliminates ~22% of variants — a real data loss — justified only because fewer false-positive heterozygous SNPs improves downstream statistics
- Only SNPs and single-letter indels are imported from the VCF; all other variant types are ignored
- Outlier BAF dispersion arises from systematic misalignment, incomplete reference genome, and site-specific sequencing noise — the mask mitigates but does not correct underlying alignment artifacts

## Evidence

- [discussion] Mask filtering removes outlier BAF heterozygous variants: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [discussion] P-mask is used to filter variants in non-P regions: "We use that mask to filter out variants called in non-P regions."
- [discussion] ~74% of 1000 Genomes mask bases are passed: [evidence span withheld — anchor status: not_found]
- [methods] SNP import command from VCF: "### Importing and using single nucleotide polymorphism data:
> cnvpytor -root file.pytor -snp file.vcf -sample sample_name"
- [discussion] Only SNPs and single-letter indels are imported: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."

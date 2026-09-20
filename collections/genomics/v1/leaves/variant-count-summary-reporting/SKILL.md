---
name: variant-count-summary-reporting
description: 'Use when validating BAF/SNP filtering in a CNVpytor .pytor analysis:
  i.e., when you need to count heterozygous variants before and after -mask_snps P-mask
  filtering and reproduce the paper''s ~22% removal statistic.'
when_to_use_negative:
- Input VCF contains variant classes CNVpytor ignores (CNVpytor imports only SNPs
  and single-letter indels; all other variants are ignored), so counts would be misleading
- The goal is RD-based CNV calling or call filtering (e.g., Q0_range, size_range,
  p_range in view mode), not SNP/BAF mask-filtering statistics
- Analysis on a non-human genome or assembly where the 1000 Genomes Project strict
  mask (built for hg19/GRCh37-style human data) does not apply
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Imports SNPs from VCF into the .pytor root file and applies the 1000 Genomes
    Project P-mask via -mask_snps; stores heterozygous variant counts before/after
    masking
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Used (with pip-installed cnvpytor) to query stored counts from the .pytor
    file, compute the removed fraction, and write mask_filtering_summary.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:c551a5f2232ab551cb448c14bd427f3c75f76bd10594cb2d9bf4266c8d97fd42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# variant-count-summary-reporting

## Summary

Quantifies heterozygous variant retention under CNVpytor's 1000 Genomes Project strict-mask (P-mask) SNP filter, reporting total, retained, and removed counts plus the removed fraction (~22%) as a CSV summary.

## When to use

Use when validating BAF/SNP filtering in a CNVpytor .pytor analysis: i.e., when you need to count heterozygous variants before and after -mask_snps P-mask filtering and reproduce the paper's ~22% removal statistic.

## When NOT to use

- Input VCF contains variant classes CNVpytor ignores (CNVpytor imports only SNPs and single-letter indels; all other variants are ignored), so counts would be misleading
- The goal is RD-based CNV calling or call filtering (e.g., Q0_range, size_range, p_range in view mode), not SNP/BAF mask-filtering statistics
- Analysis on a non-human genome or assembly where the 1000 Genomes Project strict mask (built for hg19/GRCh37-style human data) does not apply

## Inputs

- Variant (VCF) file with SNP calls for the sample
- CNVpytor .pytor root file holding imported SNP/BAF data
- 1000 Genomes Project strict mask (P regions), bundled/applied via cnvpytor -mask_snps

## Outputs

- Heterozygous variant counts before masking (total)
- Heterozygous variant counts retained after masking
- Removed variant count and removed fraction (~0.22 expected)
- Summary CSV (e.g., mask_filtering_summary.csv) with total, retained, removed, removed_fraction columns

## How to apply

Import SNP data from a VCF into a .pytor root file with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name' (only SNPs and single-letter indels are imported; other variants are ignored). Run 'cnvpytor -root file.pytor -mask_snps' to filter out variants located in non-P regions of the 1000 Genomes Project strict mask, which marks ~74% of bases as passed (P), corresponding to ~77% of non-N bases. Query the stored heterozygous variant counts before and after masking from the .pytor file, compute removed = total − retained, and the removed fraction = removed/total. The paper reports masking eliminates ~22% of variants, while improving heterozygous SNP statistics by removing false-positive heterozygous SNPs caused by systematic misalignment, incomplete reference genome, or site-specific sequencing noise. Write total, retained, removed counts and the removed fraction to a summary CSV (e.g., mask_filtering_summary.csv).

## Related tools

- **CNVpytor** (Imports SNPs from VCF into the .pytor root file and applies the 1000 Genomes Project P-mask via -mask_snps; stores heterozygous variant counts before/after masking) — https://github.com/abyzovlab/CNVpytor
- **Python** (Used (with pip-installed cnvpytor) to query stored counts from the .pytor file, compute the removed fraction, and write mask_filtering_summary.csv)

## Examples

```
cnvpytor -root file.pytor -mask_snps
```

## Evaluation signals

- Removed fraction in mask_filtering_summary.csv is approximately 0.22, matching the paper's report that masking eliminates ~22% of variants
- Removed count equals total minus retained (consistency check on the three counts in the CSV)
- Retained fraction is compatible with the mask's scope: ~74% of bases marked passed (P), ~77% of non-N bases
- BAF values of retained heterozygous variants closely follow the expected binomial distribution after filtering
- Pre-mask total equals the number of heterozygous SNPs imported from the VCF via -snp -sample

## Limitations

- Mask filtering removes ~22% of variants unconditionally, including true variants, trading sensitivity for improved heterozygous-SNP statistics
- Only SNPs and single-letter indels contribute to counts; other variants in the VCF are silently ignored
- The ~22% figure is dataset-dependent (paper used example data such as the CEPH trio/Platinum Genomes); other samples may deviate
- The strict mask excludes ~26% of bases, so filtered statistics describe only P-region variation

## Evidence

- [discussion] This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [discussion] We use that mask to filter out variants called in non-P regions.: "We use that mask to filter out variants called in non-P regions."
- [methods] > cnvpytor -root file.pytor -mask_snps                         # OPTIONAL: "> cnvpytor -root file.pytor -mask_snps                         # OPTIONAL"
- [readme] > cnvpytor -root file.pytor -snp file.vcf -sample sample_name: "> cnvpytor -root file.pytor -snp file.vcf -sample sample_name"
- [discussion] There are ∼74% of bases marked passed (P), which corresponds to ∼77% of non-N bases: "There are ∼74% of bases marked passed (P), which corresponds to ∼77% of non-N bases"

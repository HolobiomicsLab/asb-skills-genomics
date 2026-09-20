---
name: genome-mask-variant-filtering
description: Use when a VCF of SNPs/indels has been imported into CNVpytor and heterozygous
  BAF dispersion shows outliers from misalignment or reference problems; the mask
  filter removes non-P-region variants before BAF-based calling.
when_to_use_negative:
- The analysis only uses read depth without SNP/BAF evidence, so heterozygous-SNP
  contamination cannot affect results (masking is done via -mask_snps on imported
  VCF variants)
- Variants were not imported from a VCF into the .pytor file, or the sample is non-human
  / not alignable to the GRCh37/GRCh38 reference builds for which the 1000 Genomes
  mask applies
- Calling somatic CNAs with the prototype joint caller, where a more accurate mask-free
  approach is still under development
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CNVpytor
  role: Command-line tool whose -mask_snps step removes variants in non-P regions
    of the 1000 Genomes Project strict mask; -rd_use_mask applies the P filter to
    RD signal during genotyping
  repo: https://github.com/abyzovlab/CNVpytor
- name: Python
  role: Language of CNVpytor and the scripting layer used to query variant counts
    and compute the removed fraction (~22%)
provenance:
  source_task_ids:
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:879472ce07f94917e2871ce6a031ff774b6837c2ea719cf5c786a6c2a4ad7792
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# genome-mask-variant-filtering

## Summary

Filters imported SNP variants to those falling in 'P' (passed) regions of the 1000 Genomes Project strict mask, removing ~22% of variants whose heterozygous BAF values are false positives caused by systematic misalignment, incomplete reference, or site-specific sequencing noise.

## When to use

Use when a VCF of SNPs/indels has been imported into CNVpytor and heterozygous BAF dispersion shows outliers from misalignment or reference problems; the mask filter removes non-P-region variants before BAF-based calling.

## When NOT to use

- The analysis only uses read depth without SNP/BAF evidence, so heterozygous-SNP contamination cannot affect results (masking is done via -mask_snps on imported VCF variants)
- Variants were not imported from a VCF into the .pytor file, or the sample is non-human / not alignable to the GRCh37/GRCh38 reference builds for which the 1000 Genomes mask applies
- Calling somatic CNAs with the prototype joint caller, where a more accurate mask-free approach is still under development

## Inputs

- CNVpytor .pytor root file containing SNP data imported from a VCF ('cnvpytor -snp file.vcf -sample sample_name')
- 1000 Genomes Project strict genome mask (bundled resource applied by -mask_snps; ~74% of bases marked passed)
- Optional: BAM alignment file for the -pileup step

## Outputs

- Mask-filtered heterozygous variant set stored in the .pytor file (non-P-region variants removed)
- Variant counts before and after masking; removed fraction expected to be ~0.22
- Filtered SNP input for downstream BAF likelihood computation (-baf) and genotyping

## How to apply

After importing SNP data from a VCF with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name', run 'cnvpytor -root file.pytor -mask_snps' to drop variants located in non-P regions of the 1000 Genomes Project strict mask (~74% of mask bases are marked passed, corresponding to ~77% of non-N bases). The rationale is that B-allele frequency likelihood relies on heterozygous SNPs, so false-positive heterozygous calls outside accessible regions corrupt BAF statistics; the paper reports masking eliminates ~22% of variants while improving heterozygous-SNP statistics, and retained variants closely follow the expected binomial distribution. The related '-rd_use_mask' option applies the same P-filter to read-depth signal during genotyping. Optionally run '-pileup file.bam' beforehand, then proceed to '-baf 10000 100000' to compute BAF likelihood on the filtered variant set.

## Related tools

- **CNVpytor** (Command-line tool whose -mask_snps step removes variants in non-P regions of the 1000 Genomes Project strict mask; -rd_use_mask applies the P filter to RD signal during genotyping) — https://github.com/abyzovlab/CNVpytor
- **Python** (Language of CNVpytor and the scripting layer used to query variant counts and compute the removed fraction (~22%))

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name && cnvpytor -root file.pytor -pileup file.bam && cnvpytor -root file.pytor -mask_snps
```

## Evaluation signals

- The fraction of removed variants is approximately 0.22, matching the paper's report that masking eliminates ~22% of variants
- BAF values of retained heterozygous variants closely follow the expected binomial distribution (no outlier dispersion)
- The mask_snps step is marked OPTIONAL in the workflow and runs without error after -snp import and before -baf
- Genotyping with -rd_use_mask returns copy-number and extended statistics (e-value, q0, pN, BAF level, het/hom counts, BAF P-value) consistent with unmasked output for diploid regions (copy number near 2)

## Limitations

- Masking removes ~22% of variants, so allele-imbalance sensitivity in excluded non-P regions is reduced; the benefit is fewer false-positive heterozygous SNPs and improved statistics
- Only ~74% of mask bases are marked passed (~77% of non-N bases), so the accessible genome is a subset of the reference
- CNVpytor imports only SNPs and single-letter indels from the VCF; all other variant types are ignored

## Evidence

- [discussion] mask filtering eliminates ~22% of variants but improves heterozygous-SNP statistics: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [discussion] ~74% of mask bases are marked passed: "There are ∼74% of bases marked passed (P), which corresponds to ∼77% of non-N bases"
- [discussion] mask is used to filter out non-P variants: "We use that mask to filter out variants called in non-P regions."
- [readme] -mask_snps is an optional step in the SNP workflow: "> cnvpytor -root file.pytor -mask_snps                         # OPTIONAL"
- [methods] P-filtered RD genotyping command: "Genotyping using P filtered (1000 Genome Project strict mask) RD signal:"

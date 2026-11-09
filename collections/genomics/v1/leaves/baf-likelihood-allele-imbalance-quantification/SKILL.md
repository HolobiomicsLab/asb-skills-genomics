---
name: baf-likelihood-allele-imbalance-quantification
description: Use when read depth alone is insufficient — e.g. suspected copy number-neutral
  LOH, or when a CNV/CNA/genotype call needs maximum-likelihood BAF level, homozygous/heterozygous
  counts, and a BAF P-value from SNP data.
when_to_use_negative:
- No SNP/genotype VCF data exist for the sample — BAF likelihood cannot be computed,
  so the RD-only workflow (-rd, -his, -partition, -call) must be used instead.
- The variant file contains variant classes other than SNPs and single-letter indels;
  CNVpytor ignores all other variants, so BAF evidence would be silently incomplete.
- The goal is purely coverage-based de novo CNV discovery without any variant-level
  validation or LOH assessment.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_0203
tools:
- name: CNVpytor
  role: Imports VCF SNPs/single-letter indels, computes per-bin BAF likelihood with
    a symmetrized beta distribution, stores it in the .pytor archive, and reports
    maximum-likelihood BAF level and BAF P-value in '-genotype -a' output
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: RD-based predecessor whose core engine CNVpytor inherits; BAF allele-imbalance
    evidence extends CNVnator's coverage-only CNV/CNA analysis
- name: igv.js
  role: Browser visualization of .pytor RD and BAF/likelihood tracks alongside CNV
    calls
  repo: https://github.com/igvteam/igv.js
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:3bb61110cfc488169fa70edab48e6f6deeb68f882f7f41b092163cd684677e07
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - Python
  - igv.js
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# baf-likelihood-allele-imbalance-quantification

## Summary

CNVpytor converts B-allele frequencies of SNPs and single-letter indels from a VCF into per-bin BAF likelihood profiles (symmetrized beta distribution), used as additional evidence for CNVs/CNAs and as primary evidence for copy number-neutral losses of heterozygosity.

## When to use

Use when read depth alone is insufficient — e.g. suspected copy number-neutral LOH, or when a CNV/CNA/genotype call needs maximum-likelihood BAF level, homozygous/heterozygous counts, and a BAF P-value from SNP data.

## When NOT to use

- No SNP/genotype VCF data exist for the sample — BAF likelihood cannot be computed, so the RD-only workflow (-rd, -his, -partition, -call) must be used instead.
- The variant file contains variant classes other than SNPs and single-letter indels; CNVpytor ignores all other variants, so BAF evidence would be silently incomplete.
- The goal is purely coverage-based de novo CNV discovery without any variant-level validation or LOH assessment.

## Inputs

- VCF file containing SNPs and single-letter indels with an explicit sample name (-snp file.vcf -sample sample_name)
- BAM alignment file for optional SNP pileup (-pileup)
- Existing .pytor (HDF5) archive holding the sample's RD signal
- 1000 Genomes Project strict P-mask, applied via -mask_snps (OPTIONAL step)

## Outputs

- Per-bin BAF likelihood matrix / likelihood profiles stored in the .pytor archive at the chosen bin sizes (e.g. 10000 and 100000)
- Likelihood (lh) plot panels rendered in -view interactive mode
- Extended genotype records (-genotype ... -a) containing maximum-likelihood BAF level, homozygous and heterozygous variant counts, and the BAF-based P-value
- Mask-filtered SNP set after -mask_snps (variants in non-P 1000 Genomes regions removed)

## How to apply

Import SNPs and single-letter indels from the variant VCF into the .pytor HDF5 archive with 'cnvpytor -root file.pytor -snp file.vcf -sample sample_name' (optionally '-pileup file.bam'); CNVpytor ignores all other variant types. Optionally run '-mask_snps' to filter variants in non-P regions of the 1000 Genomes Project strict mask — this eliminates ~22% of variants but improves heterozygous SNP statistics, since ~74% of mask bases pass. Compute per-bin BAF likelihood with '-baf 10000 100000', modelling allele balance with a symmetrized beta distribution; since v1.3.1 the likelihood matrix is stored compressed in the .pytor file, or omitted entirely with '-nolh' (final file <50 MB) and recalculated at run time during '-call'. Inspect results via the likelihood (lh) panel in '-view' mode (parameter 'lh_lite' when the full matrix was not stored) or via the extended '-genotype ... -a' record, which reports the maximum-likelihood BAF level, homozygous and heterozygous variant counts, and the BAF-based P-value next to the RD-derived copy number. Treat BAF likelihood as additional evidence for CNVs/CNAs but as primary evidence for copy number-neutral LOH, where RD stays near 2 while allele balance shifts.

## Related tools

- **CNVpytor** (Imports VCF SNPs/single-letter indels, computes per-bin BAF likelihood with a symmetrized beta distribution, stores it in the .pytor archive, and reports maximum-likelihood BAF level and BAF P-value in '-genotype -a' output) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (RD-based predecessor whose core engine CNVpytor inherits; BAF allele-imbalance evidence extends CNVnator's coverage-only CNV/CNA analysis)
- **igv.js** (Browser visualization of .pytor RD and BAF/likelihood tracks alongside CNV calls) — https://github.com/igvteam/igv.js

## Examples

```
cnvpytor -root file.pytor -snp file.vcf -sample sample_name; cnvpytor -root file.pytor -baf 10000 100000
```

## Evaluation signals

- After the -baf step, the .pytor file contains a BAF likelihood dataset for the requested bin sizes (e.g. 10000 and 100000).
- The '-genotype 10000 100000 -a' output includes the maximum-likelihood BAF level, homozygous and heterozygous variant counts, and a BAF-based P-value alongside the copy-number estimate.
- After -mask_snps, roughly 22% of variants are removed and heterozygous SNP statistics improve; remaining variants fall inside the ~74% of 1000 Genomes mask bases marked passed (P).
- In -view mode the likelihood panel renders a single lh profile (default lh_colors yellow), or switches to the 'lh_lite' parameter when the full BAF likelihood matrix was not stored (-nolh).
- For copy number-neutral LOH, the RD-derived copy number stays plausibly near 2 while the BAF profile shows shifted allele balance, confirming BAF as the primary evidence.

## Limitations

- Only SNPs and single-letter indels are imported from the VCF; all other variants are ignored.
- Storing the full BAF likelihood matrix inflates the .pytor file; -nolh avoids this (<50 MB) but forces likelihood calculation at run time during -call.
- Mask filtering against the 1000 Genomes strict mask eliminates ~22% of variants, reducing SNP density in non-P regions.
- Likelihood must be modelled with a symmetrized beta distribution rather than a plain beta distribution.

## Evidence

- [abstract] CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs and as primary information for copy: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs and as primary information for copy"
- [discussion] CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored.: "CNVpytor imports information about SNPs and single-letter indels from the variant (VCF) file. All other variants are ignored."
- [discussion] we have to use the symmetrized beta distribution for likelihood: "we have to use the symmetrized beta distribution for likelihood"
- [discussion] This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [readme] Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB: "Option to avoid storing the full BAF likelihood matrix (-nolh), drastically reducing the final Pytor file size to less than 50 MB"

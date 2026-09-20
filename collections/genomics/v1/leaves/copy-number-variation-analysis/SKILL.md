---
name: copy-number-variation-analysis
description: Use when analyzing WGS alignment data (BAM) for CNV/CNA discovery and
  genotyping, especially when RD signal alone is insufficient and SNP/indel allele
  imbalance (BAF) evidence or copy-neutral LOH detection is needed.
when_to_use_negative:
- Input is already a called CNV segment/feature table rather than raw alignment (BAM)
  and variant (VCF) data — CNVpytor's RD binning and BAF likelihood steps operate
  on alignments and variant-level genotypes
- Analysis requires detecting small insertions/deletions or structural variants without
  copy-number change — the tool targets CNV/CNA from read depth and allele imbalance,
  not balanced SVs
- Somatic/cancer CNA calling requiring high sensitivity in impure samples — the article
  notes the somatic CNA caller is only a prototype and a more accurate approach for
  cancer and mosaic CNAs is under development
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: CNVpytor
  role: 'Primary tool: performs RD binning, GC correction via packaged GC/AT resource,
    mean-shift partitioning, CNV calling, BAF likelihood computation, genotyping,
    filtering, annotation, and visualization'
  repo: https://github.com/abyzovlab/CNVpytor
- name: CNVnator
  role: Predecessor whose reimplemented core engine (RD binning, mean-shift algorithm,
    partitioning, calling) CNVpytor inherits; used as baseline for call-overlap comparison
- name: Python
  role: Implementation language and scripting environment (with h5py, numpy, scipy,
    matplotlib) for reconstructing GC correction and BAF likelihood steps and exporting
    TSV outputs
  repo: https://github.com/abyzovlab/CNVpytor
- name: igv.js
  role: Embeddable genome viewer supporting .pytor files for interactive visualization
    of CNVpytor results
  repo: https://github.com/igvteam/igv.js/wiki/cnvpytor
- name: JBrowse
  role: Export target for CNVpytor data; a lightweight CNVpytor plugin enables GUI-assisted
    analysis
provenance:
  source_task_ids:
  - task_005
  - task_003
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giab074
    title: 'CNVpytor: a tool for copy number variation detection and analysis from
      read depth and allele imbalance in whole-genome sequencing'
  source_content_hash: sha256:381305bc61f334a51cb90eaeb4f8c600f37aa3376a4b6854553bcb0d93b07dc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/abyzovlab/CNVpytor
  tools:
  - CNVnator
  - CNVpytor
  - JBrowse
  - NumPy
  - Python
  - SciPy
  - h5py
  - igv.js
  - matplotlib
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# copy-number-variation-analysis

## Summary

Detect and analyze copy number variations (CNVs/CNAs) from whole-genome sequencing by combining binned read-depth signal (mean-shift partitioning, GC correction) with B-allele frequency likelihoods from SNPs and small indels, as implemented in CNVpytor, a Python extension of CNVnator.

## When to use

Use when analyzing WGS alignment data (BAM) for CNV/CNA discovery and genotyping, especially when RD signal alone is insufficient and SNP/indel allele imbalance (BAF) evidence or copy-neutral LOH detection is needed.

## When NOT to use

- Input is already a called CNV segment/feature table rather than raw alignment (BAM) and variant (VCF) data — CNVpytor's RD binning and BAF likelihood steps operate on alignments and variant-level genotypes
- Analysis requires detecting small insertions/deletions or structural variants without copy-number change — the tool targets CNV/CNA from read depth and allele imbalance, not balanced SVs
- Somatic/cancer CNA calling requiring high sensitivity in impure samples — the article notes the somatic CNA caller is only a prototype and a more accurate approach for cancer and mosaic CNAs is under development

## Inputs

- Coordinate-sorted WGS alignment file (BAM/CRAM) from a diploid or cancer sample
- Variant call file (VCF) containing SNPs and single-letter indels with genotype/sample information
- Reference genome build selection: GRCh37 (hg19) or GRCh38 for the shipped GC/AT 100-bp-bin resource
- Optional 1000 Genomes Project strict-mask P-region filter applied via -mask_snps or -rd_use_mask

## Outputs

- .pytor root file (HDF5) storing RD histograms, GC-corrected signal, and BAF likelihood matrices with 100-bp-bin compression
- CNV/CNA call tables with copy number, e-val1, Q0, pN, dG metrics (TSV/XLS/VCF export)
- Per-bin BAF maximum-likelihood values and variant counts (bin sizes e.g. 10000 and 100000)
- Region genotypes (copy number as mean RD / mean autosomal RD scaled by 2)
- Plots (Manhattan, circular, region, RD/likelihood panels) and JBrowse/igv.js-compatible exports

## How to apply

Import the BAM into a .pytor HDF5 root file with 'cnvpytor -root file.pytor -rd file.bam', then compute histograms at bin sizes such as 1000, 10000, and 100000 followed by -partition and -call steps, which apply the CNVnator mean-shift algorithm, partitioning, and CNV calling. CNVpytor removes GC-correlated RD bias by joining RD bins with pre-calculated GC/AT content resource data (per 100-bp bin for GRCh37/hg19 and GRCh38) shipped with the package, equivalent to the rd_use_gc_corr viewer parameter which defaults to True — no reference FASTA parsing needed. For allele-based evidence, import SNPs and single-letter indels from a VCF ('-snp file.vcf -sample sample_name'), optionally refine with '-pileup file.bam' or filter variants with '-mask_snps' (1000 Genomes Project strict mask, which removes ~22% of variants and eliminates most outlier-BAF heterozygous SNPs), then compute per-bin BAF likelihood with '-baf 10000 100000' using a symmetrized beta distribution over the discretized [0, 0.5] interval (default 101 grid points), multiplying per-variant likelihoods within each bin. Filter called regions in view mode using the five parameters CNV size, e-val1, Q0 (e.g. 'set Q0_range 0 0.5'), pN, and dG (e.g. 'set p_range 0 0.00001', 'set size_range 100000 inf'), and optionally annotate calls via the Ensembl REST API or merge calls across samples with reciprocal overlap >50%. Genotype regions by dividing mean RD within the region by mean autosomal RD scaled by 2, optionally restricted to P-mask regions with '-rd_use_mask'. Intermediate RD signal is compressed by storing it in 100-bp bins within the HDF5 file, and results can be exported to JBrowse or analyzed via igv.js.

## Related tools

- **CNVpytor** (Primary tool: performs RD binning, GC correction via packaged GC/AT resource, mean-shift partitioning, CNV calling, BAF likelihood computation, genotyping, filtering, annotation, and visualization) — https://github.com/abyzovlab/CNVpytor
- **CNVnator** (Predecessor whose reimplemented core engine (RD binning, mean-shift algorithm, partitioning, calling) CNVpytor inherits; used as baseline for call-overlap comparison)
- **Python** (Implementation language and scripting environment (with h5py, numpy, scipy, matplotlib) for reconstructing GC correction and BAF likelihood steps and exporting TSV outputs) — https://github.com/abyzovlab/CNVpytor
- **igv.js** (Embeddable genome viewer supporting .pytor files for interactive visualization of CNVpytor results) — https://github.com/igvteam/igv.js/wiki/cnvpytor
- **JBrowse** (Export target for CNVpytor data; a lightweight CNVpytor plugin enables GUI-assisted analysis)

## Examples

```
cnvpytor -root file.pytor -rd file.bam; cnvpytor -root file.pytor -his 10000 100000; cnvpytor -root file.pytor -snp file.vcf -sample sample_name; cnvpytor -root file.pytor -baf 10000 100000; cnvpytor -root file.pytor -call 10000 100000
```

## Evaluation signals

- CNVpytor and CNVnator called regions should overlap by >99% on the same input data
- Applying -mask_snps should remove approximately 22% of variants (paper: 'This way we eliminate ∼22% of variants'), with retained heterozygous BAF values closely following the expected binomial distribution
- All maximum-likelihood BAF values must lie within the [0, 0.5] interval, and per-bin likelihoods derive from multiplying symmetrized-beta likelihoods of variants in each bin (101-point default grid)
- GC-corrected RD must show one corrected value per genomic bin using the shipped GRCh37/GRCh38 per-100-bp-bin GC/AT resource, matching rd_use_gc_corr=True behavior
- Genotyped copy numbers should be near-integer multiples around 2 for diploid regions (mean RD divided by mean autosomal RD scaled by 2)

## Limitations

- The somatic CNA caller is only a prototype; a more accurate approach for discovering cancer and mosaic CNAs is under development and not yet available
- Mask filtering with the 1000 Genomes strict mask removes ~22% of variants — a trade-off accepted to reduce false-positive heterozygous SNPs
- Only SNPs and single-letter indels are imported from VCF files; all other variant types are ignored
- Annotation via the Ensembl REST API is an optional step requiring web connection, and version 1.3.1 notes error logs for missing annotation links in reference genome settings

## Evidence

- [abstract] CNVpytor uses BAF likelihood information from SNPs and small indels as CNV/CNA evidence: "CNVpytor uses B-allele frequency likelihood information from single-nucleotide polymorphisms and small indels data as additional evidence for CNVs/CNAs and as primary information for copy"
- [discussion] RD pipeline steps inherited from CNVnator: "Calculations for RD binning, mean-shift algorithm, partitioning, and calling CNVs are explained in detail in the CNVnator article"
- [discussion] GC/AT resource shipped with the package for GRCh37/GRCh38: "For the 2 versions GRCh37 (hg19) and GRCh38 of the human reference genome, information about GC and AT content for each 100-bp bin is provided"
- [discussion] Symmetrized beta distribution for BAF likelihood: "we have to use the symmetrized beta distribution for likelihood"
- [discussion] Mask filtering removes ~22% of variants: "This way we eliminate ∼22% of variants, but there is a benefit because with fewer false-positive heterozygous SNPs the statistics is improved"
- [discussion] CNVpytor–CNVnator call concordance: "In spite of slightly different implementation, called regions by CNVnator and CNVpytor overlap by >99%."
- [readme] CLI commands for RD calling: "> cnvpytor -root file.pytor -rd file.bam
> cnvpytor -root file.pytor -his 1000 10000 100000"
- [methods] SNP import and optional filtering commands: "> cnvpytor -root file.pytor -snp file.vcf -sample sample_name"

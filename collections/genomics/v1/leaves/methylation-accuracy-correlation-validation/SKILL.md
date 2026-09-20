---
name: methylation-accuracy-correlation-validation
description: Use when a new methylation-calling or methylation-phasing output (bedmethyl,
  per-haplotype frequencies, DMR lists) must be benchmarked against BS-seq or nanopore
  ground truth to confirm accuracy.
when_to_use_negative:
- 'Validating non-CpG 5mC or other modifications (e.g. 6mA): the models only detect
  symmetric 5mCpG and ''cannot be applied for 5mC detection in non-CpGs or the detection
  of other DNA modifications (such as 6mA)'''
- 'Benchmarking hemimethylated CpG detection: the methods ''are incapable of detecting
  hemimethylated CpGs'''
- Input is a per-read methylation call file not yet aggregated into site-level or
  haplotype-level frequencies — aggregate into bedmethyl first before correlation
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3295
tools:
- name: ccsmethphase
  role: Produces the haplotype-resolved bedmethyl files and DML/DMR outputs that are
    validated against BS-seq/nanopore benchmarks
  repo: https://github.com/PengNi/ccsmethphase
- name: ccsmeth
  role: Generates per-read 5mCpG calls whose read-level accuracy and site-level correlations
    are benchmarked
  repo: https://github.com/PengNi/ccsmeth
- name: DSS
  role: Differential methylation analysis used to call CCS DMLs and DMRs for concordance
    comparison
- name: NanoMethPhase
  role: Source of the DSS-based ASM detection code referenced by ccsmethphase
  repo: https://github.com/vahidAK/NanoMethPhase
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:e46ea9ad76701a0917233746bb18db4e4e81dcd6499867b1a5b478ba2c555f28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmethphase
  tools:
  - DSS
  - NanoMethPhase
  - Singularity
  - ccsmeth
  - ccsmethphase
  - clair3
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# methylation-accuracy-correlation-validation

## Summary

Validates haplotype-aware methylation and allele-specific methylation (ASM) calls from PacBio CCS-based pipelines by correlating haplotype methylation frequencies against orthogonal benchmarks (BS-seq, nanopore) and checking DMR concordance, using Pearson correlation and proximity-based overlap metrics.

## When to use

Use when a new methylation-calling or methylation-phasing output (bedmethyl, per-haplotype frequencies, DMR lists) must be benchmarked against BS-seq or nanopore ground truth to confirm accuracy.

## When NOT to use

- Validating non-CpG 5mC or other modifications (e.g. 6mA): the models only detect symmetric 5mCpG and 'cannot be applied for 5mC detection in non-CpGs or the detection of other DNA modifications (such as 6mA)'
- Benchmarking hemimethylated CpG detection: the methods 'are incapable of detecting hemimethylated CpGs'
- Input is a per-read methylation call file not yet aggregated into site-level or haplotype-level frequencies — aggregate into bedmethyl first before correlation

## Inputs

- Per-haplotype bedmethyl files (hp1 and hp2) from the mods_freq directory
- CCS-derived DMR tables from DSS differential methylation analysis
- HG002 BS-seq methylation bed file (e.g. hg002_bsseq_chr20_demo.bed)
- Nanopore-based methylation calls and DMR lists
- ccsmethphase_results directory (phased modBAM, clair3 and whatshap VCFs, DML/DMR tables)

## Outputs

- Pearson correlation coefficients of haplotype-level methylation frequencies vs BS-seq and nanopore benchmarks
- DMR concordance statistics (fraction of benchmark DMRs within 10 kb of CCS DMRs)
- Read-level accuracy and AUC metrics for 5mCpG detection
- Pass/fail validation against the ~0.93 correlation threshold and completeness of ccsmethphase_results outputs

## How to apply

After ccsmethphase produces per-haplotype bedmethyl files (hp1/hp2) in the mods_freq directory, run the DSS-based differential methylation analysis to call DMLs and DMRs, then compare against HG002 BS-seq bed files and nanopore calls with Pearson correlation on site-level methylation frequencies. For ASM concordance, check what fraction of benchmark DMRs lie within 10 kb of CCS-generated DMRs (the paper reports 81.4% of BS-seq DMRs within 10 kb of the 14,390 CCS DMRs, versus 2,463 BS-seq and 16,250 nanopore DMRs). Consider validating at multiple coverages, since ccsmeth achieves >0.90 site-level correlations with only 10x reads and 0.90 read-level accuracy / 0.97 AUC on >=10 Kb CCS reads. Validation passes when haplotype-level correlations against BS-seq/nanopore exceed ~0.93 and expected outputs (phased modBAM, hp1/hp2 bedmethyl beds, clair3/whatshap VCFs, DML/DMR tables) all exist under ccsmethphase_results.

## Related tools

- **ccsmethphase** (Produces the haplotype-resolved bedmethyl files and DML/DMR outputs that are validated against BS-seq/nanopore benchmarks) — https://github.com/PengNi/ccsmethphase
- **ccsmeth** (Generates per-read 5mCpG calls whose read-level accuracy and site-level correlations are benchmarked) — https://github.com/PengNi/ccsmeth
- **DSS** (Differential methylation analysis used to call CCS DMLs and DMRs for concordance comparison)
- **NanoMethPhase** (Source of the DSS-based ASM detection code referenced by ccsmethphase) — https://github.com/vahidAK/NanoMethPhase

## Examples

```
nextflow run main.nf --dsname hg002_asm --input input_sheet.tsv --genome ref.fa --include_all_ctgs true -profile singularity
```

## Evaluation signals

- Pearson correlation of haplotype-level methylation frequencies against BS-seq and nanopore exceeds ~0.93
- 81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs (14,390 CCS DMRs vs 2,463 BS-seq and 16,250 nanopore DMRs)
- Site-level correlations with BS-seq and nanopore stay >0.90 even using only 10x coverage
- Read-level 5mCpG detection reaches 0.90 accuracy and 0.97 AUC on >=10 Kb CCS reads
- All expected outputs exist under ccsmethphase_results (phased modBAM, hp1/hp2 bedmethyl beds, clair3 and whatshap VCFs, DML/DMR tables)

## Limitations

- Models cannot detect hemimethylated CpGs
- Cannot be applied to non-CpG 5mC detection or other DNA modifications such as 6mA
- Strand-specific-methylation model has significantly reduced performance due to limited subread depth (0.85 read-level accuracy on HG002 15 Kb dataset)
- Whether the site-level model design applies to non-CpG 5mCs and other modifications has not been verified

## Evidence

- [abstract] Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution.: "Using long (≥10 Kb) CCS reads, ccsmeth achieves 0.90 accuracy and 0.97 Area Under the Curve on 5mCpG detection at single-molecule resolution."
- [abstract] ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [methods] mods_freq: bedmethyl files, site-level methylation frequencies: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [discussion] Hence, these methods are incapable of detecting hemimethylated CpGs: "Hence, these methods are incapable of detecting hemimethylated CpGs"
- [other] 81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs: [evidence span withheld — anchor status: not_found]

---
name: differential-methylated-region-detection
description: Use when you have haplotype-resolved (hp1/hp2) site-level methylation
  frequencies from phased long reads and need to call DMRs for allele-specific methylation,
  or to benchmark against BS-seq/nanopore ASM calls.
when_to_use_negative:
- The reads are not phased or only a single combined (all.bed) methylation track exists
  — no haplotype contrast means ASM/DMR calling cannot be performed
- You need detection of hemimethylated CpGs or non-CpG 5mC or other modifications
  such as 6mA — the underlying methods are incapable of these cases
- Input is raw unaligned HiFi BAMs — methylation calling (ccsmeth), alignment (pbmm2),
  SNV calling (clair3) and phasing (whatshap) must run first
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: ccsmethphase
  role: Nextflow pipeline orchestrating the haplotype methylation frequency aggregation
    and DSS-based DMR calling
  repo: https://github.com/PengNi/ccsmethphase
- name: DSS
  role: Statistical differential methylation analysis used to call DMLs and DMRs between
    haplotypes; code referenced from NanoMethPhase
  repo: https://github.com/vahidAK/NanoMethPhase
- name: whatshap
  role: Phases PASS SNVs and reads to produce the haplotype tags that stratify methylation
    frequencies into hp1/hp2 before DMR calling
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:4b4a71f324328e0d9d62c53ff41ea2f2c044b9081790785b00ca0c897d519add
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmethphase
  tools:
  - DSS
  - Singularity
  - ccsmethphase
  - clair3
  - pbmm2
  - whatshap
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# differential-methylated-region-detection

## Summary

Detect differentially methylated loci (DMLs) and regions (DMRs) between haplotypes for allele-specific methylation (ASM) using a DSS-based analysis on haplotype-resolved bedmethyl frequencies from PacBio CCS reads, as implemented in the ccsmethphase Nextflow pipeline.

## When to use

Use when you have haplotype-resolved (hp1/hp2) site-level methylation frequencies from phased long reads and need to call DMRs for allele-specific methylation, or to benchmark against BS-seq/nanopore ASM calls.

## When NOT to use

- The reads are not phased or only a single combined (all.bed) methylation track exists — no haplotype contrast means ASM/DMR calling cannot be performed
- You need detection of hemimethylated CpGs or non-CpG 5mC or other modifications such as 6mA — the underlying methods are incapable of these cases
- Input is raw unaligned HiFi BAMs — methylation calling (ccsmeth), alignment (pbmm2), SNV calling (clair3) and phasing (whatshap) must run first

## Inputs

- Haplotype-specific bedmethyl files (hp1.bed, hp2.bed, all.bed) with site-level methylation frequencies from the ccsmethphase mods_freq directory
- Phased haplotype-tagged modBAMs and whatshap-phased SNV VCFs (context for the haplotags)
- Optional: BS-seq methylation bed file and nanopore methylation calls as benchmarks

## Outputs

- DML tables and DMR bed files, e.g. G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.freq.aggregate.hp_callDMR.autosomes_cf0.2.bed
- diff_methyl output directory under ccsmethphase_results/<dsname>

## How to apply

After ccsmethphase phasing, take the aggregated haplotype-specific bedmethyl files (hp1.bed, hp2.bed) produced by aggregating per-haplotype methylation frequencies under mods_freq. Run the DSS-based differential methylation analysis — code referenced from NanoMethPhase — with parameters calling DMLs and DMRs at p < 0.001 with |diff| > 0.2, filtering to autosomes only with the cf threshold set to 0.2. For validation on HG002, compare haplotype methylation frequencies against HG002 BS-seq bed files and nanopore calls using Pearson correlation, and check overlap of CCS-derived DMRs with benchmark DMRs. In the article's benchmark, 71.0x CCS reads yielded 14,390 DMRs versus 2,463 from BS-seq and 16,250 from nanopore, with 81.4% of BS-seq DMRs within 10 kb of CCS DMRs.

## Related tools

- **ccsmethphase** (Nextflow pipeline orchestrating the haplotype methylation frequency aggregation and DSS-based DMR calling) — https://github.com/PengNi/ccsmethphase
- **DSS** (Statistical differential methylation analysis used to call DMLs and DMRs between haplotypes; code referenced from NanoMethPhase) — https://github.com/vahidAK/NanoMethPhase
- **whatshap** (Phases PASS SNVs and reads to produce the haplotype tags that stratify methylation frequencies into hp1/hp2 before DMR calling)

## Examples

```
nextflow run main.nf --dsname <job> --input input_sheet.tsv --genome <ref.fa> --include_all_ctgs true -profile singularity
```

## Evaluation signals

- All expected outputs exist under ccsmethphase_results including hp1/hp2 bedmethyl beds and the DML/DMR tables with the documented naming pattern ending 'hp_callDMR.autosomes_cf0.2.bed'
- Haplotype-level methylation Pearson correlation against HG002 BS-seq and nanopore benchmarks exceeds ~0.93
- DMR counts are in the expected range relative to benchmarks (e.g. 14,390 CCS DMRs on 71.0x HG002 vs 2,463 BS-seq and 16,250 nanopore)
- Overlap check: ~81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs

## Limitations

- Methods consider only symmetric methylation and combine features from both strands, so they are incapable of detecting hemimethylated CpGs
- Cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA
- DMRs detected by PacBio CCS for assisting haplotyping and genome assembly remain to be studied

## Evidence

- [results] Running ccsmethphase on the total 71.0x HG002 CCS reads generates 14,390 DMRs, versus 2,463 DMRs from BS-seq and 16,250 from nanopore reads; 81.4% of BS-seq DMRs lie within 10 kb of CCS-generated DMRs: [evidence span withheld — anchor status: not_found]
- [full_text] Perform DSS-based differential methylation analysis to call CCS DMLs and DMRs, and compare haplotype methylation frequencies against the HG002 BS-seq bed file and nanopore calls using Pearson correlation: [evidence span withheld — anchor status: not_found]
- [methods] Code for ASM detection using DSS were referenced from NanoMethPhase (https://github.com/vahidAK/NanoMethPhase): "Code for ASM detection using DSS were referenced from NanoMethPhase (https://github.com/vahidAK/NanoMethPhase)"
- [discussion] Hence, these methods are incapable of detecting hemimethylated CpGs: "Hence, these methods are incapable of detecting hemimethylated CpGs"

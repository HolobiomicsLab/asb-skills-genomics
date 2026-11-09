---
name: coverage-subsampling-of-ccs-reads
description: Use when benchmarking site-level methylation callers on PacBio CCS/HiFi
  BAMs at matched low coverage, or when preparing 10x-subampled HG002 15/20/24 Kb,
  SD0651_P1, and CHM13 libraries for correlation analysis.
when_to_use_negative:
- The goal is read-level 5mCpG detection accuracy (e.g., the 0.90 accuracy / 0.97
  AUC read-level benchmark), not site-level methylation frequency comparison across
  coverage conditions.
- The input data are not PacBio CCS/HiFi reads (e.g., nanopore or short-read BAMs);
  subsampling to 10x is meaningful only for the CCS-based ccsmeth workflow.
- The modification of interest is not symmetric 5mCpG methylation — the workflow cannot
  address hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA.
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0102
tools:
- name: ccsmeth
  role: count-mode feature extraction (IPD/PW) and read-level 5mCpG probability calling
    on both 10x-subsampled and full-depth CCS BAMs
  repo: https://github.com/PengNi/ccsmeth
- name: ccsmethphase
  role: Nextflow pipeline whose mods_freq step aggregates aligned modbam calls into
    site-level methylation-frequency bedmethyl files for each coverage condition
  repo: https://github.com/PengNi/ccsmethphase
- name: pbmm2
  role: aligns ccsmeth modbam files to the hg38 reference before site-level aggregation
  repo: https://github.com/PacificBiosciences/pbmm2
- name: ccs
  role: upstream tool that generates the PacBio HiFi (CCS) read BAMs that are subsequently
    subsampled to 10x
  repo: https://github.com/PacificBiosciences/ccs
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:7f33578576d2cbf8d189efabb86dff34a84264630f700e2dd3e4a187d8aded2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - ccs
  - ccsmeth
  - ccsmethphase
  - pbmm2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# coverage-subsampling-of-ccs-reads

## Summary

Subsample PacBio HiFi/CCS read BAMs to a fixed depth (e.g., 10x) alongside full-depth CCS data (HG002 combined 71.0x) so that ccsmeth site-level 5mCpG methylation calls can be benchmarked against BS-seq and nanopore references as a function of coverage.

## When to use

Use when benchmarking site-level methylation callers on PacBio CCS/HiFi BAMs at matched low coverage, or when preparing 10x-subampled HG002 15/20/24 Kb, SD0651_P1, and CHM13 libraries for correlation analysis.

## When NOT to use

- The goal is read-level 5mCpG detection accuracy (e.g., the 0.90 accuracy / 0.97 AUC read-level benchmark), not site-level methylation frequency comparison across coverage conditions.
- The input data are not PacBio CCS/HiFi reads (e.g., nanopore or short-read BAMs); subsampling to 10x is meaningful only for the CCS-based ccsmeth workflow.
- The modification of interest is not symmetric 5mCpG methylation — the workflow cannot address hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA.

## Inputs

- PacBio CCS/HiFi read BAM files for HG002 15/20/24 Kb libraries, SD0651_P1 15 Kb library, and CHM13 library
- Full-depth (total) HG002 CCS reads (combined 71.0x)
- hg38 reference genome provided via an input sheet tsv
- Reference methylation calls: HG002 and SD0651_P1 BS-seq methylation calls, CHM13 nanopore methylation calls

## Outputs

- 10x-subsampled CCS BAM per library and paired full-depth BAM condition
- Site-level methylation-frequency bedmethyl files per coverage condition (mods_freq output)
- Pearson correlation coefficients per dataset and coverage condition recorded in ccsmeth_site_level_correlations.csv (e.g., 0.9198/0.9083/0.9087 at 10x vs BS-seq; 0.9463/0.9271/0.9410 at total depth)

## How to apply

Take the CCS read BAM files (HG002 15/20/24 Kb libraries, SD0651_P1 15 Kb library, CHM13 library) and create two coverage conditions per dataset: a subsampled set at 10x depth and the total/full-depth set (HG002 total CCS reads combined to 71.0x). Feed each coverage condition through ccsmeth in count mode to extract per-read CpG kinetics features (IPD and PW), run the read-level model to obtain 5mCpG probabilities, convert to modbam files, align with pbmm2, and aggregate to site-level methylation-frequency bedmethyl files via the ccsmethphase mods_freq step. The rationale is that the paper claims >0.90 site-level correlations with BS-seq and nanopore sequencing at only 10x reads, so low-coverage subsampling is the strictest test; intersect methylation frequencies with BS-seq calls (HG002, SD0651_P1) or nanopore calls (CHM13) on matched CpG sites and compute Pearson correlations per coverage condition. Treat 10x and total-depth results as paired conditions for the same library so performance loss from reduced coverage is directly attributable to read depth rather than dataset differences.

## Related tools

- **ccsmeth** (count-mode feature extraction (IPD/PW) and read-level 5mCpG probability calling on both 10x-subsampled and full-depth CCS BAMs) — https://github.com/PengNi/ccsmeth
- **ccsmethphase** (Nextflow pipeline whose mods_freq step aggregates aligned modbam calls into site-level methylation-frequency bedmethyl files for each coverage condition) — https://github.com/PengNi/ccsmethphase
- **pbmm2** (aligns ccsmeth modbam files to the hg38 reference before site-level aggregation) — https://github.com/PacificBiosciences/pbmm2
- **ccs** (upstream tool that generates the PacBio HiFi (CCS) read BAMs that are subsequently subsampled to 10x) — https://github.com/PacificBiosciences/ccs

## Evaluation signals

- Pearson correlations at 10x coverage against BS-seq reach ~0.90 for HG002 15/20/24 Kb (0.9198/0.9083/0.9087) and 0.8750 for SD0651_P1.
- Total-depth (71.0x combined) HG002 correlations with BS-seq match 0.9463/0.9271/0.9410 for 15/20/24 Kb libraries, exceeding the 10x values as expected with increased coverage.
- CHM13 benchmarked against nanopore methylation calls yields a Pearson correlation of 0.9328.
- Every coverage condition produces a per-dataset row in ccsmeth_site_level_correlations.csv so subsampled and total results are directly comparable.
- Read-level calls converted to modbam, aligned with pbmm2, and aggregated via mods_freq yield bedmethyl files covering matched CpG sites for intersection with reference calls.

## Limitations

- Methods consider only symmetric methylation and combine features from both DNA strands, so they are incapable of detecting hemimethylated CpGs.
- The workflow cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA.
- Strand-specific-methylation model performance is significantly reduced due to limited subread depth, an effect that subsampling to low coverage will exacerbate.

## Evidence

- [abstract] ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [full_text] Repeat steps 2-3 on the full-depth (total) HG002 CCS reads (combined 71.0x) to obtain total-coverage methylation frequencies.: [evidence span withheld — anchor status: not_found]
- [full_text] Validation thresholds require >0.90 correlations at 10x coverage against BS-seq/nanopore references.: [evidence span withheld — anchor status: not_found]
- [discussion] ccsmeth achieves better site-level prediction than existing methods, especially under low coverages.: "ccsmeth achieves better performance on site-level prediction, especially under low coverages."
- [readme] README confirms ccsmeth operates on PacBio CCS input data.: [evidence span withheld — anchor status: external_doc]

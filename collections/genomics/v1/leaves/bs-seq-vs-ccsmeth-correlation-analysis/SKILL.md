---
name: bs-seq-vs-ccsmeth-correlation-analysis
description: Use when validating per-site methylation frequencies produced from PacBio
  CCS/HiFi reads against an orthogonal gold standard (BS-seq bedmethyl or nanopore
  methylation calls) by intersecting matched CpGs and computing Pearson correlation
  coefficients.
when_to_use_negative:
- The modification of interest is not symmetric CpG methylation — the approach cannot
  detect hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA
- No orthogonal ground-truth methylation calls (BS-seq or nanopore) are available
  for the matched CpG sites, so correlations cannot be computed
- Reads are not PacBio CCS/HiFi — ccsmeth relies on SMRT kinetics features (IPD and
  PW) specific to CCS reads
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: ccsmeth
  role: Deep-learning detection of 5mCpG from CCS read kinetics features (IPD, PW)
    in count mode, producing read-level probabilities converted to modbam files
  repo: https://github.com/PengNi/ccsmeth
- name: ccsmethphase
  role: Nextflow pipeline whose mods_freq step aggregates read-level calls into site-level
    methylation-frequency bedmethyl files
  repo: https://github.com/PengNi/ccsmethphase
- name: pbmm2
  role: Aligns ccsmeth modbam files to the reference genome before methylation frequency
    aggregation
  repo: https://github.com/PacificBiosciences/pbmm2
- name: Nextflow
  role: Workflow engine executing the ccsmethphase pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:aa5a920cd6ea17a0d2fbed4079a087d20126333bc9ac55892c61cea0de8fd1f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - Nextflow
  - ccsmeth
  - ccsmethphase
  - pbmm2
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# bs-seq-vs-ccsmeth-correlation-analysis

## Summary

Benchmark site-level 5mCpG methylation frequencies predicted by ccsmeth from PacBio CCS reads by computing Pearson correlations against BS-seq and nanopore methylation calls on matched CpG sites, validating that ccsmeth reaches >0.90 correlations even at 10x coverage.

## When to use

Use when validating per-site methylation frequencies produced from PacBio CCS/HiFi reads against an orthogonal gold standard (BS-seq bedmethyl or nanopore methylation calls) by intersecting matched CpGs and computing Pearson correlation coefficients.

## When NOT to use

- The modification of interest is not symmetric CpG methylation — the approach cannot detect hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA
- No orthogonal ground-truth methylation calls (BS-seq or nanopore) are available for the matched CpG sites, so correlations cannot be computed
- Reads are not PacBio CCS/HiFi — ccsmeth relies on SMRT kinetics features (IPD and PW) specific to CCS reads

## Inputs

- CCS read BAM files (HG002 15/20/24 Kb libraries subsampled to 10x depth, SD0651_P1 15 Kb library, CHM13 library)
- Total-depth HG002 CCS reads (combined 71.0x)
- hg38 reference genome provided via an input sheet tsv for ccsmethphase
- HG002 and SD0651_P1 BS-seq methylation calls (bedmethyl)
- CHM13 nanopore methylation calls

## Outputs

- Site-level methylation-frequency bedmethyl files from ccsmethphase mods_freq
- Pearson correlation coefficients per dataset and coverage condition (ccsmeth_site_level_correlations.csv)

## How to apply

Run ccsmeth in count mode to extract per-read CpG kinetics features (IPD and PW), apply the ccsmeth read-level model to call 5mCpG probabilities, and convert outputs to modified-bam (modbam) files. Align modbam files to hg38 with pbmm2, then aggregate read-level calls into site-level methylation-frequency bedmethyl files with the ccsmethphase mods_freq step, processing both subsampled 10x and full-depth (total, e.g. combined 71.0x) HG002 CCS reads. Intersect the ccsmeth CpG methylation frequencies with BS-seq methylation calls (HG002, SD0651_P1) or nanopore methylation calls (CHM13) on matched CpG sites and compute Pearson correlation coefficients per dataset and per coverage condition. Accept the reproduction if correlations exceed 0.90 at 10x against BS-seq/nanopore (expected: 0.9198/0.9083/0.9087 for 10x HG002 15/20/24 Kb vs BS-seq, 0.9463/0.9271/0.9410 for total HG002 vs BS-seq, 0.8750 for SD0651_P1 vs BS-seq, 0.9328 for CHM13 vs nanopore), recorded in ccsmeth_site_level_correlations.csv.

## Related tools

- **ccsmeth** (Deep-learning detection of 5mCpG from CCS read kinetics features (IPD, PW) in count mode, producing read-level probabilities converted to modbam files) — https://github.com/PengNi/ccsmeth
- **ccsmethphase** (Nextflow pipeline whose mods_freq step aggregates read-level calls into site-level methylation-frequency bedmethyl files) — https://github.com/PengNi/ccsmethphase
- **pbmm2** (Aligns ccsmeth modbam files to the reference genome before methylation frequency aggregation) — https://github.com/PacificBiosciences/pbmm2
- **Nextflow** (Workflow engine executing the ccsmethphase pipeline)

## Examples

```
nextflow run PengNi/ccsmethphase --input input_sheet.tsv --mods_freq true; python scripts/compute_correlations.py --ccsmeth_ccpg ccsmeth.bedmethyl --truth hg002_bsseq.bed --out ccsmeth_site_level_correlations.csv
```

## Evaluation signals

- Pearson correlations at 10x coverage vs BS-seq should reach ~0.92/0.91/0.91 for HG002 15/20/24 Kb (reported 0.9198/0.9083/0.9087)
- Total-depth HG002 correlations vs BS-seq should reach 0.9463, 0.9271, and 0.9410 for 15 Kb, 20 Kb, and 24 Kb datasets
- SD0651_P1 vs BS-seq correlation should be ~0.8750 and CHM13 vs nanopore ~0.9328
- All correlation values >0.90 at 10x coverage against BS-seq/nanopore, recorded in ccsmeth_site_level_correlations.csv
- mods_freq step must emit valid bedmethyl files of site-level methylation frequencies

## Limitations

- Methods cannot detect hemimethylated CpGs because only symmetric methylation is considered and features from both DNA strands are combined
- The design cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA; applicability of the site-level model to non-CpG 5mCs is unverified
- Correlations depend on matched CpG-site intersection with ground truth (BS-seq or nanopore), so coverage and CpG overlap affect comparability

## Evidence

- [abstract] At the genome-wide site level, ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10x reads: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [methods] mods_freq produces bedmethyl site-level methylation frequencies: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [intro] ccsmeth uses kinetics features from CCS reads: "we propose ccsmeth, a deep-learning method to detect DNA 5mCpGs by using kinetics features (IPDs and PWs) of PacBio CCS reads"
- [methods] pbmm2 used for aligning modbam outputs: "G1.HG002_demo.hifi.ccsmeth.modbam.pbmm2.merged_size1.SNV_PASS_whatshap.bam"
- [readme] pb-CpG-tools README describes generating site methylation probabilities from mapped HiFi reads: [evidence span withheld — anchor status: external_doc]

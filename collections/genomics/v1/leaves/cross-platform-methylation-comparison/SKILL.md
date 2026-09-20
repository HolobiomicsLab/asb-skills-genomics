---
name: cross-platform-methylation-comparison
description: Use when validating PacBio CCS 5mCpG calls against bisulfite or nanopore
  ground truth — e.g. HG002/SD0651_P1/CHM13 WGS with matching BS-seq or nanopore methylation
  calls available.
when_to_use_negative:
- The target modification is not 5mCpG — the method cannot detect hemimethylated CpGs,
  non-CpG 5mCs, or other modifications such as 6mA
- The input is read-level accuracy assessment rather than site-level methylation frequency
  comparison
- No matched BS-seq or nanopore methylation calls exist for the sample, so no orthogonal
  benchmark can be intersected
edam_operation: http://edamontology.org/operation_3204
edam_topics:
- http://edamontology.org/topic_3674
- http://edamontology.org/topic_0654
tools:
- name: ccsmeth
  role: Detects 5mCpGs from PacBio CCS read kinetics (IPD/PW) via deep-learning read-level
    and site-level models
  repo: https://github.com/PengNi/ccsmeth
- name: ccsmethphase
  role: Nextflow pipeline step (mods_freq) that aggregates read-level calls into site-level
    methylation-frequency bedmethyl files
  repo: https://github.com/PengNi/ccsmethphase
- name: pbmm2
  role: Aligns ccsmeth modbam files to the reference genome before site-level aggregation
  repo: https://github.com/PacificBiosciences/pbmm2
- name: Nextflow
  role: Workflow engine running the ccsmethphase methylation-phasing/detection pipeline
  repo: https://github.com/PengNi/ccsmethphase
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-39784-9
    title: DNA 5-methylcytosine detection and methylation phasing using PacBio circular
      consensus sequencing
  source_content_hash: sha256:e318e346a6c5d304f38966cb7619a5b8bfe947cdc9f4c7631c753dbb3edb0a32
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PengNi/ccsmeth
  tools:
  - Nextflow
  - ccsmeth
  - ccsmethphase
  - pbmm2
  edam_operation: http://edamontology.org/operation_3204
  edam_topics:
  - http://edamontology.org/topic_3674
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# Cross-platform methylation comparison

## Summary

Benchmark ccsmeth-derived site-level CpG methylation frequencies from PacBio HiFi/CCS reads against orthogonal BS-seq and nanopore methylation calls by intersecting matched CpG sites and computing Pearson correlation coefficients per dataset and coverage condition.

## When to use

Use when validating PacBio CCS 5mCpG calls against bisulfite or nanopore ground truth — e.g. HG002/SD0651_P1/CHM13 WGS with matching BS-seq or nanopore methylation calls available.

## When NOT to use

- The target modification is not 5mCpG — the method cannot detect hemimethylated CpGs, non-CpG 5mCs, or other modifications such as 6mA
- The input is read-level accuracy assessment rather than site-level methylation frequency comparison
- No matched BS-seq or nanopore methylation calls exist for the sample, so no orthogonal benchmark can be intersected

## Inputs

- PacBio CCS/HiFi BAM files (e.g. HG002 15/20/24 Kb libraries subsampled to 10x, SD0651_P1 15 Kb library, CHM13 library, total-coverage HG002 at 71.0x)
- hg38 reference genome
- HG002 and SD0651_P1 BS-seq methylation calls
- CHM13 nanopore methylation calls

## Outputs

- Site-level CpG methylation-frequency bedmethyl files
- modbam files of per-read 5mCpG calls
- Pearson correlation coefficients vs BS-seq and nanopore per dataset and coverage condition (recorded e.g. in ccsmeth_site_level_correlations.csv)

## How to apply

Run ccsmeth in count mode to extract per-read CpG IPD/PW kinetics features, apply the ccsmeth read-level model to call 5mCpG probabilities, and convert outputs to modbam files. Align modbams to hg38 with pbmm2, then aggregate read-level calls into site-level methylation-frequency bedmethyl files using the ccsmethphase mods_freq step. To test coverage robustness, repeat on reads subsampled to 10x before running total-coverage (e.g. combined 71.0x HG002) calls. Intersect the CpG methylation frequencies with matched BS-seq (HG002, SD0651_P1) or nanopore (CHM13) calls on common CpG sites and compute Pearson correlations per dataset and per coverage condition. Accept the benchmark if correlations exceed 0.90 at 10x coverage against BS-seq/nanopore, matching the reported values.

## Related tools

- **ccsmeth** (Detects 5mCpGs from PacBio CCS read kinetics (IPD/PW) via deep-learning read-level and site-level models) — https://github.com/PengNi/ccsmeth
- **ccsmethphase** (Nextflow pipeline step (mods_freq) that aggregates read-level calls into site-level methylation-frequency bedmethyl files) — https://github.com/PengNi/ccsmethphase
- **pbmm2** (Aligns ccsmeth modbam files to the reference genome before site-level aggregation) — https://github.com/PacificBiosciences/pbmm2
- **Nextflow** (Workflow engine running the ccsmethphase methylation-phasing/detection pipeline) — https://github.com/PengNi/ccsmethphase

## Examples

```
nextflow run PengNi/ccsmethphase --input sheet.tsv --ref hg38.fa --mods_freq true  # then intersect bedmethyl CpG frequencies with BS-seq/nanopore calls and compute Pearson correlations
```

## Evaluation signals

- Pearson correlation vs BS-seq exceeds 0.90 at 10x coverage: 0.9198/0.9083/0.9087 for HG002 15/20/24 Kb, and 0.9463/0.9271/0.9410 with total CCS reads
- Total-coverage HG002 correlations with nanopore sequencing reach 0.9287, 0.9127, and 0.9240 (15/20/24 Kb)
- SD0651_P1 correlation with BS-seq reaches ~0.8750 and CHM13 correlation with nanopore ~0.9328
- mods_freq step produces well-formed bedmethyl site-level methylation-frequency files after pbmm2 alignment
- Correlations recorded per dataset and coverage condition in ccsmeth_site_level_correlations.csv match reported values

## Limitations

- Methods combining both DNA strands for prediction cannot detect hemimethylated CpGs
- The approach cannot be applied to 5mC detection in non-CpGs or other DNA modifications such as 6mA
- Strand-specific-methylation model performance is significantly reduced due to limited subread depth
- Whether the site-level model design applies to non-CpG 5mCs and other modifications has not been verified

## Evidence

- [abstract] ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads: "ccsmeth achieves >0.90 correlations with bisulfite sequencing and nanopore sequencing using only 10× reads"
- [methods] mods_freq produces bedmethyl site-level methylation frequencies: "mods_freq: bedmethyl files, site-level methylation frequencies"
- [methods] ccsmethphase takes PacBio reads and a reference genome as input: "ccsmethphase takes files of PacBio reads (subreads.bam or hifi.bam), and a reference genome as input."
- [discussion] Methods cannot detect hemimethylated CpGs: "Hence, these methods are incapable of detecting hemimethylated CpGs"
- [readme] ccsmethphase is a Nextflow pipeline for haplotype-aware methylation with CCS reads: "Methylation phasing using PacBio CCS reads"

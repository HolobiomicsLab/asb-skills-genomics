---
name: pacbio-kinetic-read-filtering
description: Use when input is a raw PacBio CCS BAM with kinetic tags (fi, ri, fp,
  rp) that must be quality-filtered before Z-Calling base calling, SVM read classification,
  or metagenome dZ-source detection.
when_to_use_negative:
- Input reads lack the kinetic tags (fi, ri, fp, rp) — e.g. CCS BAMs generated without
  the '--keep-kinetics' option — since filtering and all Z-Calling models depend on
  PW/IPD signals
- Input is short-read (Illumina/ONT) data rather than PacBio CCS HiFi reads, where
  PW/IPD kinetics do not exist
- The goal is site-specific A/Z calling on already-filtered BAMs — re-filtering could
  discard reads needed for per-site Z-frequency aggregation
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Z-Calling
  role: Provides py/filter_bam.py, the script that applies the kinetic-tag, Median_PW/Median_IPD,
    and pass-count filters to produce clean BAMs for base calling and read classification
  repo: https://github.com/xiaochuanle/Z-Calling
- name: PacBio Circular Consensus Sequencing (CCS)
  role: Generates the input BAMs with --keep-kinetics so PW and IPD signals are available
    for filtering
- name: pbmm2
  role: Optionally aligns the filtered reads to a reference (--preset CCS --sort)
    when the input BAM was unmapped
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_008
  source_papers:
  - doi: 10.1038/s42003-026-09849-8
    title: 'Z-Calling: a tool for A/Z (2,6-diaminopurine) base calling and dZ-DNA
      detection using PacBio HiFi reads'
  source_content_hash: sha256:1088ae0ee53f829cbcd3c45b49a66615a20e96d9d21d6d4a7c9ac9cb38813728
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xiaochuanle/Z-Calling
  repo_url: https://github.com/xiaochuanle/Z-Calling
  tools:
  - PacBio CCS
  - PacBio Circular Consensus Sequencing (CCS)
  - Z-Calling
  - minimap2
  - pbmm2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# pacbio-kinetic-read-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-processing step of the Z-Calling pipeline that removes PacBio CCS reads lacking kinetic tags, poor Median_PW/Median_IPD signal quality, or insufficient polymerase passes, so that downstream dZ-DNA base calling and read classification operate on reliable kinetic signals.

## When to use

Use when input is a raw PacBio CCS BAM with kinetic tags (fi, ri, fp, rp) that must be quality-filtered before Z-Calling base calling, SVM read classification, or metagenome dZ-source detection.

## When NOT to use

- Input reads lack the kinetic tags (fi, ri, fp, rp) — e.g. CCS BAMs generated without the '--keep-kinetics' option — since filtering and all Z-Calling models depend on PW/IPD signals
- Input is short-read (Illumina/ONT) data rather than PacBio CCS HiFi reads, where PW/IPD kinetics do not exist
- The goal is site-specific A/Z calling on already-filtered BAMs — re-filtering could discard reads needed for per-site Z-frequency aggregation

## Inputs

- Raw PacBio CCS BAM file (mapped or unmapped) containing kinetic tags fi, ri, fp, rp
- Optional reference FASTA for downstream pbmm2 alignment of filtered reads

## Outputs

- Filtered BAM file retaining only reads passing kinetic-tag, Median_PW/Median_IPD, and pass-count criteria
- Optionally, a CCS-aligned mapping BAM (pbmm2 --preset CCS --sort) for downstream z-calling

## How to apply

Run py/filter_bam.py on each raw BAM, keeping only reads whose kinetic tags (fi, ri, fp, rp) are present, whose Median_PW/Median_IPD ratio is neither below 0.3 nor above the 99th percentile of the distribution, and which have at least 3 passes on both strands (fn >= 3, rn >= 3). Rationale: Z incorporation perturbs pulse width (PW) relative to inter-pulse duration (IPD), so reads lacking these tags or with outlier kinetic ratios would corrupt k-mer MLP scoring. If the input BAM is unmapped, align the filtered reads with pbmm2 (--preset CCS --sort). The filtered BAM is the required input for build/z-calling-base (-k 21 or -k 11) and for samtools fasta conversion feeding minimap2 mapping in the source-detection workflow.

## Related tools

- **Z-Calling** (Provides py/filter_bam.py, the script that applies the kinetic-tag, Median_PW/Median_IPD, and pass-count filters to produce clean BAMs for base calling and read classification) — https://github.com/xiaochuanle/Z-Calling
- **PacBio Circular Consensus Sequencing (CCS)** (Generates the input BAMs with --keep-kinetics so PW and IPD signals are available for filtering)
- **pbmm2** (Optionally aligns the filtered reads to a reference (--preset CCS --sort) when the input BAM was unmapped)

## Examples

```
python py/filter_bam.py -b <RawBAM> -o <FilteredBAM>
```

## Evaluation signals

- Every read in the output BAM carries the kinetic tags fi, ri, fp, and rp
- No retained read has Median_PW/Median_IPD ratio below 0.3 or above the 99th percentile of the distribution
- All retained reads satisfy fn >= 3 and rn >= 3 (minimum 3 passes on both forward and reverse strands)
- Downstream z-calling-read predict with min_len 500 should classify reads as '+', '-', or unclassified without failures caused by missing kinetic data
- In the metagenome benchmark, all mixtures with >= 25 dZ-positive yeast hybrid reads are called dZ-positive after filtering, with zero false positives

## Limitations

- The SVM read classifier is optimized for fully substituted dZ-genomes and may miss hybrid reads with diluted kinetic signatures (e.g. metagenome 7 with only 10 yeast hybrid reads was the sole false negative)
- PacBio kinetic noise is inherent: the subtle Z-induced PW shift (0.5–1.5-fold) overlaps with canonical Adenine variance, limiting hybrid sample quantification
- The data quality threshold recommends at least 60,000 reads per dataset with pass numbers no less than 3 for reliable analysis

## Evidence

- [discussion] Kinetic filter rationale in article: [evidence span withheld — anchor status: not_found]

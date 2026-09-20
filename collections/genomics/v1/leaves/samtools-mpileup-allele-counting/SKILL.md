---
name: samtools-mpileup-allele-counting
description: Use when validating whole-exome sequencing DNV calls from a parent-child
  trio by read-level allele counting at candidate sites to classify them into high-
  vs low-confidence callsets.
when_to_use_negative:
- The data are not WGS or WES short-read data — the Hare v1.2 workflow this filter
  relies on is effective for WGS and WES data only, not PacBio long-read data
- You lack BAM/CRAM files for the trio — mpileup read counting cannot be performed;
  HAT-FLEX operating directly on existing VCFs is the appropriate alternative
- The candidate sites fall outside the trimmed capture windows (±10 bp of the 50 bp
  buffered exon region), where coverage-based allele counting is not meaningful for
  WES
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Samtools
  role: Runs mpileup on the trio BAMs at every candidate DNV site to count alternate-allele
    supporting reads for the high/low confidence labeling rule
- name: Hare v1.2
  role: Provides the WES workflow inputs (trio DNV callset, capture-region handling)
    that frame the mpileup filter; effective for WGS and WES data only
  repo: https://github.com/TNTurnerLab/Hare
- name: acorn
  role: R package for follow-up statistics on HAT output, used to summarize the high/low
    split and confirmation rates per confidence class
  repo: https://github.com/TNTurnerLab/acorn
- name: HAT
  role: Source workflow whose DNV output is post-processed by this WES confidence
    filter
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:5cf4c75183c12a3017cbff2e33f38f7a7149ee24844b27055579f064c6adb016
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Hare
  tools:
  - HAT
  - Hare
  - Hare v1.2
  - Samtools
  - acorn
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# samtools-mpileup-allele-counting

## Summary

A WES follow-up filtering method that separates trio DNV calls into high- and low-confidence classes by trimming capture-region BED windows (±10 bp of a 50 bp buffered region) and running Samtools mpileup allele counting, requiring 0 alternate-allele reads in both parents and ≥1 in the child.

## When to use

Use when validating whole-exome sequencing DNV calls from a parent-child trio by read-level allele counting at candidate sites to classify them into high- vs low-confidence callsets.

## When NOT to use

- The data are not WGS or WES short-read data — the Hare v1.2 workflow this filter relies on is effective for WGS and WES data only, not PacBio long-read data
- You lack BAM/CRAM files for the trio — mpileup read counting cannot be performed; HAT-FLEX operating directly on existing VCFs is the appropriate alternative
- The candidate sites fall outside the trimmed capture windows (±10 bp of the 50 bp buffered exon region), where coverage-based allele counting is not meaningful for WES

## Inputs

- Trio DNV callset derived from SPARK (per-child candidate de novo variants)
- WES capture-region BED file (exon capture intervals)
- Aligned trio BAM/CRAM files (parent-father, parent-mother, child)
- Hare v1.2 WES workflow configuration

## Outputs

- Tidy per-child DNV output table with high/low confidence labels and full provenance
- High/low confidence split summary (expected 190 high / 92 low)
- Confirmation rates per confidence class (expected 91.6% high, 70.7% low)

## How to apply

Load the SPARK-derived trio DNV callset, the WES capture-region BED file, and the aligned trio BAM/CRAM files using the Hare v1.2 WES workflow inputs (effective for WGS and WES data only). Trim the WES capture regions to ±10 bp of a 50 bp buffered region per exon boundary to define the callable evaluation window. Run Samtools mpileup on the trio BAMs at every candidate DNV site and apply the read-count rule: the alternate allele must have 0 supporting reads in both parents (at any base quality) and ≥1 supporting read in the child. Candidates passing this mpileup check are labeled high-confidence DNVs and all remaining candidates are labeled low-confidence, written as tidy per-child outputs with full provenance; downstream summarization of the high/low split and confirmation rates can be done with the acorn R package or pandas.

## Related tools

- **Samtools** (Runs mpileup on the trio BAMs at every candidate DNV site to count alternate-allele supporting reads for the high/low confidence labeling rule)
- **Hare v1.2** (Provides the WES workflow inputs (trio DNV callset, capture-region handling) that frame the mpileup filter; effective for WGS and WES data only) — https://github.com/TNTurnerLab/Hare
- **acorn** (R package for follow-up statistics on HAT output, used to summarize the high/low split and confirmation rates per confidence class) — https://github.com/TNTurnerLab/acorn
- **HAT** (Source workflow whose DNV output is post-processed by this WES confidence filter) — https://github.com/TNTurnerLab/HAT

## Examples

```
samtools mpileup -f reference.fa -r chr1:start-end parent1.bam parent2.bam child.bam
```

## Evaluation signals

- The output table must reproduce the published 190/92 high/low split within tolerance
- Confirmation rates per class must match 91.6% (high) and 70.7% (low) within tolerance
- Every high-confidence call has exactly 0 alternate-allele reads in both parents and ≥1 alternate-allele read in the child in the mpileup output
- All evaluated sites fall within capture regions trimmed to ±10 bp of the 50 bp buffered exon-boundary window
- Per-child outputs are tidy and carry full provenance metadata

## Limitations

- The mpileup rule accepts any base quality for the child's ≥1 supporting read, so low-quality supporting reads can inflate the high-confidence class
- The Hare v1.2 workflow underlying this filter is effective for WGS and WES data only; PacBio long-read data require the Tortoise workflow
- Sites outside the trimmed ±10 bp capture windows cannot be evaluated by this WES-specific confidence labeling

## Evidence

- [methods] Trim the WES capture regions to +-10 bp of a 50 bp buffered region per exon boundary to define the callable evaluation window.: [evidence span withheld — anchor status: not_found]
- [methods] Run Samtools mpileup on the trio BAMs at every candidate DNV site, requiring the alternate allele to have 0 supporting reads in both parents and >=1 supporting read in the child at any base quality.: [evidence span withheld — anchor status: not_found]
- [methods] The updated Hare, v1.2, workflow can be found [here](https://github.com/TNTurnerLab/Hare). This workflow is effective for WGS and WES data only.: [evidence span withheld — anchor status: not_found]
- [methods] If you would like to do some followup statistics on the output from HAT, you can install the R package acorn: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
- [methods] To prep the output from HAT, you can run squirrel.py found here in this repo.: "To prep the output from HAT, you can run squirrel.py found here in this repo."

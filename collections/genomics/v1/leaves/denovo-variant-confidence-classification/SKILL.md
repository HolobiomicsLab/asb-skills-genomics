---
name: denovo-variant-confidence-classification
description: Use when classifying trio DNV calls from HAT/Hare WES workflows into
  high- vs low-confidence callsets using capture-region BED constraints and trio BAM/CRAM
  read support.
when_to_use_negative:
- The data are not WGS or WES — the Hare v1.2 workflow is effective for WGS and WES
  data only (e.g., PacBio HiFi long-read data belongs to the Tortoise workflow).
- You only have VCFs and no BAM/CRAM files — use HAT-FLEX instead, since mpileup-based
  read counting requires aligned reads.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: Hare v1.2
  role: Source of the WES/WGS trio DNV callset and workflow inputs fed into the confidence
    classification
  repo: https://github.com/TNTurnerLab/Hare
- name: Samtools
  role: mpileup read counting at candidate DNV sites to enforce 0 alt reads in parents
    and >=1 alt read in the child
- name: acorn
  role: R package for follow-up statistics and summarizing DNV characteristics of
    the classified high/low confidence callsets
  repo: https://github.com/TNTurnerLab/acorn
- name: squirrel.py
  role: Converts HAT output into acorn-compatible input before summarization
  repo: https://github.com/TNTurnerLab/HAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:198fe9d1f797dbbca89e859db0720f15d4a881007e5f2911137e2048849675c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Hare
  tools:
  - Hare
  - Hare v1.2
  - Samtools
  - acorn
  - squirrel.py
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# denovo-variant-confidence-classification

## Summary

A follow-up classification workflow that labels trio DNV calls (from HAT/Hare output) as high- or low-confidence for WES data by trimming capture regions to ±10 bp of a 50 bp buffered region and validating parental/child allele support with Samtools mpileup.

## When to use

Use when classifying trio DNV calls from HAT/Hare WES workflows into high- vs low-confidence callsets using capture-region BED constraints and trio BAM/CRAM read support.

## When NOT to use

- The data are not WGS or WES — the Hare v1.2 workflow is effective for WGS and WES data only (e.g., PacBio HiFi long-read data belongs to the Tortoise workflow).
- You only have VCFs and no BAM/CRAM files — use HAT-FLEX instead, since mpileup-based read counting requires aligned reads.

## Inputs

- SPARK-derived trio DNV callset (per-child DNV sites from HAT/Hare)
- WES capture-region BED file (exon targets)
- Aligned trio BAM/CRAM files (child, mother, father)

## Outputs

- Per-child tidy DNV table with high/low confidence labels and full provenance
- Summary of high/low split counts (expected 190 high / 92 low)
- Per-class confirmation rates (expected 91.6% high, 70.7% low)

## How to apply

Load the SPARK-derived trio DNV callset, the WES capture-region BED file, and aligned trio BAM/CRAM files using the Hare v1.2 WES workflow inputs, which are effective for WGS and WES data only. Trim each WES capture region to ±10 bp of a 50 bp buffered region per exon boundary to define the callable evaluation window — sites within this window are candidates for high confidence. Run Samtools mpileup on the trio BAMs at every candidate DNV site and label a site high confidence only if the alternate allele has 0 supporting reads in both parents (at any base quality) and ≥1 supporting read in the child; all remaining candidates are labeled low confidence. Write tidy per-child outputs with full provenance, then summarize the high/low split (expected 190 high / 92 low) and per-class confirmation rates (expected 91.6% high, 70.7% low) using the acorn R package or pandas.

## Related tools

- **Hare v1.2** (Source of the WES/WGS trio DNV callset and workflow inputs fed into the confidence classification) — https://github.com/TNTurnerLab/Hare
- **Samtools** (mpileup read counting at candidate DNV sites to enforce 0 alt reads in parents and >=1 alt read in the child)
- **acorn** (R package for follow-up statistics and summarizing DNV characteristics of the classified high/low confidence callsets) — https://github.com/TNTurnerLab/acorn
- **squirrel.py** (Converts HAT output into acorn-compatible input before summarization) — https://github.com/TNTurnerLab/HAT

## Examples

```
samtools mpileup -f GRCh38.fa -r chr1:start-end child.bam mother.bam father.bam; Rscript -e "library(acorn); d<-readDNV('classified_dnvs.txt')"
```

## Evaluation signals

- The high/low split must reproduce the published 190 high / 92 low counts within tolerance.
- Per-class confirmation rates must match 91.6% (high) and 70.7% (low) within tolerance.
- Every high-confidence site must show 0 supporting alternate-allele reads in both parents and >=1 supporting read in the child at any base quality in the mpileup output.
- The callable evaluation window must equal capture regions trimmed to ±10 bp of the 50 bp buffered region per exon boundary.
- Per-child output tables must be tidy and carry full provenance (input callset, BED version, BAM paths).

## Limitations

- Applies only to WGS and WES short-read data; PacBio HiFi long-read data require the Tortoise workflow.
- Requires BAM/CRAM access — if only VCFs are available, HAT-FLEX is the drop-in alternative.
- The mpileup rule accepts any base quality for the child's supporting read, so low-quality supporting reads can inflate the high-confidence class; the 190/92 and 91.6%/70.7% validation anchors guard against drift.

## Evidence

- [methods] The WES follow-up workflow trims capture regions (±10 bp of the 50 bp buffered region marks high confidence) and uses Samtools mpileup counts: [evidence span withheld — anchor status: not_found]
- [methods] if the alternate allele is absent in the parents and appears at least once in the child, the DNV is placed in the high confidence callset: [evidence span withheld — anchor status: not_found]
- [methods] Hare v1.2 workflow is effective for WGS and WES data only: [evidence span withheld — anchor status: not_found]
- [methods] followup statistics on HAT output via acorn: "If you would like to do some followup statistics on the output from HAT, you can install the R package acorn"
- [abstract] HAT takes aligned CRAM/BAM from a trio and outputs DNVs: "HAT is a computational workflow that begins with aligned read data (i.e. CRAM or BAM) from a parent–child sequenced trio and outputs DNVs."

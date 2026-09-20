---
name: exome-capture-interval-buffering
description: Use when validating de novo variant calls from whole-exome sequencing
  trios, where capture-region boundaries bias callability and DNVs must be split into
  high vs low confidence classes.
when_to_use_negative:
- Input data is WGS-only with no exome capture-region BED file — the ±10 bp/50 bp
  interval buffering step is specific to WES capture boundaries
- Only VCFs are available without BAM/CRAM files — HAT-FLEX is the caller-agnostic
  drop-in alternative, and mpileup read counting cannot be performed
- The data is PacBio HiFi long-read WGS, which the Hare v1.2 workflow does not cover
  (use Tortoise instead)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Hare v1.2
  role: Provides the workflow inputs (trio DNV callset, BAM/CRAM handling) for WES
    and WES follow-up; effective for WGS and WES data only
  repo: https://github.com/TNTurnerLab/Hare
- name: Samtools
  role: Runs mpileup on the trio BAMs at candidate DNV sites to count alternate-allele
    supporting reads (0 in parents, >=1 in child)
- name: acorn
  role: R package for follow-up statistics and summarization of HAT DNV output, prepped
    via squirrel.py
  repo: https://github.com/TNTurnerLab/acorn
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad775
    title: 'HAT: <i>de novo</i> variant calling for highly accurate short-read and
      long-read sequencing data'
  source_content_hash: sha256:4fc0e2400b03836355e9393c51e5c3448dacf968815d793edab74127eb0fe6dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TNTurnerLab/Hare
  tools:
  - Hare
  - Hare v1.2
  - Samtools
  - acorn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# exome-capture-interval-buffering

## Summary

A WES follow-up method from the HAT workflow that trims exome capture regions to ±10 bp around a 50 bp buffered exon boundary to define high-confidence callable windows, then labels trio DNVs high/low confidence using Samtools mpileup read counts in child and parents.

## When to use

Use when validating de novo variant calls from whole-exome sequencing trios, where capture-region boundaries bias callability and DNVs must be split into high vs low confidence classes.

## When NOT to use

- Input data is WGS-only with no exome capture-region BED file — the ±10 bp/50 bp interval buffering step is specific to WES capture boundaries
- Only VCFs are available without BAM/CRAM files — HAT-FLEX is the caller-agnostic drop-in alternative, and mpileup read counting cannot be performed
- The data is PacBio HiFi long-read WGS, which the Hare v1.2 workflow does not cover (use Tortoise instead)

## Inputs

- SPARK-derived trio DNV callset
- WES capture-region BED file
- Aligned trio BAM/CRAM files
- Reference genome FASTA for mpileup

## Outputs

- Tidy per-child DNV table labeled high-confidence vs low-confidence with full provenance
- High/low split summary (expected 190 high / 92 low) and per-class confirmation rates (expected 91.6% high / 70.7% low)

## How to apply

Load the SPARK-derived trio DNV callset, the WES capture-region BED file, and aligned trio BAM/CRAM files using the Hare v1.2 workflow inputs, which the paper restricts to WGS and WES data. Trim each WES capture interval so that only positions within ±10 bp of the 50 bp buffered region per exon boundary count as the high-confidence evaluation window. Run Samtools mpileup on the trio BAMs at every candidate DNV site and apply the read-count rule: 0 supporting reads for the alternate allele in both parents and >=1 supporting read in the child, at any base quality. Candidates passing the mpileup check are labeled high-confidence; all others are low-confidence, and results are written as tidy per-child outputs with full provenance for downstream summarization in the acorn R package or pandas.

## Related tools

- **Hare v1.2** (Provides the workflow inputs (trio DNV callset, BAM/CRAM handling) for WES and WES follow-up; effective for WGS and WES data only) — https://github.com/TNTurnerLab/Hare
- **Samtools** (Runs mpileup on the trio BAMs at candidate DNV sites to count alternate-allele supporting reads (0 in parents, >=1 in child))
- **acorn** (R package for follow-up statistics and summarization of HAT DNV output, prepped via squirrel.py) — https://github.com/TNTurnerLab/acorn

## Examples

```
samtools mpileup -f ref.fa -r chr1:12345-12345 -b trio_bam_list.txt > dnv_site.pileup
```

## Evaluation signals

- Output table reproduces the published 190 high / 92 low confidence DNV split within tolerance
- Per-class confirmation rates match 91.6% for high-confidence and 70.7% for low-confidence calls within tolerance
- Every high-confidence DNV has 0 alternate-allele supporting reads in both parents and >=1 supporting read in the child at any base quality
- All callable positions fall within ±10 bp of the 50 bp buffered region per exon boundary of the trimmed capture intervals
- Outputs are tidy per-child tables carrying full provenance of the filtering decisions

## Limitations

- The ±10 bp / 50 bp buffer thresholds and the 190/92, 91.6%/70.7% expectations are specific to the SPARK-derived WES trio callset; other datasets need their own validation targets
- Hare v1.2 workflow is effective for WGS and WES data only, so the interval-buffering follow-up does not extend to PacBio HiFi long-read data
- DNVs outside trimmed capture windows are inherently low-confidence; the method cannot rescue calls in poorly covered exome regions

## Evidence

- [other] Trim the WES capture regions to +-10 bp of a 50 bp buffered region per exon boundary to define the callable evaluation window.: [evidence span withheld — anchor status: not_found]
- [other] Run Samtools mpileup on the trio BAMs at every candidate DNV site, requiring the alternate allele to have 0 supporting reads in both parents and >=1 supporting read in the child at any base quality.: [evidence span withheld — anchor status: not_found]
- [methods] The updated Hare, v1.2, workflow can be found [here]. This workflow is effective for WGS and WES data only.: "This workflow is effective for WGS and WES data only."
- [readme] Acorn is an R package that works with de novo variants (DNVs) already called using a DNV caller.: [evidence span withheld — anchor status: external_doc]

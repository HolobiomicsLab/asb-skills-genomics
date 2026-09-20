---
name: sam-to-bam-conversion-paired-strand-alignments
description: Use when _align.py has emitted per-strand SAM files (output_CT.sam, output_GA.sam)
  from nanoEM C-to-T and G-to-A alignments and they must become sorted, indexed BAMs
  for methylation calling.
when_to_use_negative:
- Input files are already sorted, indexed BAMs (1.sorted.bam / 2.sorted.bam from minimap2)
  — these are best_align.py inputs, not its SAM outputs, and must not be re-sorted.
- You are still at the read-conversion stage with only *_CT.fq.gz / *_GA.fq.gz FASTQs
  — mapping with minimap2 must happen before this conversion step.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: samtools
  role: view -b converts the best-alignment SAMs to BAM; sort coordinate-sorts them;
    index builds the .bai index files required by sambamba mpileup and samtools merge
  repo: https://github.com/yos-sk/nanoEM
- name: pysam
  role: used inside best_align.py to compare mapping qualities between CT and GA alignments
    and write output_CT.sam / output_GA.sam that this step converts
  repo: https://github.com/yos-sk/nanoEM
- name: sambamba
  role: downstream consumer of the sorted BAMs (sambamba mpileup on output_CT.sorted.bam
    and output_GA.sorted.bam)
  repo: https://github.com/yos-sk/nanoEM
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:7d9f16266611e83d2ce002200c9be326ddbf2ac51b5e795001f6b4ff2a0924f5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yos-sk/nanoEM
  repo_url: https://github.com/yos-sk/nanoEM
  tools:
  - Python3
  - pysam
  - sambamba
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sam-to-bam-conversion-paired-strand-alignments

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts the best-alignment SAM outputs of the nanoEM best_align.py step (output_CT.sam and output_GA.sam, one per C-to-T / G-to-A strand) into coordinate-sorted, indexed BAM files so they can feed sambamba mpileup methylation calling and IGV bisulfite visualization.

## When to use

Use when best_align.py has emitted per-strand SAM files (output_CT.sam, output_GA.sam) from nanoEM C-to-T and G-to-A alignments and they must become sorted, indexed BAMs for methylation calling.

## When NOT to use

- Input files are already sorted, indexed BAMs (1.sorted.bam / 2.sorted.bam from minimap2) — these are best_align.py inputs, not its SAM outputs, and must not be re-sorted.
- You are still at the read-conversion stage with only *_CT.fq.gz / *_GA.fq.gz FASTQs — mapping with minimap2 must happen before this conversion step.

## Inputs

- output_CT.sam — best alignments for C-to-T converted reads produced by best_align.py
- output_GA.sam — best alignments for G-to-A converted reads produced by best_align.py

## Outputs

- output_CT.sorted.bam — coordinate-sorted BAM of best C-to-T alignments
- output_GA.sorted.bam — coordinate-sorted BAM of best G-to-A alignments
- output_CT.sorted.bam.bai and output_GA.sorted.bam.bai — samtools index files

## How to apply

After best_align.py compares mapping qualities between the C-to-T alignments (1.sorted.bam) and G-to-A alignments (2.sorted.bam) and writes the best alignments to output_CT.sam and output_GA.sam, convert each SAM to BAM with 'samtools view -b'. Coordinate-sort each BAM with 'samtools sort' to produce output_CT.sorted.bam and output_GA.sorted.bam, then index both with 'samtools index' to generate the .bai files. These sorted, indexed BAMs are the required input format for the downstream methylation-calling step (sambamba mpileup on output_CT.sorted.bam and output_GA.sorted.bam) and for the IGV bisulfite visualization BAM merge. SAM is kept as the interchange format from pysam-based best-alignment selection because BAM is what sambamba mpileup and samtools merge/index require.

## Related tools

- **samtools** (view -b converts the best-alignment SAMs to BAM; sort coordinate-sorts them; index builds the .bai index files required by sambamba mpileup and samtools merge) — https://github.com/yos-sk/nanoEM
- **pysam** (used inside best_align.py to compare mapping qualities between CT and GA alignments and write output_CT.sam / output_GA.sam that this step converts) — https://github.com/yos-sk/nanoEM
- **sambamba** (downstream consumer of the sorted BAMs (sambamba mpileup on output_CT.sorted.bam and output_GA.sorted.bam)) — https://github.com/yos-sk/nanoEM

## Examples

```
samtools view -b output_CT.sam | samtools sort -o output_CT.sorted.bam; samtools index output_CT.sorted.bam; samtools view -b output_GA.sam | samtools sort -o output_GA.sorted.bam; samtools index output_GA.sorted.bam
```

## Evaluation signals

- Each read from the original nanopore FASTQ appears exactly once across output_CT.sam and output_GA.sam, assigned to whichever strand alignment carried the higher mapping quality.
- Both output_CT.sorted.bam and output_GA.sorted.bam are coordinate-sorted (samtools sort completes without errors) and index successfully with samtools index, producing .bai files.
- sambamba mpileup runs on both sorted BAMs against cpg_sites.bed without complaining about missing indexes or unsorted records.
- samtools merge of output_CT.sorted.bam and the vis-converted output_GA.sorted.bam succeeds, confirming both BAMs share a consistent header/reference.

## Limitations

- Alignment itself was performed with minimap2 against the converted reference (output.fa); this step only formats/sorts/indexes the best-alignment output and does not remap or re-score reads.
- Multi-mapping is resolved upstream by best_align.py choosing the highest mapping quality between the CT and GA alignments; this step inherits that decision and cannot recover reads absent from the SAM inputs.
- No software version numbers are stated for samtools in the article or README, so exact sort/index behavior may vary across versions.

## Evidence

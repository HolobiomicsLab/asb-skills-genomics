---
name: mapping-quality-based-read-resolution
description: Use when you have two coordinate-sorted BAMs — one from C-to-T converted
  reads and one from G-to-A converted reads, both aligned to the same converted reference
  — and must assign each original FASTQ read to exactly one alignment before methylation
  calling.
when_to_use_negative:
- Reads have not been split into separate C-to-T and G-to-A FASTQs and aligned to
  the converted reference (output.fa) — comparing unconverted alignments is meaningless
  here
- You need direct signal-level methylation from native DNA; use nanopolish call-methylation
  on nanopore WGS data instead of this conversion-resolution step
- You intend to distinguish 5mC from 5hmC on the same molecule — the article states
  nanoEM cannot distinguish between them
edam_operation: http://edamontology.org/operation_0328
edam_topics:
- http://edamontology.org/topic_3295
- http://edamontology.org/topic_0622
tools:
- name: pysam
  role: Python library used inside best_align.py to read both sorted BAMs, compare
    mapping qualities per read, and write the best-alignment SAM files
  repo: https://github.com/yos-sk/nanoEM
- name: samtools
  role: Converts output_CT.sam/output_GA.sam to BAM ('samtools view -b'), coordinate-sorts
    them, and indexes the sorted BAMs
- name: minimap2
  role: Upstream aligner (map-ont, --eqx) that produced 1.sorted.bam and 2.sorted.bam
    from the C-to-T and G-to-A FASTQs against output.fa
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab397
    title: Long-read whole-genome methylation patterning using enzymatic base conversion
      and nanopore sequencing
  source_content_hash: sha256:bc97985a4a1a82e549c32ea8cd8003dffb838ec19c883ea9119d9d62f8c8e49c
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
  - minimap2
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0328
  edam_topics:
  - http://edamontology.org/topic_3295
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# mapping-quality-based-read-resolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Resolve double-alignment ambiguity in the nanoEM pipeline: because each original nanopore read is converted twice (C-to-T and G-to-A), both versions align to the same converted reference, and this skill compares mapping qualities per read to keep only the best alignment.

## When to use

Use when you have two coordinate-sorted BAMs — one from C-to-T converted reads and one from G-to-A converted reads, both aligned to the same converted reference — and must assign each original FASTQ read to exactly one best alignment before methylation calling.

## When NOT to use

- Reads have not been split into separate C-to-T and G-to-A FASTQs and aligned to the converted reference (output.fa) — comparing unconverted alignments is meaningless here
- You need direct signal-level methylation from native DNA; use nanopolish call-methylation on nanopore WGS data instead of this conversion-resolution step
- You intend to distinguish 5mC from 5hmC on the same molecule — the article states nanoEM cannot distinguish between them

## Inputs

- C-to-T converted read alignments as coordinate-sorted BAM (1.sorted.bam)
- G-to-A converted read alignments as coordinate-sorted BAM (2.sorted.bam)
- Original nanopore FASTQ read file
- Python3 with pysam; samtools on PATH

## Outputs

- output_CT.sam and output_GA.sam (best alignments per read)
- output_CT.sorted.bam and output_GA.sorted.bam after samtools view -b and sort
- output_CT.sorted.bam.bai and output_GA.sorted.bam.bai index files

## How to apply

Run best_align.py with pysam, passing the C-to-T alignment BAM (1.sorted.bam) as --bam1, the G-to-A alignment BAM (2.sorted.bam) as --bam2, and the original nanopore FASTQ as --fastq; the script compares mapping qualities per read and writes the winning alignments to output_CT.sam and output_GA.sam. When both the C-to-T and G-to-A versions of the same read align, this is judged as multi-mapping and the alignment with the highest mapping quality value is retained. Then convert each SAM to BAM with 'samtools view -b', coordinate-sort with 'samtools sort', and index with 'samtools index' to produce output_CT.sorted.bam(.bai) and output_GA.sorted.bam(.bai), which feed downstream sambamba mpileup methylation calling. The rationale is that nanopore reads are error-prone and mostly C-to-T converted, so the strand-conversion giving the higher MAPQ is the more trustworthy placement.

## Related tools

- **pysam** (Python library used inside best_align.py to read both sorted BAMs, compare mapping qualities per read, and write the best-alignment SAM files) — https://github.com/yos-sk/nanoEM
- **samtools** (Converts output_CT.sam/output_GA.sam to BAM ('samtools view -b'), coordinate-sorts them, and indexes the sorted BAMs)
- **minimap2** (Upstream aligner (map-ont, --eqx) that produced 1.sorted.bam and 2.sorted.bam from the C-to-T and G-to-A FASTQs against output.fa)

## Examples

```
python src/best_align.py --bam1 1.sorted.bam --bam2 2.sorted.bam --fastq fastq; samtools view -b output_CT.sam | samtools sort -o output_CT.sorted.bam; samtools index output_CT.sorted.bam
```

## Evaluation signals

- Each read from the input FASTQ appears exactly once across the pair of output_CT.sam and output_GA.sam
- Each read is assigned to whichever strand alignment (CT or GA) carried the higher mapping quality value
- Both output_CT.sorted.bam and output_GA.sorted.bam index successfully with 'samtools index' without errors
- Sorted BAMs can be consumed downstream by 'sambamba mpileup -L cpg_sites.bed' without missing-read errors

## Limitations

- Reads where both CT and GA alignments carry similar MAPQ may still be ambiguous; the rule is simply to pick the highest mapping quality value
- No software version numbers are stated for best_align.py, pysam, or samtools in the article or README
- The judged multi-mapping behavior depends on both conversions being run identically against the same converted reference (output.fa)

## Evidence

- [methods] when both the C-to-T read and the G-to-A read derived from the same original read are aligned, this is judged as multi-mapping: [evidence span withheld — anchor status: not_found]

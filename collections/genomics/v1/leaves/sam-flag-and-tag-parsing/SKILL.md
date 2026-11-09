---
name: sam-flag-and-tag-parsing
description: Reference for interpreting SAM alignment flags and auxiliary tags when
  scanning synthetic long-read (SLR) BAMs to identify badly aligned read pairs or
  group reads by BX barcodes.
when_to_use_negative:
- Input is not an indexed SLR BAM with BX tags — Blackbird's -b input requires position-sorted
  and indexed reads with barcode tags
- The goal is to keep all reads for assembly, e.g. when invoking Blackbird with the
  -n option which skips collecting poorly aligned reads
- The reads are plain short reads without barcode information, where barcode-aware
  retrieval M(B) has no meaning
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Implements the read-scanning routines that parse SAM flags/tags and build
    the read map M of badly aligned reads
  repo: github.com/1dayac/Blackbird
- name: BWA
  role: Provides the BWA-indexed reference genome against which SLR reads are aligned
    before flag/tag parsing
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:9bc8a87108cb71a5a005c2a8485137202f9dc41ff12a98db61bea45a977972a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BWA
  - Blackbird
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# SAM flag and tag parsing

## Summary

Parses SAM/BAM alignment flags and auxiliary tags (AM, BX) to flag badly aligned read pairs in position-sorted SLR BAM files and to key barcoded reads into Blackbird's read map M for later barcode-based retrieval.

## When to use

Reference for interpreting SAM alignment flags and auxiliary tags when scanning synthetic long-read (SLR) BAMs to identify badly aligned read pairs or group reads by BX barcodes.

## When NOT to use

- Input is not an indexed SLR BAM with BX tags — Blackbird's -b input requires position-sorted and indexed reads with barcode tags
- The goal is to keep all reads for assembly, e.g. when invoking Blackbird with the -n option which skips collecting poorly aligned reads
- The reads are plain short reads without barcode information, where barcode-aware retrieval M(B) has no meaning

## Inputs

- Position-sorted and indexed SLR read BAM file with BX barcode tags
- Optional paired position-sorted and indexed long-read BAM and matching FASTQ

## Outputs

- Badly aligned read set R (flagged read pairs)
- Read map M: parallel flat hash map keyed by barcode supporting M(B) membership queries
- Filtered read stream feeding barcode-aware sliding-window local assembly

## How to apply

Scan a position-sorted and indexed SLR read BAM annotated with BX barcode tags and inspect each read pair's alignment state and tags. Mark pairs as badly aligned when reads have base qualities above Phred 20 that are unplaced or soft-clipped over more than 20% of their length, or when they carry a zero/absent AM tag. Accumulate flagged pairs into the badly aligned read set R, then store R in a parallel flat hash map (read map M) keyed by barcode so that M(B) queries can retrieve reads sharing a given barcode list. Note that running Blackbird with this filter enabled removes reads relative to the -n ('Don't collect poorly aligned reads') configuration, which bypasses this collection step.

## Related tools

- **Blackbird** (Implements the read-scanning routines that parse SAM flags/tags and build the read map M of badly aligned reads) — github.com/1dayac/Blackbird
- **BWA** (Provides the BWA-indexed reference genome against which SLR reads are aligned before flag/tag parsing)

## Examples

```
/home/dmm2017/Blackbird/assembler/bin/blackbird -b /local/storage/data/10X/HG002/NA24385.GRCh37.phased_possorted_bam.bam -r /local/workdir/dmm2017/hg37/refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- The hash map resolves M(B) queries for every barcode in a given barcode list B
- Running with the badly-aligned-read filter enabled removes reads relative to the -n (don't collect poorly aligned reads) configuration
- Every flagged pair satisfies the criteria: Phred 20+ bases unplaced or soft-clipped over >20% of read length, or zero/absent AM tag
- Input BAM is position-sorted and indexed with BX tags present, as required by the --bam/-b option

## Limitations

- AM tag is optional information: reads lacking it are treated as badly aligned, which may exclude usable pairs if the aligner does not emit AM tags
- Blackbird 0.1 is an early release forked from the SPAdes codebase, and no changelog was found in the article's discussion
- Calls in out.vcf (other than out_50.vcf, calls >50 bp) are explicitly described as not reliable

## Evidence

- [readme] position-sorted and indexed SLR Read bam-file with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Don't collect poorly aligned reads: "`-n` - Don't collect poorly aligned reads"
- [methods] badly-aligned-read criteria with Phred 20 and 20% soft-clip thresholds and AM tag check: [evidence span withheld — anchor status: not_found]
- [methods] read map M is a parallel flat hash map supporting M(B) queries: [evidence span withheld — anchor status: not_found]
- [methods] Blackbird uses barcode-aware sliding window assembly of genome segments: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."

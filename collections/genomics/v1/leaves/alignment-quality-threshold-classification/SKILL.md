---
name: alignment-quality-threshold-classification
description: Use when SLR read pairs in a BX-tagged BAM must be flagged as badly aligned
  (unplaced or soft-clipped >20% of length at Phred >20, or zero/absent AM tag) prior
  to barcode-based local assembly of genome segments.
when_to_use_negative:
- Input reads are long reads (PacBio HiFi) rather than SLR/10X BX-tagged reads — Blackbird's
  SLR alignment filtering criteria are designed for the SLR BAM
- The BAM is not position-sorted and indexed or lacks BX tags, since the criteria
  and downstream barcode queries depend on them
- You are running a long-read-only SV caller (e.g. PBSV or Sniffles2) that does not
  consume the SLR read map M
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Scans the SLR BAM, applies the badly-aligned-read thresholds, and builds the
    read map M of poorly aligned reads
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for blackbird
  role: Shared codebase / spades.py backend required alongside Blackbird for local
    assembly of segments after filtering
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:2a98f89c5d4bf31107a0fb4e42852981ecf138b06c8dfc413ecca310e6a18ac0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - Blackbird
  - SPAdes for blackbird
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# alignment-quality-threshold-classification

## Summary

Classifies read pairs from a position-sorted, indexed SLR BAM (with BX tags) as badly aligned using fixed alignment-quality thresholds, and accumulates the flagged reads into a badly-aligned read set R for exclusion or tracking during Blackbird's barcode-aware sliding-window assembly.

## When to use

Use when SLR read pairs in a BX-tagged BAM must be flagged as badly aligned (unplaced or soft-clipped >20% of length at Phred >20, or zero/absent AM tag) prior to barcode-based local assembly of genome segments.

## When NOT to use

- Input reads are long reads (PacBio HiFi) rather than SLR/10X BX-tagged reads — Blackbird's SLR alignment filtering criteria are designed for the SLR BAM
- The BAM is not position-sorted and indexed or lacks BX tags, since the criteria and downstream barcode queries depend on them
- You are running a long-read-only SV caller (e.g. PBSV or Sniffles2) that does not consume the SLR read map M

## Inputs

- Position-sorted and indexed SLR read BAM file with BX barcode tags (Blackbird --bam/-b)
- BWA-indexed reference genome (Blackbird --rerefence/-r)

## Outputs

- Badly aligned read set R (read pairs flagged by the Phred 20 / 20% soft-clip / AM-tag criteria)
- Read map M supporting barcode subset queries M(B) used by downstream barcode-aware sliding-window assembly

## How to apply

Open the position-sorted and indexed SLR read BAM annotated with BX barcode tags using Blackbird's read-scanning routines (Blackbird is forked from the SPAdes codebase and compiled via spades_compile.sh). For each read pair, apply the badly-aligned criteria: pairs containing reads with base qualities above Phred 20 that are unplaced or soft-clipped over more than 20% of their length, or that carry a zero/absent AM tag, are marked badly aligned. Accumulate all flagged pairs into the set R, which can be stored in a parallel flat hash map (read map M) so that membership queries M(B) for any barcode list B are efficient. By default these poorly aligned reads are collected for downstream handling; running with Blackbird's hidden -n option skips collecting them. Validate by confirming M(B) resolves for every barcode in the list and that enabling the filter changes results relative to the -n configuration.

## Related tools

- **Blackbird** (Scans the SLR BAM, applies the badly-aligned-read thresholds, and builds the read map M of poorly aligned reads) — https://github.com/1dayac/Blackbird
- **SPAdes for blackbird** (Shared codebase / spades.py backend required alongside Blackbird for local assembly of segments after filtering) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
/home/dmm2017/Blackbird/assembler/bin/blackbird -b /local/storage/data/10X/HG002/NA24385.GRCh37.phased_possorted_bam.bam -r /local/workdir/dmm2017/hg37/refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Every flagged pair satisfies at least one criterion: Phred >20 bases unplaced or soft-clipped over >20% of read length, or a zero/absent AM tag
- Read map M answers M(B) queries for every barcode in the barcode list without misses
- Running with the filter enabled removes reads relative to the -n (don't collect poorly aligned reads) configuration
- Input BAM is confirmed position-sorted, indexed, and BX-tagged before filtering begins
- Downstream out_50.vcf is produced with insertion and deletion calls longer than 50 bp after assembly on the filtered read set

## Limitations

- Thresholds (Phred 20, >20% unplaced/soft-clipped length, AM-tag presence) are fixed in the source card; no user-facing CLI option is documented to tune them — only the hidden -n flag toggles collection
- out.vcf calls other than those in out_50.vcf are explicitly described as 'not reliable' in the README
- Hidden options like -n are undocumented in the main options list, so behavior changes may be easy to miss

## Evidence

- [readme] `--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Don't collect poorly aligned reads: "`-n` - Don't collect poorly aligned reads"
- [methods] Blackbird employs the barcode information encoded in SLR reads: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"
- [methods] barcode-aware sliding window assembly: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."

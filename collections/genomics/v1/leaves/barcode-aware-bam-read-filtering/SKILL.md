---
name: barcode-aware-bam-read-filtering
description: Use when preprocessing SLR (e.g. 10X linked-read) BAMs with BX tags before
  barcode-aware sliding-window assembly, to exclude poorly aligned reads while retaining
  barcode-based retrieval.
when_to_use_negative:
- The input BAM is not position-sorted and indexed or lacks BX barcode tags, so M(B)
  queries cannot be built or resolved
- You want no poorly aligned reads collected at all — use the hidden -n option instead
  of running this filter
- Reads are plain long reads (PacBio Hi-Fi / ONT) rather than SLR reads with barcode
  tags; Blackbird's SLR mode and its filter are designed around SLR BAM input
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Blackbird
  role: Performs the BAM scan, applies badly-aligned-read criteria (Phred 20 / >20%
    unplaced or soft-clipped / zero-or-absent AM tag), and builds the read map M hash
    structure; the hidden -n option disables collection of poorly aligned reads
  repo: https://github.com/1dayac/Blackbird
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:2a8ffc042bb83d47c7d1848677217e3e6f9fe4c012c17b170d212d3045a0582e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - Blackbird
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Barcode-aware BAM read filtering

## Summary

A filtering step in Blackbird that scans a position-sorted, indexed SLR read BAM with BX barcode tags, flags badly aligned read pairs (unplaced/soft-clipped over >20% of length at Phred >20, or zero/absent AM tag), and indexes the resulting read set R in a parallel flat hash map (read map M) so reads sharing barcodes can be retrieved as M(B) during barcode-based local assembly.

## When to use

Use when preprocessing SLR (e.g. 10X linked-read) BAMs with BX tags before barcode-aware sliding-window assembly, to exclude poorly aligned reads while retaining barcode-based retrieval.

## When NOT to use

- The input BAM is not position-sorted and indexed or lacks BX barcode tags, so M(B) queries cannot be built or resolved
- You want no poorly aligned reads collected at all — use the hidden -n option instead of running this filter
- Reads are plain long reads (PacBio Hi-Fi / ONT) rather than SLR reads with barcode tags; Blackbird's SLR mode and its filter are designed around SLR BAM input

## Inputs

- Position-sorted and indexed SLR read BAM file with BX barcode tags (e.g. a 10X phased possorted BAM)
- Barcode list B for which M(B) queries must be answerable

## Outputs

- Badly aligned read set R stored in a parallel flat hash map (read map M) supporting M(B) subset queries per barcode

## How to apply

Open the position-sorted and indexed SLR read BAM annotated with BX barcode tags using Blackbird's read-scanning routines. For each read pair, apply the badly-aligned criteria: mark pairs as badly aligned if any read has base qualities above Phred 20 that are unplaced or soft-clipped over more than 20% of its length, or if the pair carries a zero or absent AM tag; accumulate flagged pairs into the set R. Store R in a parallel flat hash map keyed by barcode so membership queries M(B) — the subset of reads in R carrying a barcode from a given barcode list B — are efficient for downstream barcode-aware sliding-window assembly. This behavior can be toggled off with the hidden Blackbird option -n ('Don't collect poorly aligned reads'), which serves as the natural A/B comparison for validation.

## Related tools

- **Blackbird** (Performs the BAM scan, applies badly-aligned-read criteria (Phred 20 / >20% unplaced or soft-clipped / zero-or-absent AM tag), and builds the read map M hash structure; the hidden -n option disables collection of poorly aligned reads) — https://github.com/1dayac/Blackbird

## Examples

```
blackbird -b /local/storage/data/10X/HG002/NA24385.GRCh37.phased_possorted_bam.bam -r /local/workdir/dmm2017/hg37/refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Every barcode in the list B resolves to a non-error M(B) subset query against the hash map
- Running with the filter enabled removes reads relative to the -n (don't collect poorly aligned reads) configuration
- Flagged read pairs satisfy at least one criterion: base qualities above Phred 20 unplaced or soft-clipped over >20% of read length, or zero/absent AM tag
- All reads retained in R carry BX barcode tags from the input BAM

## Limitations

- Requires a position-sorted and indexed BAM with BX tags; unsorted or unindexed BAM files cannot be scanned correctly
- Reads classified as badly aligned are excluded from downstream assembly windows, so reads mislabeled by the criteria could remove informative signal for nearby SV calls

## Evidence

- [methods] Badly aligned read criteria: [evidence span withheld — anchor status: not_found]
- [results] Read map M storage structure: [evidence span withheld — anchor status: not_found]
- [readme] Input BAM requirement: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Hidden filter toggle: "-n - Don't collect poorly aligned reads"

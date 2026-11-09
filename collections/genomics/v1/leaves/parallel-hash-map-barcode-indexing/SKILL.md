---
name: parallel-hash-map-barcode-indexing
description: Use when building or querying an in-memory barcode-indexed store of poorly
  aligned reads from a BX-tagged SLR BAM, e.g. to let barcode-aware sliding-window
  assembly skip badly aligned read pairs.
when_to_use_negative:
- Running Blackbird with the hidden `-n` flag, which explicitly disables collection
  of poorly aligned reads
- Input reads lack BX barcode tags or the BAM is not position-sorted/indexed, so barcode-keyed
  queries cannot be built
- The analysis does not involve SLR/hybrid SV calling with barcode-aware local assembly,
  where badly aligned reads would never be re-queried
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Provides the read-scanning routines, badly-aligned-read criteria, and the
    parallel flat hash map read map M used to store and query R; the `-n` hidden option
    toggles collection of poorly aligned reads.
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for Blackbird
  role: Forked codebase from which Blackbird is compiled and run (`spades.py` path
    passed via `-s`), providing the environment in which read map M is built and consumed.
  repo: https://github.com/1dayac/spades_for_blackbird
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:4eda035dc908a7dde9a99101c3936538ae389c374148a6ea1a5f320b9f0afd14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - Blackbird
  - SPAdes for Blackbird
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# parallel-hash-map-barcode-indexing

## Summary

Implements Blackbird's read map M: a parallel flat hash map that stores the badly aligned read set R from a position-sorted, BX-tagged SLR BAM file, keyed so that M(B) — the reads sharing any barcode in a list B — can be efficiently retrieved during barcode-based local assembly.

## When to use

Use when building or querying an in-memory barcode-indexed store of poorly aligned reads from a BX-tagged SLR BAM, e.g. to let barcode-aware sliding-window assembly skip badly aligned read pairs.

## When NOT to use

- Running Blackbird with the hidden `-n` flag, which explicitly disables collection of poorly aligned reads
- Input reads lack BX barcode tags or the BAM is not position-sorted/indexed, so barcode-keyed queries cannot be built
- The analysis does not involve SLR/hybrid SV calling with barcode-aware local assembly, where badly aligned reads would never be re-queried

## Inputs

- Position-sorted and indexed SLR read BAM file with BX barcode tags
- Badly-aligned-read criteria thresholds: Phred 20 base-quality cutoff, >20% unplaced/soft-clipped read length, zero/absent AM tag
- Barcode list B for M(B) membership queries

## Outputs

- Read map M: a parallel flat hash map containing the badly aligned read set R, supporting M(B) subset retrieval by barcode

## How to apply

Scan a position-sorted and indexed SLR read BAM annotated with BX barcode tags using Blackbird's read-scanning routines. For each read pair, flag it as badly aligned when reads have base qualities above Phred 20 but are unplaced or soft-clipped over more than 20% of their length, or carry a zero/absent AM tag; accumulate all flagged pairs into the set R. Store R in a parallel flat hash map (read map M) keyed by barcode so that membership queries M(B) for any barcode list B are answered in time suitable for the downstream barcode-aware sliding-window assembly. Running with this filter enabled (i.e. without the hidden `-n` 'don't collect poorly aligned reads' flag) changes results relative to the -n configuration, which is the intended behavioral check.

## Related tools

- **Blackbird** (Provides the read-scanning routines, badly-aligned-read criteria, and the parallel flat hash map read map M used to store and query R; the `-n` hidden option toggles collection of poorly aligned reads.) — https://github.com/1dayac/Blackbird
- **SPAdes for Blackbird** (Forked codebase from which Blackbird is compiled and run (`spades.py` path passed via `-s`), providing the environment in which read map M is built and consumed.) — https://github.com/1dayac/spades_for_blackbird

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -t 32 -s spades.py
```

## Evaluation signals

- Confirm the hash map resolves M(B) queries for every barcode in the barcode list B
- Run Blackbird with the filter enabled and verify the read count differs from (is reduced relative to) the `-n` (don't collect poorly aligned reads) configuration
- Verify every read stored in M satisfies at least one badly-aligned criterion: >20% unplaced/soft-clipped at Phred >= 20, or a zero/absent AM tag
- Check that each read pair in R carries a BX barcode key retrievable via M(B)

## Limitations

- The poorly aligned read collection is only an optional/hidden behavior (`-n` disables it), so runs may silently omit the read map unless explicitly enabled
- Only reads flagged by Blackbird's specific criteria (Phred 20, >20% soft-clip/unplaced, zero/absent AM tag) enter R; other low-quality reads are not indexed
- Output calls outside `out_50.vcf` (in `out.vcf`) are described as 'not reliable', so downstream users should not treat all calls as high confidence

## Evidence

- [other] The badly aligned read set R is stored in a parallel flat hash map data structure called read map M, whose internal structure supports efficient querying: [evidence span withheld — anchor status: not_found]
- [methods] pairs containing reads with base qualities above Phred 20 that are unplaced or soft-clipped over more than 20% of their length, or that carry a zero/absent AM tag, are marked as badly aligned: [evidence span withheld — anchor status: not_found]
- [readme] `-n` - Don't collect poorly aligned reads: "`-n` - Don't collect poorly aligned reads"
- [readme] `--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"

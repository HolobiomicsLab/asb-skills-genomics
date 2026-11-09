---
name: bits-per-base-normalization
description: Use when comparing compressed sizes of NanoSpring output streams (or
  any sequence compressor's output) across FASTQ datasets of unequal length, e.g.
  reproducing a per-stream compression breakdown like Figure 3.
when_to_use_negative:
- The analysis targets quality score or read identifier compression — NanoSpring does
  not compress these, so bits-per-base accounting of its streams cannot cover them.
- Comparing against reference-based compressors whose stream structure differs, so
  per-stream breakdowns are not directly comparable without redefining stream categories.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: NanoSpring
  role: Reference-free compressor whose .NanoSpring tar archive is decomposed into
    consensus, error, and lone-read streams for per-stream bits-per-base measurement
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: Baseline general-purpose compressor whose bits-per-base compression NanoSpring
    is reported to beat 3-6x
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:d776cee84b33804f182c8a21246cad3b90c2bacb668137f5592a21ed45b1cfa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# bits-per-base normalization

## Summary

Normalizes compressed byte sizes of individual compressed streams (consensus, error, lone-read) by the total number of sequenced bases, yielding bits-per-base per stream so compression contributions can be compared across nanopore FASTQ datasets of different sizes.

## When to use

Use when comparing compressed sizes of NanoSpring output streams (or any sequence compressor's output) across FASTQ datasets of unequal length, e.g. reproducing a per-stream compression breakdown like Figure 3.

## When NOT to use

- The analysis targets quality score or read identifier compression — NanoSpring does not compress these, so bits-per-base accounting of its streams cannot cover them.
- Comparing against reference-based compressors whose stream structure differs, so per-stream breakdowns are not directly comparable without redefining stream categories.

## Inputs

- FASTQ (or gzipped FASTQ) nanopore read files, one per dataset (e.g. Table 1 datasets: snail, banana, hs1, hs4)
- NanoSpring compressed .NanoSpring archives (tar archives of the compressed streams)
- Total base counts per dataset from the corresponding FASTQ files

## Outputs

- Per-stream byte sizes for consensus, error, and lone-read streams per dataset
- Bits-per-base table per stream per dataset (e.g. stream_bits_per_base.csv)

## How to apply

Run NanoSpring on each FASTQ dataset (e.g. snail, banana, hs1, hs4 from Table 1) with './NanoSpring -c -i <file.fastq> -o <file.NanoSpring>' using the default 20 threads. Untar the resulting .NanoSpring archive — compressed output is a tar of separate streams — to obtain individual consensus, error, and lone-read stream files. Measure each stream's byte size with standard file-size utilities (e.g. ls -l, stat) and divide by the total number of bases in the source FASTQ, multiplying by 8 to convert bytes to bits. Tabulate bits-per-base for consensus, error, and lone-read streams across all datasets (e.g. as stream_bits_per_base.csv). Rationale: bits-per-base makes stream contributions comparable regardless of dataset depth, since NanoSpring compresses only read sequences (not identifiers or quality values) and reports 0.35-0.65 bits per base for recent high-quality nanopore datasets.

## Related tools

- **NanoSpring** (Reference-free compressor whose .NanoSpring tar archive is decomposed into consensus, error, and lone-read streams for per-stream bits-per-base measurement) — https://github.com/qm2/NanoSpring
- **gzip** (Baseline general-purpose compressor whose bits-per-base compression NanoSpring is reported to beat 3-6x)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring && tar -xf file.NanoSpring && stat -c%s <streams> # divide stream byte sizes by total FASTQ bases x8 for bits/base
```

## Evaluation signals

- Error stream should contribute close to 0.5 bits/base for most datasets, matching the paper's reported pattern
- Consensus stream should be a small share of total bits-per-base relative to error and lone-read streams
- Lone-read stream should be large for snail, banana, and hs1 but negligible for hs4
- Total over all streams should fall in the 0.35-0.65 bits/base range for recently basecalled high-quality nanopore datasets
- Sum of per-stream bits-per-base, scaled by total bases, should reconcile with the overall .NanoSpring archive size

## Limitations

- NanoSpring compresses only base sequences; bits-per-base results exclude read identifiers and quality scores, so totals cannot be compared against full-FASTQ compressors like ENANO without adjustment.
- Stream breakdown pattern varies widely across datasets — lone-read contribution is dataset-dependent, so a single dataset's breakdown is not generalizable.
- Ages/qualities of basecalling matter: the 0.35-0.65 bits/base range applies to recently basecalled high-quality datasets; older, noisier reads will not match it.

## Evidence

- [readme] compressed files are tar archives of streams: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"
- [abstract] NanoSpring 0.35-0.65 bits per base: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [methods] compression command with defaults: "For compressing file.fastq using default 20 threads.
``n./NanoSpring -c -i file.fastq -o file.NanoSpring
"
- [readme] NanoSpring ignores identifiers and quality values: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
- [abstract] 3-6x better than gzip: "0.35-0.65 bits per base which is 3-6[Formula: see text] lower than general purpose compressors like gzip"

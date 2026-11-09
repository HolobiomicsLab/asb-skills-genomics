---
name: fastq-base-count-accounting
description: Use when you need a per-dataset base-count denominator (e.g., FASTQ datasets
  like snail, banana, hs1, hs4) to normalize compressed stream sizes into bits-per-base,
  or to reconcile stream contributions against total compressed size.
when_to_use_negative:
- The FASTQ encodes data compressed by a tool that also compresses read identifiers
  or quality scores, since a sequence-only base count would not cover the full archive
  size
- The input is already a bits-per-base table or the compressed archive has not been
  decomposed into its individual streams
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Reference-free nanopore FASTQ compressor whose .NanoSpring output (a tar archive
    of compressed streams) is decomposed and normalized by per-stream base counts
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose compressor used as a baseline compression reference for the
    same FASTQ datasets
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:5cafe73e1be4deb3f83644eed82c7b1d097a554aacd1aa797a83e530e659f370
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# FASTQ base-count accounting

## Summary

Counts the number of bases in FASTQ read files and uses that denominator to convert compressed stream byte sizes into bits-per-base contributions (e.g., for NanoSpring's consensus, error, and lone-read streams), enabling fair cross-dataset comparison of compression breakdowns.

## When to use

Use when you need a per-dataset base-count denominator (e.g., FASTQ datasets like snail, banana, hs1, hs4) to normalize compressed stream sizes into bits-per-base, or to reconcile stream contributions against total compressed size.

## When NOT to use

- The FASTQ encodes data compressed by a tool that also compresses read identifiers or quality scores, since a sequence-only base count would not cover the full archive size
- The input is already a bits-per-base table or the compressed archive has not been decomposed into its individual streams

## Inputs

- FASTQ files of nanopore sequencing reads (e.g., snail, banana, hs1, hs4 from Table 1; gzipped FASTQ also supported)
- NanoSpring compressed archives (.NanoSpring) untarred into individual compressed stream files (consensus, error, lone-read)

## Outputs

- Total base count per FASTQ dataset
- Bits-per-base per compressed stream per dataset (consensus, error, lone-read)
- Tabulated stream breakdown file (e.g., stream_bits_per_base.csv)

## How to apply

For each FASTQ (plain or gzipped) dataset in the comparison table, count total bases by summing the length of every 4th line (the sequence lines) in the file; NanoSpring compresses only the read sequences and ignores read identifiers and quality values, so this base count is the correct normalization denominator. After compressing with NanoSpring and untarring the resulting .NanoSpring tar archive (compressed files are tar archives of the different compressed streams), measure the byte size of each stream (consensus, error, lone-read) with file-size utilities. Divide each stream's size in bytes by the total base count and multiply by 8 to obtain bits-per-base per stream. Tabulate the three streams' bits-per-base for every dataset (e.g., in stream_bits_per_base.csv) and sanity-check that the streams sum to approximately the overall bits-per-base reported for the tool (0.35-0.65 bits/base for recent high-quality nanopore datasets).

## Related tools

- **NanoSpring** (Reference-free nanopore FASTQ compressor whose .NanoSpring output (a tar archive of compressed streams) is decomposed and normalized by per-stream base counts) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose compressor used as a baseline compression reference for the same FASTQ datasets)

## Examples

```
awk 'NR%4==2{n+=length($0)} END{print n}' file.fastq; ./NanoSpring -c -i file.fastq -o file.NanoSpring; tar -xf file.NanoSpring -C streams/ && ls -l streams/
```

## Evaluation signals

- Stream bits-per-base values sum to approximately the total bits-per-base of the .NanoSpring archive for each dataset
- Total bits-per-base falls in the reported 0.35-0.65 range for recently basecalled high-quality nanopore datasets
- Error stream contributes roughly 0.5 bits/base across most datasets
- Lone-read stream is large for snail, banana and hs1 but negligible for hs4, matching the paper's reported pattern

## Limitations

- NanoSpring compresses only base sequences, ignoring read identifiers and quality values, so base-count normalization reflects sequence compression only
- Results depend on basecaller quality: the 0.35-0.65 bits/base range applies to recently basecalled high-quality nanopore datasets; older noisier reads compress worse
- Compression ratio is acknowledged as not at fundamental limits since the assembly algorithm could still be improved

## Evidence

- [abstract] NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [readme] Compresses only the read sequences (i.e., ignores quality values and read identifiers).: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."
- [readme] Note that the compressed files are tar archives consisting of the different compressed streams: "Note that the compressed files are tar archives consisting of the different compressed streams"
- [readme] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."

---
name: lossless-compression-verification
description: Use when a NanoSpring-compressed nanopore FASTQ archive must be validated
  as lossless before reporting compression ratios, e.g. when reproducing Table 2 bits-per-base
  results or coverage-subsampled compression curves.
when_to_use_negative:
- The compression tool stores quality values or read identifiers (e.g. gzip on the
  raw FASTQ), so comparing only sed '2~4p' sequence lines would not establish full
  losslessness
- A lossy or reference-based assessment of compression quality is intended rather
  than a lossless round-trip check
- Input is not a FASTQ nanopore read file (NanoSpring requires only a FASTQ file as
  input and compresses only base sequences)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: NanoSpring
  role: Compresses and decompresses nanopore FASTQ read sequences; its -d mode regenerates
    file.reads used for the lossless comparison
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose baseline compressor whose FASTQ output is compared against
    NanoSpring ratios after independent lossless round-trips
- name: minimap2
  role: Aligns candidate reads to consensus inside NanoSpring's compression pipeline;
    correctness of its parameters (k 20, w 50, max-chain-iter 400) is validated by
    the round-trip check
provenance:
  source_task_ids:
  - task_001
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:fe92ca92db9e64dbd05ccc042e039a41a2085a40891aeabf9b75a8265a80c1c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  - minimap2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# lossless-compression-verification

## Summary

Verifies that a NanoSpring nanopore read compression round-trip is lossless by decompressing the .NanoSpring archive and byte-comparing the recovered sequences against the original FASTQ sequence lines. Because NanoSpring compresses only base sequences (not identifiers or quality values), verification must strip FASTQ to every 4th line before comparison.

## When to use

Use when a NanoSpring-compressed nanopore FASTQ archive must be validated as lossless before reporting compression ratios, e.g. when reproducing Table 2 bits-per-base results or coverage-subsampled compression curves.

## When NOT to use

- The compression tool stores quality values or read identifiers (e.g. gzip on the raw FASTQ), so comparing only sed '2~4p' sequence lines would not establish full losslessness
- A lossy or reference-based assessment of compression quality is intended rather than a lossless round-trip check
- Input is not a FASTQ nanopore read file (NanoSpring requires only a FASTQ file as input and compresses only base sequences)

## Inputs

- Nanopore FASTQ file (basecalled ONT reads; gzipped FASTQ also supported)
- NanoSpring compressed archive (.NanoSpring tar of compressed streams)

## Outputs

- Decompressed sequence-only file (file.reads)
- Pass/fail verdict from byte comparison (cmp) of decompressed reads vs FASTQ sequence lines
- Validated bits-per-base value (compressed bytes x 8 / total bases in FASTQ)

## How to apply

Compress the FASTQ with NanoSpring default parameters (20 threads, minhash kmer 23, 60 hash functions, overlap sketch threshold 6) via './NanoSpring -c -i file.fastq -o file.NanoSpring', then decompress with './NanoSpring -d -i file.NanoSpring -o file.reads -t 20'. Because NanoSpring stores only read sequences, extract the sequence lines of the original FASTQ using sed '2~4p' and run 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' to confirm exact byte identity. Record the size of the .NanoSpring tar archive (of compressed streams) and divide by total sequenced bases to compute bits-per-base only after losslessness is confirmed. Rationale: approximate-assembly compression must reconstruct reads exactly; the paper reports 0.35-0.65 bits per base for recently basecalled high-quality datasets, and any mismatch invalidates downstream ratio tables.

## Related tools

- **NanoSpring** (Compresses and decompresses nanopore FASTQ read sequences; its -d mode regenerates file.reads used for the lossless comparison) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose baseline compressor whose FASTQ output is compared against NanoSpring ratios after independent lossless round-trips)
- **minimap2** (Aligns candidate reads to consensus inside NanoSpring's compression pipeline; correctness of its parameters (k 20, w 50, max-chain-iter 400) is validated by the round-trip check)

## Examples

```
./NanoSpring -d -i file.NanoSpring -o file.reads -t 20 && cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' exits 0, confirming byte-identical sequence recovery
- Computed bits-per-base falls in the reported 0.35-0.65 range for recently basecalled high-quality nanopore datasets (e.g. below 0.7 on hs2/hs3, 0.35 on hs4)
- Decompression completes with the default thread count (e.g. -t 20) without errors from the .NanoSpring tar archive streams
- Coverage-subsampled CHM13 curve reproduces the reported 0.83/0.68/0.60/0.57/0.56 bits/base values at 12x/23x/46x/69x/126x
- Total sequenced bases extracted from the FASTQ match the denominator used in the bits-per-base normalization

## Limitations

- NanoSpring compresses only base sequences and does not compress read identifiers or quality values, so 'losslessness' applies to sequence lines only
- Compressed files are tar archives of different compressed streams; the archive byte size must be interpreted carefully when extracting sequence-only compressed size
- Decompression memory is approximate via --decompression-memory (default 5 GB); very low settings or many threads can make another stage the memory bottleneck
- Building on older architectures may require removing -msse4.1/-mavx2 flags or compiling minimap2 with sse2only=1

## Evidence

- [methods] Verify compression was lossless by comparing decompressed reads: "Compare decompressed file to reads in original file to verify compression was lossless."
- [discussion] NanoSpring compresses only base sequences: "NanoSpring, which focuses only on the base sequences in the FASTQ file"
- [abstract] 0.35-0.65 bits per base: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [readme] cmp command from README: "cmp file.reads <(cat file.fastq | sed -n '2~4p')"
- [readme] Compressed archives are tar files of streams: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"

---
name: lzma2-bsc-stream-compression
description: Use when encoding nanopore FASTQ reads into per-contig streams after
  approximate assembly and those streams must be packed into a lossless, high-ratio
  compressed archive.
when_to_use_negative:
- The goal is to compress read identifiers or quality values — NanoSpring only compresses
  base sequences in the FASTQ file and ignores the rest.
- The input is short, low-error sequencing reads (e.g., Illumina); existing compressors
  like SPRING or PgRC assume short reads and this assembly-based approach targets
  long, error-prone ONT reads.
- A general-purpose compressor is sufficient and ratio is not critical; gzip is simpler,
  though 3-6x worse on recent basecaller outputs.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: NanoSpring
  role: Owns the encoding and invokes the LZMA2/BSC stream compression stage; produces
    and consumes the .NanoSpring tar archive via -c and -d modes.
  repo: https://github.com/qm2/NanoSpring
- name: fast-lzma2
  role: LZMA2 codec used to compress the encoded streams, with a parallel buffered
    radix match-finder for multi-threaded, low-memory compression.
  repo: https://github.com/conor42/fast-lzma2
- name: libbsc
  role: Block-sorting (BSC) lossless compression library used to compress the encoded
    streams alongside LZMA2.
  repo: https://github.com/IlyaGrebnov/libbsc
- name: gzip
  role: Baseline general-purpose compressor against which the stream-compression result
    is benchmarked (NanoSpring is 3-6x lower bits per base).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:d8aefecf040f2a1debfcdd3a5330b5bcf6028c3880ad0540271d1d938db048e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - NanoSpring
  - fast-lzma2
  - gzip
  - libbsc
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# lzma2-bsc-stream-compression

## Summary

Compresses the ordered streams produced by NanoSpring's approximate-assembly encoding (consensus sequence, start/error position, error type, erroneous base, reverse-complement flag, read index, and lone reads) using LZMA2 (fast-lzma2) and BSC (libbsc), then bundles the compressed streams into a single tar archive with the .NanoSpring extension.

## When to use

Use when encoding nanopore FASTQ reads into per-contig streams after approximate assembly and those streams must be packed into a lossless, high-ratio compressed archive.

## When NOT to use

- The goal is to compress read identifiers or quality values — NanoSpring only compresses base sequences in the FASTQ file and ignores the rest.
- The input is short, low-error sequencing reads (e.g., Illumina); existing compressors like SPRING or PgRC assume short reads and this assembly-based approach targets long, error-prone ONT reads.
- A general-purpose compressor is sufficient and ratio is not critical; gzip is simpler, though 3-6x worse on recent basecaller outputs.

## Inputs

- Encoded NanoSpring streams per contig (consensus sequence, start/error position, error type, erroneous base, reverse-complement flag, read index)
- Lone-reads stream for unaligned reads
- Output path for the .NanoSpring tar archive

## Outputs

- Single tar archive of compressed streams with the .NanoSpring extension
- Losslessly decompressible read sequences (via ./NanoSpring -d) reproducing the original FASTQ base sequences

## How to apply

After NanoSpring encodes each contig's reads into the ordered streams (consensus sequence, start/error position, error type, erroneous base, reverse-complement flag, and 32-bit unsigned integer read index, with unaligned reads kept in a separate lone-reads stream), compress the encoded streams with the LZMA2 codec from the fast-lzma2 library (conor42/fast-lzma2) and the block-sorting compressor from libbsc (IlyaGrebnov/libbsc). Both libraries are built as part of the NanoSpring source build (git clone https://github.com/qm2/NanoSpring.git, then cmake/make in a build directory), so no separate invocation of fast-lzma2 or libbsc is needed. Bundle the individually compressed streams into a single tar archive and name it with the .NanoSpring extension as a convention. The rationale is that the separate streams have distinct statistical properties, so specialized codecs on each stream yield better ratios than generic compressors like gzip, which NanoSpring beats by 3-6x. Validate by decompressing with ./NanoSpring -d and checking losslessness via cmp against the original read sequences, and by confirming the total size falls near the reported 0.35-0.65 bits per base range for high-quality nanopore datasets.

## Related tools

- **NanoSpring** (Owns the encoding and invokes the LZMA2/BSC stream compression stage; produces and consumes the .NanoSpring tar archive via -c and -d modes.) — https://github.com/qm2/NanoSpring
- **fast-lzma2** (LZMA2 codec used to compress the encoded streams, with a parallel buffered radix match-finder for multi-threaded, low-memory compression.) — https://github.com/conor42/fast-lzma2
- **libbsc** (Block-sorting (BSC) lossless compression library used to compress the encoded streams alongside LZMA2.) — https://github.com/IlyaGrebnov/libbsc
- **gzip** (Baseline general-purpose compressor against which the stream-compression result is benchmarked (NanoSpring is 3-6x lower bits per base).)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring && ./NanoSpring -d -i file.NanoSpring -o file.reads && cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- The .NanoSpring tar archive is produced and contains the compressed streams.
- Roundtrip check passes: cmp file.reads <(cat file.fastq | sed -n '2~4p') reports no differences (decompressed reads match original FASTQ sequences exactly).
- Total bits per base, computed as (compressed archive size in bits) / (total number of bases), falls in the reported 0.35-0.65 range for recently basecalled high-quality nanopore datasets.
- Compression ratio is at least ~3x better than gzip on the same base sequences for recent basecaller versions.
- Decompression with 20 threads completes significantly faster than CoLoRd (paper reports >4x faster decompression).

## Limitations

- Only base sequences in FASTQ are compressed; read identifiers and quality values are not included in the streams and will be lost by the archive alone.
- Compression ratio is not at fundamental limits; the paper notes future work on the assembly algorithm to get closer to fundamental limits.
- The tar-based multi-stream archive has higher decompression memory usage (default --decompression-memory 5 GB, approximate, and may have no effect with many threads or very low settings).
- Older architectures may need to remove -msse4.1 and -mavx2 compile flags in CMakeLists.txt for the codecs to build.

## Evidence

- [methods] Compress the encoded streams with LZMA2 (fast-lzma2) and BSC (libbsc) and bundle them into a single tar archive: [evidence span withheld — anchor status: not_found]
- [readme] README tar archive note: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"
- [abstract] bits per base abstract finding: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [readme] README lossless verification: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] README scope statement: "Compresses only the read sequences (i.e., ignores quality values and read identifiers). Achieves over 3 times better compression ratios than Gzip"

---
name: bits-per-base-compression-ratio-calculation
description: Use when comparing read-sequence compression performance of FASTQ compressors
  (NanoSpring, Gzip, ENANO, CoLoRd) on nanopore datasets, or when reporting compressed
  sizes in bits per base.
when_to_use_negative:
- When compressing or measuring full FASTQ files including identifiers and quality
  scores — NanoSpring intentionally compresses only base sequences, so the metric
  would not be comparable
- When reads come from an older basecaller with error rates well above the ~5% range,
  where ratios outside the reported 0.35–0.65 bpb range are expected (e.g. fast-mode
  Guppy on S. aureus gives ~0.69 bpb)
- For reference-based compressors such as RENANO, which the paper evaluates separately
  and whose size accounting differs
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: NanoSpring
  role: Lossless reference-free compressor of nanopore read sequences; produces the
    .NanoSpring archive whose size in bits is divided by total bases
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose compressor used as the baseline for comparing bits-per-base
    ratios
- name: ENANO
  role: Comparison compressor focusing on quality score compression; its read-sequence
    ratio is extracted for the same normalization
- name: CoLoRd
  role: State-of-the-art overlap-graph compressor run in reference-free balanced mode
    for ratio comparison
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:2e6c650bc1e3508cd9bd1fec01fdbcef98190f5681527e0942f664836067d8b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - Guppy
  - NanoSpring
  - gzip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# bits-per-base-compression-ratio-calculation

## Summary

Compute compressed size in bits per base (bits/base, bpb) for nanopore FASTQ read sequences, the primary compression-ratio metric used by NanoSpring, by dividing the compressed archive size in bits by the total number of sequenced bases.

## When to use

Use when comparing read-sequence compression performance of FASTQ compressors (NanoSpring, Gzip, ENANO, CoLoRd) on nanopore datasets, or when reporting compressed sizes in bits per base.

## When NOT to use

- When compressing or measuring full FASTQ files including identifiers and quality scores — NanoSpring intentionally compresses only base sequences, so the metric would not be comparable
- When reads come from an older basecaller with error rates well above the ~5% range, where ratios outside the reported 0.35–0.65 bpb range are expected (e.g. fast-mode Guppy on S. aureus gives ~0.69 bpb)
- For reference-based compressors such as RENANO, which the paper evaluates separately and whose size accounting differs

## Inputs

- Original FASTQ file of nanopore reads (gzipped FASTQ also supported)
- Compressed archive from the compressor (e.g. .NanoSpring tar archive, gzip file, ENANO or CoLoRd output)

## Outputs

- Bits-per-base value (compressed size in bits divided by total number of sequenced bases)
- Tabulated per-dataset, per-compressor bits-per-base values (e.g. table2_bits_per_base.tsv or coverage_bits_per_base.csv)

## How to apply

Compress the input FASTQ with the tool under test (e.g. './NanoSpring -c -i file.fastq -o file.NanoSpring' with default 20 threads, kmer 23, 60 hash functions, overlap sketch threshold 6). Record the byte size of the compressed output — for NanoSpring this is a tar archive of compressed streams — and divide the total compressed bits by the total number of sequenced bases in the FASTQ. Count only base sequences, excluding read identifiers and quality scores, since NanoSpring does not compress those. Verify losslessness before trusting a ratio by decompressing ('./NanoSpring -d -i file.NanoSpring -o file.reads') and running 'cmp file.reads <(cat file.fastq | sed -n '2~4p')'. Expect recently basecalled high-quality nanopore datasets to fall in the 0.35–0.65 bits/base range for NanoSpring, roughly 3–6x below Gzip.

## Related tools

- **NanoSpring** (Lossless reference-free compressor of nanopore read sequences; produces the .NanoSpring archive whose size in bits is divided by total bases) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose compressor used as the baseline for comparing bits-per-base ratios)
- **ENANO** (Comparison compressor focusing on quality score compression; its read-sequence ratio is extracted for the same normalization)
- **CoLoRd** (State-of-the-art overlap-graph compressor run in reference-free balanced mode for ratio comparison)

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring && stat -c %s file.NanoSpring
```

## Evaluation signals

- Decompressed reads are byte-identical to the original FASTQ sequences ('cmp file.reads <(cat file.fastq | sed -n '2~4p')' reports no differences)
- Bits-per-base for NanoSpring falls in the reported 0.35–0.65 range on recently basecalled high-quality datasets
- Subsampled CHM13/hs3 values reproduce the reported trend of 0.83, 0.68, 0.60, 0.57 and 0.56 bits/base at 12x, 23x, 46x, 69x and 126x coverage
- Gzip baseline is roughly 2.2 bpb, confirming the reported 3–6x gap
- Normalization consistently uses total sequenced bases (sequence lines only), excluding identifiers and quality values from the denominator accounting

## Limitations

- Ratios depend strongly on basecaller quality: sup-mode Guppy (mean error rate below 4%) reaches ~0.41 bpb on S. aureus versus ~0.69 bpb for fast mode
- Compression improves with coverage; low-coverage subsamples (e.g. 12x CHM13 at 0.83 bpb) fall outside the headline 0.35–0.65 range
- NanoSpring archives are tar archives of multiple compressed streams (LZMA2/fast-lzma2 and BSC), so the archive size, not a single stream, must be used
- The compression ratio is not at fundamental limits; the approximate assembly algorithm has room for improvement per the paper's discussion

## Evidence

- [abstract] For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base: "For recently basecalled high quality nanopore datasets, NanoSpring, which focuses only on the base sequences in the FASTQ file, uses just 0.35-0.65 bits per base"
- [abstract] gzip 3-6x lower: "0.35-0.65 bits per base which is 3-6[Formula: see text] lower than general purpose compressors like gzip"
- [readme] lossless verification: "cmp file.reads <(cat file.fastq | sed -n '2~4p')"
- [readme] tar archive structure: "Note that the compressed files are tar archives consisting of the different compressed streams, although we recommend using the ``NanoSpring extension"
- [readme] only base sequences compressed: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."

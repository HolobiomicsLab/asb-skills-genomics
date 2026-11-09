---
name: lossless-roundtrip-verification
description: Use when validating that a compressed-then-decompressed nanopore FASTQ
  dataset is identical to the original reads, e.g. after benchmarking NanoSpring compression
  or reproducing compression/decompression timing tables.
when_to_use_negative:
- The compressor is intentionally lossy or only stores subset data (e.g., NanoSpring
  drops quality scores and read identifiers, so full-FASTQ diffing will always fail)
- Output of an assembly or polishing step where reconstructed reads are expected to
  differ from input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: NanoSpring
  role: Compressor/decompressor whose decompression output ('./NanoSpring -d -i file.NanoSpring
    -o file.reads') is compared to the original FASTQ reads for losslessness
  repo: https://github.com/qm2/NanoSpring
- name: gzip
  role: General-purpose compressor used as comparison tool; its decompressed output
    can be verified with the same roundtrip check
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:216a01a7051488a057c1cf6d34a5edf538bef24bd2d13abde899d50cdacca74f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - CoLoRd
  - ENANO
  - NanoSpring
  - gzip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# lossless-roundtrip-verification

## Summary

Verify that a genomic sequence compressor (e.g., NanoSpring) reconstructs data bit-for-bit by comparing the decompressed output against the original reads, as required to validate lossless FASTQ compression workflows.

## When to use

Use when validating that a compressed-then-decompressed nanopore FASTQ dataset is identical to the original reads, e.g. after benchmarking NanoSpring compression or reproducing compression/decompression timing tables.

## When NOT to use

- The compressor is intentionally lossy or only stores subset data (e.g., NanoSpring drops quality scores and read identifiers, so full-FASTQ diffing will always fail)
- Output of an assembly or polishing step where reconstructed reads are expected to differ from input

## Inputs

- Original nanopore sequencing FASTQ file (gzipped FASTQ also supported)
- Decompressed output file (e.g., file.reads) produced by './NanoSpring -d'
- Compressed archive (file.NanoSpring) if roundtrip is run end-to-end

## Outputs

- Pass/fail signal from cmp (no differences = lossless roundtrip)
- Validation status per dataset for benchmark tables (Tables 3-4)

## How to apply

After decompressing with './NanoSpring -d -i file.NanoSpring -o file.reads', strip the FASTQ to its sequence lines only, since NanoSpring compresses only base sequences and ignores read identifiers and quality values. Run 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' so the decompressed reads are compared directly against every 4th line (the sequence lines) of the original FASTQ; a silent exit with no differences confirms losslessness. This check is the validation gate for benchmark workflows (Tables 3-4): a dataset's timing/memory measurement is only accepted if the roundtrip comparison passes. The same pattern generalizes to any lossless compressor benchmark where the tool guarantees exact sequence recovery but not header/quality recovery.

## Related tools

- **NanoSpring** (Compressor/decompressor whose decompression output ('./NanoSpring -d -i file.NanoSpring -o file.reads') is compared to the original FASTQ reads for losslessness) — https://github.com/qm2/NanoSpring
- **gzip** (General-purpose compressor used as comparison tool; its decompressed output can be verified with the same roundtrip check)

## Examples

```
cmp file.reads <(cat file.fastq | sed -n '2~4p')
```

## Evaluation signals

- 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' exits silently (byte-identical sequence lines)
- Roundtrip passes for every dataset before accepting timing/memory values in benchmark tables (Tables 3-4)
- Compressed archive exists and decompression completes with the requested thread count (e.g., -t 20)
- Bits-per-base computed as archive size / total bases falls in the reported 0.35-0.65 range for high-quality nanopore datasets
- Test installation passes the repository test script ('./../util/test_script.sh') before benchmarking

## Limitations

- NanoSpring compresses only base sequences, not read identifiers or quality values, so comparison must be restricted to sequence lines (every 4th line of FASTQ)
- Building on older architectures may require removing -msse4.1/-mavx2 flags or building minimap2 with sse2only, or the executable may not run
- The repo test script only validates installation, not losslessness on your specific data; the cmp check must still be run per dataset

## Evidence

- [methods] Compare decompressed file to reads in original file to verify compression was lossless.: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] cmp file.reads <(cat file.fastq | sed -n '2~4p'): "cmp file.reads <(cat file.fastq | sed -n '2~4p')"
- [discussion] NanoSpring is open-source and available on GitHub: "NanoSpring, which focuses only on the base sequences in the FASTQ file"
- [readme] Compresses only the read sequences (i.e., ignores quality values and read identifiers).: "Compresses only the read sequences (i.e., ignores quality values and read identifiers). Achieves over 3 times better compression ratios than Gzip"

---
name: ont-signal-basecalling
description: Use when raw ONT signal data must be converted to basecalled FASTQ and
  you need to measure how Guppy basecaller mode (fast, hac, sup) affects read error
  rate and subsequent NanoSpring compression performance.
when_to_use_negative:
- Input is already a basecalled FASTQ with no raw signal available, so Guppy modes
  cannot be varied
- The goal requires compressed output that preserves read identifiers or quality values
  — NanoSpring explicitly does not compress them
- Basecaller error rates are far above the ~5% regime the approximate-assembly approach
  relies on (e.g. very old basecallers), where NanoSpring's compression gains are
  not expected
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Guppy
  role: Basecalls the ONT raw signal dataset into FASTQ, run once per mode (fast,
    hac, sup) with version 6.1.2
- name: NanoSpring
  role: Compresses each basecalled FASTQ (default 20 threads) to measure bits/base
    per basecaller mode and provides lossless decompression for verification
  repo: https://github.com/qm2/NanoSpring
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-023-29267-8
    title: Reference-free lossless compression of nanopore sequencing reads using
      an approximate assembly approach
  source_content_hash: sha256:3319071df290d87526d2fd3e5f6bea4355a1bc5cce3756d1b08a6d9a3bdf5e17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qm2/NanoSpring
  tools:
  - Guppy
  - NanoSpring
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ont-signal-basecalling

## Summary

Basecalls Oxford Nanopore raw signal data into FASTQ with Guppy in fast, high-accuracy (hac), and super-accurate (sup) modes, and quantifies how basecaller mode changes mean error rate and downstream NanoSpring compression (bits/base) on datasets such as S. aureus.

## When to use

Use when raw ONT signal data must be converted to basecalled FASTQ and you need to measure how Guppy basecaller mode (fast, hac, sup) affects read error rate and subsequent NanoSpring compression performance.

## When NOT to use

- Input is already a basecalled FASTQ with no raw signal available, so Guppy modes cannot be varied
- The goal requires compressed output that preserves read identifiers or quality values — NanoSpring explicitly does not compress them
- Basecaller error rates are far above the ~5% regime the approximate-assembly approach relies on (e.g. very old basecallers), where NanoSpring's compression gains are not expected

## Inputs

- Oxford Nanopore raw signal dataset (e.g. S. aureus FAST5/pod5 reads)
- Guppy 6.1.2 basecaller with fast, hac and sup models
- NanoSpring compressor executable (conda 'nanospring' or built from source)

## Outputs

- One FASTQ per Guppy mode (fast, hac, sup)
- Per-mode metrics: mean error rate, compressed size in bits/base, compression time, peak memory
- A table mirroring Table 5 (table5_s_aureus.csv) with one row per Guppy mode
- .NanoSpring archives and .reads decompressed files for lossless verification

## How to apply

Basecall the raw signal dataset three times with Guppy 6.1.2 using the fast, hac and sup modes, yielding one FASTQ per mode. For each basecalled set, compress the read sequences with NanoSpring in compress mode using the documented pattern ('NanoSpring -c -i input.fastq -o output.NanoSpring') with the default 20 threads, since NanoSpring uses only the base sequences and ignores identifiers and quality values. Measure mean error rate per mode (the paper notes sup mode has a mean error rate below 4%, consistent with deep-learning basecallers reaching ~5% or better), compressed size in bits per base, compression time, and peak memory. Assemble these into a per-mode table (e.g. table5_s_aureus.csv) mirroring Table 5, with expected bits/base near 0.69 (fast), 0.48 (hac) and 0.41 (sup). Finally verify losslessness by decompressing each archive ('NanoSpring -d -i file.NanoSpring -o file.reads') and comparing the recovered reads to the original FASTQ reads. The rationale is that falling basecaller error rates are precisely what makes NanoSpring's approximate-assembly compression effective, so error rate is the key explanatory variable.

## Related tools

- **Guppy** (Basecalls the ONT raw signal dataset into FASTQ, run once per mode (fast, hac, sup) with version 6.1.2)
- **NanoSpring** (Compresses each basecalled FASTQ (default 20 threads) to measure bits/base per basecaller mode and provides lossless decompression for verification) — https://github.com/qm2/NanoSpring

## Examples

```
./NanoSpring -c -i file.fastq -o file.NanoSpring
```

## Evaluation signals

- Reproduced bits per base are close to the paper's reported 0.69 (fast), 0.48 (hac) and 0.41 (sup) on S. aureus
- The sup-mode basecalled read set has a mean error rate below 4%
- Losslessness verified by 'cmp file.reads <(cat file.fastq | sed -n '2~4p')' or equivalent read-by-read comparison
- One FASTQ and one .NanoSpring archive exist per Guppy mode, produced with the default 20 threads
- The resulting table has exactly three rows (fast, hac, sup) with columns for mean error rate, bits/base, compression time and peak memory

## Limitations

- NanoSpring focuses only on the base sequences in the FASTQ file and does not compress read identifiers or quality values, so full-FASTQ size metrics will differ from bits/base
- Bits/base values depend on the basecaller version and its error rate; older architectures may require removing -msse4.1/-mavx2 flags from CMakeLists.txt to build NanoSpring
- Compression results are dataset-dependent (S. aureus values here; the paper reports 0.35–0.65 bits per base for recently basecalled high-quality nanopore datasets)

## Evidence

- [intro] Falling error rates make approximate assembly viable: "the error rate has fallen dramatically in the recent years with the advent of deep learning based basecallers which achieve median error rate close to 5% or better5"
- [other] Sup mode error rate and compression outcome: [evidence span withheld — anchor status: not_found]
- [other] Expected compression values per mode: [evidence span withheld — anchor status: not_found]
- [readme] Lossless verification method: "Compare decompressed file to reads in original file to verify compression was lossless."
- [readme] NanoSpring scope: sequences only: "Compresses only the read sequences (i.e., ignores quality values and read identifiers)."

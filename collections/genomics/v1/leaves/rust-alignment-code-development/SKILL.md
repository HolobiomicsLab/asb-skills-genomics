---
name: rust-alignment-code-development
description: 'Use when reproducing or extending Barbell''s pattern-aware demultiplexing
  scoring in Rust: you need to compute subsequence-based barcode scores from edit-distance
  CIGARs of Nanopore barcode/flank alignments and apply the default acceptance thresholds.'
when_to_use_negative:
- You only need to run demultiplexing as-is — use `barbell kit`/`annotate` presets
  instead of reimplementing the scoring
- Input reads lack barcode/flank structure at read ends (no rapid or native barcoding
  construct), so a flank-aware edit-distance score is meaningless
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Reference implementation whose subsequence scoring scheme (default >=20% perfect-score
    and >=10% separation cutoffs) is being reconstructed in Rust
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching library providing the edit-distance
    alignment (CIGAR) used to derive the subsequence score
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: Rust build/install tool; used with RUSTFLAGS='-C target-cpu=native' to enable
    SIMD fast paths
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:eab069f9c5d0940f96af8b8db54c059e4533d646d2ea87ad98088d8e69f1f8b6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Rust
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# rust-alignment-code-development

## Summary

Implement and validate Barbell's subsequence-based barcode scoring scheme in Rust using Sassy edit-distance alignment: score each read's barcode region from the alignment CIGAR (rewarding long contiguous exact-match runs), normalize against the perfect-match score, and accept barcodes at >=20% of perfect score with >=10% separation from the second-best.

## When to use

Use when reproducing or extending Barbell's pattern-aware demultiplexing scoring in Rust: you need to compute subsequence-based barcode scores from edit-distance CIGARs of Nanopore barcode/flank alignments and apply the default acceptance thresholds.

## When NOT to use

- You only need to run demultiplexing as-is — use `barbell kit`/`annotate` presets instead of reimplementing the scoring
- Input reads lack barcode/flank structure at read ends (no rapid or native barcoding construct), so a flank-aware edit-distance score is meaningless

## Inputs

- Synthetic Nanopore-style FASTQ reads with known barcodes/flanks embedded at read ends
- Query FASTA with shared left flank + barcode + shared right flank, one header per barcode
- Per-element alignment cost records (flank edits, barcode edits) from Sassy/Barbell edit-distance machinery

## Outputs

- Per-read TSV with read_id, computed score, normalized score, assigned barcode label, and accepted/rejected status
- Subsequence scoring function derived from alignment CIGARs, normalized against perfect-match score

## How to apply

Generate synthetic Nanopore-style FASTQ reads with known barcode and shared left/right flank constructs at read ends, one header per barcode in the query FASTA. Use the Sassy edit-distance machinery (as Barbell does) to align query constructs to read ends, recording flank edits and barcode edits separately as in Barbell's annotate output columns (flank_cost, barcode_cost). Derive the subsequence score from each match's CIGAR by rewarding long contiguous exact-match runs and penalizing interleaved match/error patterns, then normalize by the score of a perfect match to the expected construct. Apply the default acceptance rule: normalized score >=20% of the perfect score AND >=10% separation from the second-best barcode; classify each read as accepted/rejected and record read_id, score, normalized score, assigned label, and status in a TSV. Build with `RUSTFLAGS="-C target-cpu=native" cargo install barbell` conventions so SIMD (AVX2/NEON via Sassy) is actually used, and validate that every accepted read carries exactly the barcode embedded during synthesis while rejected reads fall below the threshold.

## Related tools

- **Barbell** (Reference implementation whose subsequence scoring scheme (default >=20% perfect-score and >=10% separation cutoffs) is being reconstructed in Rust) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching library providing the edit-distance alignment (CIGAR) used to derive the subsequence score) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Rust build/install tool; used with RUSTFLAGS='-C target-cpu=native' to enable SIMD fast paths)

## Examples

```
RUSTFLAGS="-C target-cpu=native" cargo install barbell
```

## Evaluation signals

- Every read accepted at the >=20% normalized score with >=10% separation is assigned exactly the barcode embedded during synthetic read generation
- All rejected reads have normalized scores below the 20%-of-perfect acceptance threshold
- No assignment contradicts the ground-truth synthetic labels
- Per-read TSV contains read_id, computed score, normalized score, assigned label, and accepted/rejected status for every read
- Compiled binary uses SIMD fast paths (verify with ensure_simd-style AVX2/NEON compile-time checks and target-cpu=native builds)

## Limitations

- Barbell's scoring details require consulting the paper; README notes the git version (Sassy V2) is ~5x faster and 'results can slightly differ (<0.01% or so)' vs releases
- Too many Fflank matches indicate spurious/random alignments; flank error thresholds (--flank-max-errors) may need adjustment, and defaults are otherwise lenient
- Sassy is intended for patterns up to ~1000 nt; very long query constructs may fall outside its tested range

## Evidence

- [methods] Barbell is written in Rust.: "Barbell is written in Rust."
- [methods] RUSTFLAGS="-C target-cpu=native" cargo install barbell: "RUSTFLAGS="-C target-cpu=native" cargo install barbell"
- [readme] Sassy is a library and tool for searching short strings in texts,: [evidence span withheld — anchor status: not_found]
- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [methods] Current git version is around 5x faster than the releases by using [Sassy V2]: "Current git version is around 5x faster than the releases by using [Sassy V2](https://github.com/RagnarGrootKoerkamp/sassy)"

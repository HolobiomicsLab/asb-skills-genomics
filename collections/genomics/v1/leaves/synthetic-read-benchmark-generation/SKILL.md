---
name: synthetic-read-benchmark-generation
description: 'Use when you need ground-truth demultiplexing benchmarks: synthetic
  FASTQ reads with embedded barcodes at read ends to test scoring, acceptance thresholds,
  or trimming/demux correctness.'
when_to_use_negative:
- Input is already demultiplexed or reads lack embedded barcode/flank constructs at
  read ends
- The goal is kit-level demultiplexing of real data with presets (use `barbell kit`
  instead of a custom synthetic benchmark)
edam_operation: http://edamontology.org/operation_0252
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: Pattern-aware demultiplexer whose annotate/scoring output (flank_cost, barcode_cost,
    CIGAR-derived scores) is exercised and validated by the synthetic benchmark
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string-matching library underlying Barbell's
    edit-distance computation used to derive alignment costs and scores
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: Build tool for installing Barbell from source with native SIMD flags (RUSTFLAGS="-C
    target-cpu=native" cargo install barbell)
  repo: https://github.com/rickbeeloo/barbell
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:9f71d6d3b70d3bcfe6734e0934e7b1a413154830b9f5bfd29a83eb58110e25af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0252
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Synthetic read benchmark generation

## Summary

Generates synthetic Nanopore-style FASTQ reads with known barcode/flank constructs embedded at read ends so that Barbell's subsequence-based barcode scoring and acceptance thresholds (score >=20% of perfect, >=10% separation from second-best) can be validated against ground-truth labels.

## When to use

Use when you need ground-truth demultiplexing benchmarks: synthetic FASTQ reads with embedded barcodes at read ends to test scoring, acceptance thresholds, or trimming/demux correctness.

## When NOT to use

- Input is already demultiplexed or reads lack embedded barcode/flank constructs at read ends
- The goal is kit-level demultiplexing of real data with presets (use `barbell kit` instead of a custom synthetic benchmark)

## Inputs

- Set of known barcode sequences (as a query FASTA with shared left flank + barcode + shared right flank per header)
- Barcode/flank construct definitions for synthetic read generation
- Nanopore-style FASTQ reads (synthetic or real) to be scored

## Outputs

- Synthetic Nanopore-style FASTQ reads with ground-truth embedded barcode labels
- Per-read TSV table with read_id, computed score, normalized score, assigned label, and accepted/rejected status
- Validation report comparing assigned labels to ground-truth synthetic labels

## How to apply

Generate synthetic Nanopore-style FASTQ reads where each read carries a known construct (shared left flank + barcode + shared right flank) at read ends, with one query FASTA header per barcode. Compute per-element alignment costs using the Barbell/Sassy edit-distance machinery (flank edits and barcode edits recorded separately, as in the annotate TSV output). Derive the subsequence score from each match's CIGAR by rewarding long contiguous exact-match runs and penalizing interleaved match/error patterns, then normalize by the score of a perfect match to the expected construct. Apply the default acceptance thresholds — normalized score >=20% of the perfect score and >=10% score separation from the second-best barcode — to classify each read as accepted or rejected, then write a per-read TSV with read_id, computed score, normalized score, assigned label, and accepted/rejected status.

## Related tools

- **Barbell** (Pattern-aware demultiplexer whose annotate/scoring output (flank_cost, barcode_cost, CIGAR-derived scores) is exercised and validated by the synthetic benchmark) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string-matching library underlying Barbell's edit-distance computation used to derive alignment costs and scores) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Build tool for installing Barbell from source with native SIMD flags (RUSTFLAGS="-C target-cpu=native" cargo install barbell)) — https://github.com/rickbeeloo/barbell

## Examples

```
RUSTFLAGS="-C target-cpu=native" cargo install barbell && barbell annotate --kit SQK-RBK114-96 -i synthetic_reads.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- Every accepted read is assigned exactly the barcode that was embedded during synthesis
- Rejected reads carry normalized scores below the 20%-of-perfect acceptance threshold
- No assignment contradicts the ground-truth synthetic labels
- Recorded flank_cost and barcode_cost values match the edit distances between read ends and the expected flank/barcode constructs
- Accepted reads show >=10% score separation from the second-best barcode

## Limitations

- Barbell's subsequence scoring rewards contiguous exact-match runs, so constructs with heavily interleaved match/error patterns may score below threshold even with the correct barcode
- Release (conda/cargo) and GitHub versions can give slightly different results (<0.01%) due to Sassy V2 speedups; use a consistent version when benchmarking
- Random matches can appear as `Fflank` match types if the flank error threshold (--flank-max-errors) is set too leniently; inspect match locations before trusting assignments

## Evidence

- [methods] Barbell scores each read's barcode region using a subsequence score derived from an edit-distance alignment CIGAR that rewards long contiguous exact-match runs: [evidence span withheld — anchor status: not_found]
- [readme] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [methods] Current git version is around 5x faster than the releases by using [Sassy V2]: "Current git version is around 5x faster than the releases by using [Sassy V2]"
- [readme] Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported): "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)"

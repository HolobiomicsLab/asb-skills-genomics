---
name: cigar-subsequence-scoring-implementation
description: Use when implementing or validating barcode-scoring logic for Nanopore
  demultiplexing, or when reproducing how Barbell converts a Sassy edit-distance CIGAR
  into a normalized per-read barcode score with acceptance thresholds.
when_to_use_negative:
- Input is raw genomic data without barcode/flank constructs — the scoring scheme
  operates on barcode regions matched against known constructs
- The goal is production demultiplexing of a supported kit — use `barbell kit` directly
  rather than re-implementing the scorer
- Data lack ground-truth barcode labels, so the accepted-read-vs-embedded-barcode
  validation cannot be performed
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Source implementation being reconstructed; provides the subsequence scoring
    scheme, annotate output format, and default thresholds
  repo: github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated edit-distance search engine whose alignment CIGAR feeds the
    subsequence score computation
  repo: github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: Build/install Rust code implementing and validating the scoring scheme (`RUSTFLAGS="-C
    target-cpu=native" cargo install barbell`)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:5abfde1d5db4461b115ab8fef6faf8d0d1bab48933a9c560c7842aa8fc28f41c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# CIGAR subsequence-based barcode scoring implementation

## Summary

Reconstructs Barbell's subsequence-based barcode scoring scheme: a score derived from the edit-distance alignment CIGAR that rewards long contiguous exact-match runs, normalized against a perfect-match score, with default acceptance at ≥20% of the perfect score and ≥10% separation from the second-best barcode.

## When to use

Use when implementing or validating barcode-scoring logic for Nanopore demultiplexing, or when reproducing how Barbell converts a Sassy edit-distance CIGAR into a normalized per-read barcode score with acceptance thresholds.

## When NOT to use

- Input is raw genomic data without barcode/flank constructs — the scoring scheme operates on barcode regions matched against known constructs
- The goal is production demultiplexing of a supported kit — use `barbell kit` directly rather than re-implementing the scorer
- Data lack ground-truth barcode labels, so the accepted-read-vs-embedded-barcode validation cannot be performed

## Inputs

- Synthetic Nanopore-style FASTQ reads with known barcode and flank constructs embedded at read ends (ground-truth labels)
- Query FASTA of barcode constructs (shared left flank + barcode + shared right flank, one header per barcode)
- Optional: Barbell annotate TSV (read_id, read_len, flank_cost, barcode_cost, label, cuts, etc.) as input for score re-derivation

## Outputs

- Per-read TSV with read_id, computed subsequence score, normalized score, assigned barcode label, and accepted/rejected status

## How to apply

Generate synthetic Nanopore-style FASTQ reads with known barcode/flank constructs embedded at the read ends (query FASTA layout: shared left flank + barcode + shared right flank, one header per barcode). Align each read's barcode region using the Barbell/Sassy edit-distance machinery, recording flank edits and barcode edits separately as in the annotate TSV output (columns include flank_cost and barcode_cost). Derive the subsequence score from the CIGAR of each match by rewarding long contiguous exact-match runs and penalizing interleaved match/error patterns, then normalize by the score of a perfect match to the expected construct. Apply the default acceptance thresholds — normalized score ≥20% of the perfect score with ≥10% separation from the second-best barcode — to classify each read as accepted or rejected, and write a per-read TSV with read_id, computed score, normalized score, assigned label, and accepted/rejected status.

## Related tools

- **Barbell** (Source implementation being reconstructed; provides the subsequence scoring scheme, annotate output format, and default thresholds) — github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated edit-distance search engine whose alignment CIGAR feeds the subsequence score computation) — github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Build/install Rust code implementing and validating the scoring scheme (`RUSTFLAGS="-C target-cpu=native" cargo install barbell`))

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- Every read classified as accepted is assigned the exact barcode that was embedded during synthetic read construction
- All rejected reads carry normalized scores below the 20%-of-perfect-score acceptance threshold
- No assignment contradicts the ground-truth synthetic labels in the read headers
- Normalized scores are ≤ 1 by construction (perfect match normalizes to 1.0), and accepted reads show ≥10% score separation from the second-best barcode
- Per-read flank_cost and barcode_cost values computed via the Sassy edit-distance path reproduce the columns reported in Barbell's annotate TSV

## Limitations

- Scores depend on the Sassy version: the README notes the git version 'results can slightly differ (<0.01% or so)' from releases due to Sassy V2 speedups
- Random or spurious flank matches can appear if flank error thresholds are relaxed too far (showing up as Fflank matches at random locations), contaminating the score distribution
- The scoring details are documented primarily in the paper; the README only points to it, so full parameter provenance requires the Bioinformatics publication

## Evidence

- [methods] Barbell scores each read's barcode region using a subsequence score derived from an edit-distance alignment CIGAR that rewards long contiguous exact-match runs: [evidence span withheld — anchor status: not_found]
- [results] scores are normalized relative to the perfect-match score, and by default a barcode is accepted if its score is ≥20% of the perfect score with ≥10% separation: [evidence span withheld — anchor status: not_found]
- [readme] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [readme] See the paper for more details on the Barbell scoring and comparisons with other tools: "See the paper for more details on the Barbell scoring and comparisons with other tools"
- [methods] Current git version is around 5x faster than the releases by using [Sassy V2]: "Current git version is around 5x faster than the releases by using [Sassy V2]"

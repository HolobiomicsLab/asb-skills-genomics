---
name: barcode-edit-distance-scoring
description: Use when demultiplexing Nanopore FASTQ reads with Barbell and you need
  to understand, reproduce, or tune how barcode/flank edit-distance costs are converted
  into accept/reject decisions during annotation.
when_to_use_negative:
- The goal is basecalling or alignment rather than barcode match scoring — Barbell
  scores barcode/flank edits only (Dorado covers basecalling/modification calling).
- Reads contain no barcode constructs or flanking adapter sequences to align against
  (random Fflank matches should not be scored as barcodes).
- Only raw kit barcodes with default settings suffice and scoring internals are irrelevant
  — 'barbell kit' wraps the cutoffs without exposing them.
edam_operation: http://edamontology.org/operation_3220
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Barbell
  role: Computes edit-distance matches of barcodes/flanks per read and applies the
    score thresholds to classify and label reads during annotate
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching engine underlying Barbell's edit-distance
    computation; Sassy V2 makes the git version ~5x faster
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: Dorado
  role: Baseline demultiplexer whose trimming errors the edit-distance scoring scheme
    is benchmarked against
  repo: https://github.com/nanoporetech/dorado
- name: Rust/cargo
  role: Implementation language; Barbell is built with RUSTFLAGS="-C target-cpu=native"
    for SIMD performance
provenance:
  source_task_ids:
  - task_005
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:bc0213f38ac81b66e2917e44d5fd8a62d368b397d0007364471a91c07cb0274b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Rust/cargo
  - Sassy
  - cargo
  edam_operation: http://edamontology.org/operation_3220
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# barcode-edit-distance-scoring

## Summary

Reconstructs Barbell's subsequence-based barcode scoring scheme, in which each read's barcode match (an edit-distance alignment produced with the Sassy engine) is scored by rewarding long contiguous exact-match runs, normalized against the perfect-match score, and accepted under default thresholds of >=20% of the perfect score and >=10% separation from the second-best barcode.

## When to use

Use when demultiplexing Nanopore FASTQ reads with Barbell and you need to understand, reproduce, or tune how barcode/flank edit-distance costs are converted into accept/reject decisions during annotation.

## When NOT to use

- The goal is basecalling or alignment rather than barcode match scoring — Barbell scores barcode/flank edits only (Dorado covers basecalling/modification calling).
- Reads contain no barcode constructs or flanking adapter sequences to align against (random Fflank matches should not be scored as barcodes).
- Only raw kit barcodes with default settings suffice and scoring internals are irrelevant — 'barbell kit' wraps the cutoffs without exposing them.

## Inputs

- Nanopore FASTQ reads (e.g. rapid-barcoding SQK-RBK114-96 or native SQK-NBD114-96)
- Barbell kit preset or custom query FASTA with shared left flank + barcode + shared right flank entries
- Kit-derived or manual flank error cutoff (--flank-max-errors)

## Outputs

- Annotation TSV (anno.tsv) with read_id, read_len, bar/flank positions, match_type, flank_cost, barcode_cost, label, strand, cuts
- Per-read accept/reject classification with computed score, normalized score, and assigned barcode label

## How to apply

Run 'barbell annotate' (backed by the Sassy SIMD-accelerated edit-distance engine) to locate barcode/flank matches per read and record flank edits and barcode edits separately in the annotation TSV (flank_cost, barcode_cost columns). Derive a subsequence score from each alignment's CIGAR, rewarding long contiguous exact-match runs and penalizing interleaved match/error patterns, then normalize by the score of a perfect match to the expected barcode+flank construct. Accept a barcode when its normalized score is >=20% of the perfect score and it is >=10% separated from the second-best barcode; '--min-score-diff' controls the separation and '--flank-max-errors' controls the flank edit-distance cutoff (a kit-derived cutoff is reported automatically, e.g. relax to --flank-max-errors 5 if too few reads are annotated). Validate by scoring synthetic reads with known embedded barcodes and confirming every accepted read is assigned the synthesized label while rejected reads fall below threshold.

## Related tools

- **Barbell** (Computes edit-distance matches of barcodes/flanks per read and applies the score thresholds to classify and label reads during annotate) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching engine underlying Barbell's edit-distance computation; Sassy V2 makes the git version ~5x faster) — https://github.com/RagnarGrootKoerkamp/sassy
- **Dorado** (Baseline demultiplexer whose trimming errors the edit-distance scoring scheme is benchmarked against) — https://github.com/nanoporetech/dorado
- **Rust/cargo** (Implementation language; Barbell is built with RUSTFLAGS="-C target-cpu=native" for SIMD performance)

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- On synthetic FASTQ with embedded barcodes, every accepted read is assigned the ground-truth synthesized barcode and rejected reads score below the 20%-of-perfect threshold.
- Accepted barcodes exhibit >=10% normalized-score separation from the second-best barcode.
- anno.tsv rows show separate flank_cost and barcode_cost columns with plausible edit distances (e.g. ~3 for an intact barcode+flank, ~7 for a fused second barcode).
- Tuning --flank-max-errors changes the number of annotated reads; a lower value should suppress random Fflank matches at non-read-end positions.
- Schema check: annotation TSV contains the documented columns (read_id, bar_start/bar_end, match_type, flank_cost, barcode_cost, label, strand, cuts).

## Limitations

- Thresholds interact: relaxing --flank-max-errors can admit random flank matches, while adjusting --min-score-diff is 'usually unnecessary as defaults are lenient' but may be needed for non-random match locations.
- Only rapid (SQK-RBK114-96) and native (SQK-NBD114-96) kits were thoroughly tested; other kits may show unexpected annotations.
- Results can slightly differ (<0.01%) between release and git versions because Sassy V2 changes the underlying engine; Sassy v1.4 is needed for Mac support.
- Fused double-barcode configurations show higher second-barcode edit distance (mean ~7 vs ~3), which can push scores near the acceptance boundary unless --use-extended is enabled.

## Evidence

- [abstract] Barbell reports edit-distance matches of barcodes and flanks per read: "We developed Barbell, a pattern-aware demultiplexer that is designed to detect complex barcode configurations."
- [methods] Annotate step produces the per-read scoring table: "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [readme] Flank cutoff is kit-derived and tunable: "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)."
- [readme] Default min-score-diff guidance: "This is usually unnecessary as defaults are lenient; report an issue if it occurs."
- [discussion] Sassy engine and version effects on scoring performance: "Use new version of Sassy with Mac support (v1.4)"

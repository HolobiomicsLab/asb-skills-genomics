---
name: nanopore-barcode-assignment-threshold-tuning
description: 'Use when demultiplexing Nanopore FASTQ reads with Barbell and you need
  to reproduce, verify, or tune barcode acceptance scoring: e.g., validating that
  reads are assigned their synthetically embedded barcode under the >=20%-of-perfect-score
  and >=10%-separation default thresholds.'
when_to_use_negative:
- Input reads carry no barcode constructs and the goal is basecalling or assembly
  rather than demultiplexing.
- An annotation TSV already exists and the task is only pattern summarization or trimming
  (`barbell inspect` / `barbell trim`), not threshold reconstruction or validation.
- Input is RNA or Twist Nanopore kit data, which the README notes are exceptions to
  the built-in kit presets.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: Pattern-aware demultiplexer; `annotate` produces the per-read match table
    (flank/barcode costs) from which subsequence scores and acceptance thresholds
    are derived
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching library providing the edit-distance
    alignment used by Barbell's scoring; Sassy V2 makes the current git version ~5x
    faster
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: cargo
  role: 'Build/install Barbell from source with native CPU flags: RUSTFLAGS="-C target-cpu=native"
    cargo install barbell'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:21d35454b554d9e2fc57970b4c320b618f7f19b95f35d855ba653e9c1e031a98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Sassy
  - cargo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# nanopore-barcode-assignment-threshold-tuning

## Summary

Reconstructs and validates Barbell's subsequence-based barcode scoring scheme, which derives per-read scores from edit-distance alignment CIGARs rewarding long contiguous exact-match runs, and applies the default acceptance thresholds (score >=20% of the perfect-match score with >=10% separation from the second-best barcode) to demultiplex Nanopore reads.

## When to use

Use when demultiplexing Nanopore FASTQ reads with Barbell and you need to reproduce, verify, or tune barcode acceptance scoring: e.g., validating that reads are assigned their synthetically embedded barcode under the >=20%-of-perfect-score and >=10%-separation default thresholds.

## When NOT to use

- Input reads carry no barcode constructs and the goal is basecalling or assembly rather than demultiplexing.
- An annotation TSV already exists and the task is only pattern summarization or trimming (`barbell inspect` / `barbell trim`), not threshold reconstruction or validation.
- Input is RNA or Twist Nanopore kit data, which the README notes are exceptions to the built-in kit presets.

## Inputs

- Nanopore-style FASTQ reads with known barcode/flank constructs embedded at read ends
- Query FASTA with one entry per barcode: shared left flank + barcode + shared right flank
- Barbell annotate TSV with per-read flank_cost and barcode_cost columns

## Outputs

- Per-read TSV with read_id, computed subsequence score, normalized score, assigned label, and accepted/rejected status
- Classification of each read as accepted (assigned to the embedded barcode) or rejected (score below the acceptance threshold)

## How to apply

Generate synthetic Nanopore-style FASTQ reads with known barcode/flank constructs embedded at read ends, and build a query FASTA whose entries are shared left flank + barcode + shared right flank (one header per barcode). Run `barbell annotate` (backed by the Sassy edit-distance machinery) to obtain a per-read TSV recording flank edits and barcode edits separately in the flank_cost and barcode_cost columns. Derive the subsequence score from each match's CIGAR by rewarding long contiguous exact-match runs and penalizing interleaved match/error patterns, then normalize by the score of a perfect match to the expected construct. Classify each read as accepted (normalized score >=20% of perfect, >=10% separation from the second-best barcode; related CLI knob `--min-score-diff`) or rejected, and validate against the synthetic ground-truth labels.

## Related tools

- **Barbell** (Pattern-aware demultiplexer; `annotate` produces the per-read match table (flank/barcode costs) from which subsequence scores and acceptance thresholds are derived) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching library providing the edit-distance alignment used by Barbell's scoring; Sassy V2 makes the current git version ~5x faster) — https://github.com/RagnarGrootKoerkamp/sassy
- **cargo** (Build/install Barbell from source with native CPU flags: RUSTFLAGS="-C target-cpu=native" cargo install barbell)

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv
```

## Evaluation signals

- Every accepted read is assigned the barcode that was embedded during synthesis (no assignment contradicts ground-truth synthetic labels).
- All rejected reads carry normalized subsequence scores below the acceptance threshold (<20% of the perfect-match score).
- Accepted reads show >=10% score separation from the second-best barcode (adjustable via --min-score-diff).
- flank edits and barcode edits in the computed table match the separate flank_cost and barcode_cost columns of the Barbell annotate output schema.
- Flank-based edit-distance cutoff is consistent with the conservative default; too many Fflank matches at random locations indicates the threshold was relaxed too far.

## Limitations

- The current git version uses Sassy V2 and results can slightly differ (<0.01% or so) from released versions.
- If --flank-max-errors is relaxed too much, random matches appear as Fflank at unexpected locations; if annotate misses many reads, the flank error threshold can be increased but results must be inspected.
- Built-in kit presets are only thoroughly tested for rapid and native DNA kits; RNA and Twist kits are exceptions, and unexpected annotations should be reported as issues.

## Evidence

- [methods] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [readme] flank/barcode cost schema and knob for separation threshold: "Non-random locations: Adjust `--min-score-diff`."
- [readme] Default flank cutoff behavior: "This uses a conservative flank-based edit-distance cutoff."
- [methods] Accuracy advantage motivating Barbell scoring: "**>1000× fewer trimming errors** compared to Dorado."
- [discussion] Version-dependent scoring behavior: "Current git version is around 5x faster than the releases by using [Sassy V2]"

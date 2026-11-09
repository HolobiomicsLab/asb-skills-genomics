---
name: pattern-based-read-filtering
description: Use when demultiplexing Nanopore FASTQ reads whose expected barcode/flank
  arrangement must be specified explicitly — e.g. rapid barcoding (SQK-RBK114-96)
  reads, dual-end barcodes, concatenated reads, or custom experiments — and you need
  per-read cut coordinates for trimming.
when_to_use_negative:
- You only want a quick kit-based demux without custom pattern control — use the one-shot
  preset `barbell kit -k <kit-name> ... --maximize` instead
- Input is not Nanopore read data annotated by Barbell (no anno.tsv); the filter step
  consumes Barbell's annotation table, not raw FASTQ directly
- Reads follow a simple single-barcode configuration fully handled by standard demultiplexers
  — pattern-aware filtering is overkill and slower than a preset workflow
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: Provides the `filter` command that matches annotated reads against user-supplied
    patterns and populates cut coordinates for trimming
  repo: https://github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching engine underlying Barbell's barcode/flank
    matching; git version with Sassy V2 is ~5x faster
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: Dorado
  role: Comparison baseline demultiplexer that fails on complex barcode configurations
  repo: https://github.com/nanoporetech/dorado
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:c7526b6a8de172db99d605b5c6185fcc3733a4d9aab555aa604eeddfc4a5faf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Sassy
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pattern-based-read-filtering

## Summary

Filter Nanopore reads against explicit barcode/flank layout patterns using Barbell: the `filter` step matches each read's annotation row to user-specified patterns (e.g. Ftag[fw, *, @left(0..250)] or dual-end/concatenated forms) and populates a `cuts` column used by `trim`.

## When to use

Use when demultiplexing Nanopore FASTQ reads whose expected barcode/flank arrangement must be specified explicitly — e.g. rapid barcoding (SQK-RBK114-96) reads, dual-end barcodes, concatenated reads, or custom experiments — and you need per-read cut coordinates for trimming.

## When NOT to use

- You only want a quick kit-based demux without custom pattern control — use the one-shot preset `barbell kit -k <kit-name> ... --maximize` instead
- Input is not Nanopore read data annotated by Barbell (no anno.tsv); the filter step consumes Barbell's annotation table, not raw FASTQ directly
- Reads follow a simple single-barcode configuration fully handled by standard demultiplexers — pattern-aware filtering is overkill and slower than a preset workflow

## Inputs

- Annotation TSV from `barbell annotate` (columns: read_id, read_len, read_start_bar, read_end_bar, flank_cost, barcode_cost, label, strand, cuts, ...)
- filters.txt listing expected barcode/flank patterns, one per line, with cut specifiers
- Optionally a kit name (e.g. SQK-RBK114-96) to derive annotation patterns

## Outputs

- filtered.tsv: the annotation TSV restricted to reads matching the listed patterns, with the `cuts` column populated with per-read cut coordinates
- Downstream per-pattern trimmed FASTQ files (e.g. BC01.trimmed.fastq) produced by `barbell trim -i filtered.tsv -r reads.fastq -o trimmed`

## How to apply

First run `barbell annotate --kit <kit> -i reads.fastq -o anno.tsv` to locate and edit-distance-score barcodes and flanks per read, then `barbell inspect -i anno.tsv` to see which barcode/flank pattern groups actually occur. Write a `filters.txt` file listing the patterns to keep, one per line, using Barbell's pattern syntax with cut specifiers (e.g. `Ftag[fw, *, @left(0..250), >>]` for single-end and `Ftag[fw, *, @left(0..250), >>]__Ftag[<<, rc, *, @right(0..250)]` for dual-end or concatenated reads), where the cut specifiers define where in each matched element the read will be cut. Run `barbell filter -i anno.tsv -f filters.txt -o filtered.tsv`; this matches each read to a pattern, applies optional trim rules, and fills in the `cuts` column of the TSV. The rationale is that pattern-aware matching resolves complex barcode attachments (17% of rapid barcoding reads) that fixed-configuration demultiplexers like Dorado and Flexiplex mishandle, leaving reads contaminated with adapter fragments.

## Related tools

- **Barbell** (Provides the `filter` command that matches annotated reads against user-supplied patterns and populates cut coordinates for trimming) — https://github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching engine underlying Barbell's barcode/flank matching; git version with Sassy V2 is ~5x faster) — https://github.com/RagnarGrootKoerkamp/sassy
- **Dorado** (Comparison baseline demultiplexer that fails on complex barcode configurations) — https://github.com/nanoporetech/dorado

## Examples

```
barbell filter -i anno.tsv -f filters.txt -o filtered.tsv && barbell trim -i filtered.tsv -r reads.fastq -o trimmed
```

## Evaluation signals

- Every read in anno.tsv from a synthetic test set is assigned its ground-truth barcode label (e.g. BC29, BC45) with expected flank_cost/barcode_cost
- filtered.tsv contains a populated `cuts` column for every retained read and excludes reads not matching the filters.txt patterns
- Downstream trimmed FASTQ files (e.g. BC01.trimmed.fastq) contain the expected insert sequence with barcode/flank removed exactly at the annotated cut coordinates
- `barbell inspect -i anno.tsv` confirms the expected pattern groups (single-end Ftag and dual-end/concatenated variants) before filtering
- Contaminated/adapter-carrying read counts drop relative to Dorado/Flexiplex, which left ~10% of rapid barcoding reads partially trimmed

## Limitations

- Release versions are slower than the current git version (~5x faster with Sassy V2), and results can slightly differ (<0.01%) between versions
- With relaxed flank thresholds, random `Fflank` matches can appear at unexpected locations; thresholds like --flank-max-errors or --min-score-diff must be tuned and results re-inspected
- Too few annotated reads may indicate the automatically derived flank edit-distance cutoff is too strict and should be relaxed, with inspection afterwards to avoid spurious matches

## Evidence

- [full_text] Barbell's demultiplexing pipeline operates in four fixed steps: the annotate step locates and scores barcodes/flanks in the reads: [evidence span withheld — anchor status: not_found]
- [full_text] filter step matches reads to a pattern and optional trim rules: [evidence span withheld — anchor status: not_found]
- [methods] Create a filters.txt file listing the patterns you want to keep, one per line.: "Create a `filters.txt` file listing the patterns you want to keep, one per line."
- [readme] Barbell annotate --kit SQK-RBK114-96 CLI example: "barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv"
- [abstract] Widely used demultiplexers leave ~10% of rapid barcoding reads contaminated: "fail to resolve these cases, leaving ∼10% of our rapid barcoding reads partially trimmed and contaminated with adapter fragments."

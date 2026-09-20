---
name: nanopore-read-demultiplexing
description: Use when demultiplexing Nanopore FASTQ reads from barcoding kits (e.g.
  SQK-RBK114-96 rapid or SQK-NBD114-96 native barcoding), especially when reads show
  complex barcode attachments, partially trimmed adapter contamination, or custom
  barcode patterns.
when_to_use_negative:
- Input is already demultiplexed/trimmed sequence data with no barcode constructs
  to resolve
- Reads come from RNA or Twist kits, which are exceptions to Barbell's tested DNA
  kit presets
- Ultra-strict barcode configuration is required and reads must not be maximally retained
  (consider whether the --maximize flag is appropriate)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Barbell
  role: Pattern-aware demultiplexer performing the annotate/inspect/filter/trim pipeline
    and kit presets
  repo: https://github.com/rickbeeloo/barbell
- name: Dorado
  role: Widely used baseline demultiplexer that fails on complex barcode configurations;
    used for comparison
  repo: https://github.com/nanoporetech/dorado
- name: Sassy
  role: SIMD-accelerate approximate string matching library underlying Barbell's searches
  repo: https://github.com/RagnarGrootKoerkamp/sassy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:79dffe58f0b220c271d7f6dd4ddb9a745b3697f5f46315e1434e2f2605925397
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Sassy
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# nanopore-read-demultiplexing

## Summary

Pattern-aware demultiplexing and trimming of Oxford Nanopore reads with Barbell, executed as four fixed steps (annotate → inspect → filter → trim) that detect complex barcode configurations (dual-end, concatenated, artefact attachments) that Dorado and Flexiplex miss.

## When to use

Use when demultiplexing Nanopore FASTQ reads from barcoding kits (e.g. SQK-RBK114-96 rapid or SQK-NBD114-96 native barcoding), especially when reads show complex barcode attachments, partially trimmed adapter contamination, or custom barcode patterns.

## When NOT to use

- Input is already demultiplexed/trimmed sequence data with no barcode constructs to resolve
- Reads come from RNA or Twist kits, which are exceptions to Barbell's tested DNA kit presets
- Ultra-strict barcode configuration is required and reads must not be maximally retained (consider whether the --maximize flag is appropriate)

## Inputs

- Nanopore FASTQ read file (e.g. pass_sample.fastq or reads.fastq)
- Kit name (e.g. SQK-RBK114-96, SQK-NBD114-96) from Barbell's supported kit presets
- filters.txt with one expected barcode pattern per line, including cut specifiers (>>) and dual-end/concatenated variants

## Outputs

- anno.tsv annotation table with per-read barcode/flank matches, edit-distance costs, labels (BC01...), strand, and cuts columns
- filtered.tsv with the cuts column populated for reads matching the filter patterns
- Per-pattern trimmed FASTQ files (e.g. BC01.trimmed.fastq) with barcode/flank removed at annotated cut coordinates

## How to apply

Run `barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv` to locate and edit-distance-score flanks and barcodes per read (columns include read_id, match_type Ftag/Fflank, flank_cost, barcode_cost, label, strand). Summarize the unique barcode/flank patterns with `barbell inspect -i anno.tsv` (top 10 groups by default, `-n` to increase). Write a filters.txt listing expected patterns with cut specifiers (e.g. `Ftag[fw, *, @left(0..250), >>]` and dual-end `Ftag[fw, *, @left(0..250), >>]__Ftag[<<, rc, *, @right(0..250)]` variants) and run `barbell filter` to populate the cuts column. Finally run `barbell trim -i filtered.tsv -r reads.fastq -o trimmed` to cut reads at exact annotated positions, writing per-pattern files like BC01.trimmed.fastq. Rationale: only ~83% of rapid barcoding reads follow the expected single-barcode configuration (17% show complex attachments), so pattern-aware matching is needed; for kit runs use `barbell kit -k <kit-name> -i <reads.fastq> -o <out> --maximize` and relax `--flank-max-errors` (e.g. to 5) only if many reads are missed, since too many `Fflank` matches indicate random matches.

## Related tools

- **Barbell** (Pattern-aware demultiplexer performing the annotate/inspect/filter/trim pipeline and kit presets) — https://github.com/rickbeeloo/barbell
- **Dorado** (Widely used baseline demultiplexer that fails on complex barcode configurations; used for comparison) — https://github.com/nanoporetech/dorado
- **Sassy** (SIMD-accelerate approximate string matching library underlying Barbell's searches) — https://github.com/RagnarGrootKoerkamp/sassy

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv; barbell inspect -i anno.tsv; barbell filter -i anno.tsv -f filters.txt -o filtered.tsv; barbell trim -i filtered.tsv -r reads.fastq -o trimmed
```

## Evaluation signals

- Every synthetic read is assigned its ground-truth barcode label in anno.tsv
- Trimmed FASTQ files contain the expected insert sequence with barcode/flank removed exactly at the annotated cut coordinates
- anno.tsv shows low barcode_cost and flank_cost (edit distances) and match_type Ftag rather than unexpected Fflank matches at random read positions
- Contaminated-read count drops by orders of magnitude versus Dorado/Flexiplex (Barbell reported 99.96% reduction, >400000 down to 166)
- inspect output matches the expected unique barcode/flank pattern groups for the kit

## Limitations

- Dorado/Flexiplex leave ~10% of rapid barcoding reads partially trimmed and contaminated with adapter fragments; only pattern-aware demux resolves these cases
- If --flank-max-errors is relaxed (e.g. to 5), random Fflank matches can appear and thresholds must be re-inspected
- Failing to open enough file handles (ulimit) can cause errors; results may differ slightly (<0.01%) between the release and current git version (Sassy V2)
- Non-rapid/non-native kits are not thoroughly tested; unexpected annotations should be reported as issues

## Evidence

- [other] Barbell's demultiplexing pipeline operates in four fixed steps: the annotate step locates and scores barcodes/flanks in the reads, producing a tsv file with matches per read: [evidence span withheld — anchor status: not_found]
- [abstract] We developed Barbell, a pattern-aware demultiplexer that is designed to detect complex barcode configurations.: "We developed Barbell, a pattern-aware demultiplexer that is designed to detect complex barcode configurations."
- [readme] Run `annotate` to find matches in reads and output an annotation table (TSV): "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [abstract] only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments.: "we found that only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments."
- [readme] Trim reads using the `cuts` metadata produced by `filter`: "Trim reads using the `cuts` metadata produced by `filter`"

---
name: nanopore-adapter-barcode-trimming
description: Use when Nanopore FASTQ reads from rapid (e.g. SQK-RBK114-96) or native
  (e.
when_to_use_negative:
- The data are already demultiplexed and adapter/barcode-trimmed (no annotation TSV
  or tag sequences remain to be resolved)
- Reads come from RNA or Twist kits, which are exceptions to the built-in kit presets,
  without first verifying support
- A single fast, pattern-naive split suffices and read sets follow the expected single-barcode
  configuration with no complex attachments
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: Barbell
  role: Pattern-aware demultiplexer; provides the annotate, inspect, filter, trim,
    and kit subcommands that locate barcodes/flanks, classify configurations, and
    trim reads at exact cut positions
  repo: https://github.com/rickbeeloo/barbell
- name: Dorado
  role: Widely-used Nanopore demultiplexer referenced as the baseline that fails on
    complex barcode configurations; not part of the skill workflow itself
  repo: https://github.com/nanoporetech/dorado
- name: Sassy
  role: SIMD-accelerated approximate string-matching library underlying Barbell's
    search speed (Sassy V2 makes the git version ~5x faster than releases)
  repo: https://github.com/RagnarGrootKoerkamp/sassy
- name: barbell-evals
  role: Evaluation code to reproduce the paper's benchmark results comparing demultiplexers
  repo: https://github.com/rickbeeloo/barbell-evals
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:4e23714bb7e6a83d10427ec9d081fcdb4a3ef52ddc3ca3e382aeba7bf0627b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rickbeeloo/barbell
  tools:
  - Barbell
  - Dorado
  - Sassy
  - barbell-evals
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# nanopore-adapter-barcode-trimming

## Summary

A four-step pattern-aware demultiplexing and trimming workflow (annotate → inspect → filter → trim) implemented in Barbell that locates barcodes/flanks in Nanopore reads, classifies read barcode configurations, and trims reads at exact annotated cut positions, resolving complex attachments that tools like Dorado and Flexiplex miss.

## When to use

Use when Nanopore FASTQ reads from rapid (e.g. SQK-RBK114-96) or native (e.g. SQK-NBD114-96) barcoding kits must be demultiplexed and trimmed, especially when reads show dual-end, concatenated, or otherwise complex barcode attachments that standard demultiplexers leave contaminated with adapter fragments.

## When NOT to use

- The data are already demultiplexed and adapter/barcode-trimmed (no annotation TSV or tag sequences remain to be resolved)
- Reads come from RNA or Twist kits, which are exceptions to the built-in kit presets, without first verifying support
- A single fast, pattern-naive split suffices and read sets follow the expected single-barcode configuration with no complex attachments

## Inputs

- Nanopore FASTQ reads (e.g. pass_sample.fastq) from rapid or native barcoding kits
- Kit name for built-in presets (e.g. SQK-RBK114-96, SQK-NBD114-96) or a custom query FASTA of barcodes/flanks
- filters.txt listing expected barcode patterns with cut specifiers (for the filter step)

## Outputs

- anno.tsv — per-read annotation of barcode/flank matches with edit-distance costs, label, strand, match_type
- Pattern-group summary from inspect (counts of unique barcode/flank patterns)
- filtered.tsv — annotation with populated cuts column for reads matching filter patterns
- Per-pattern trimmed FASTQ files (e.g. BC01.trimmed.fastq) with barcodes/flanks removed at annotated cut coordinates

## How to apply

First run `barbell annotate` with the appropriate kit (e.g. --kit SQK-RBK114-96) to locate and edit-distance-score flanks and barcodes per read, producing a TSV with columns such as read_id, match_type, flank_cost, barcode_cost, label, strand, and cuts. Then run `barbell inspect` on the annotation file to summarize the observed barcode/flank pattern groups (top 10 by default, adjustable with -n) and confirm which configurations are present. Next, write a filters.txt listing the expected patterns with cut specifiers (e.g. Ftag[fw, *, @left(0..250), >>] or dual-end Ftag[fw, *, @left(0..250), >>]__Ftag[<<, rc, *, @right(0..250)]) and run `barbell filter` to match reads to patterns and populate the cuts column. Finally run `barbell trim` with the filtered TSV and the original reads to cut at the exact annotated alignment positions, writing per-pattern outputs such as BC01.trimmed.fastq; alternatively use the `barbell kit` preset command with --maximize for a one-shot pipeline. If too few reads are annotated, relax --flank-max-errors; if many Fflank matches appear at random locations, lower it.

## Related tools

- **Barbell** (Pattern-aware demultiplexer; provides the annotate, inspect, filter, trim, and kit subcommands that locate barcodes/flanks, classify configurations, and trim reads at exact cut positions) — https://github.com/rickbeeloo/barbell
- **Dorado** (Widely-used Nanopore demultiplexer referenced as the baseline that fails on complex barcode configurations; not part of the skill workflow itself) — https://github.com/nanoporetech/dorado
- **Sassy** (SIMD-accelerated approximate string-matching library underlying Barbell's search speed (Sassy V2 makes the git version ~5x faster than releases)) — https://github.com/RagnarGrootKoerkamp/sassy
- **barbell-evals** (Evaluation code to reproduce the paper's benchmark results comparing demultiplexers) — https://github.com/rickbeeloo/barbell-evals

## Examples

```
barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv && barbell inspect -i anno.tsv && barbell filter -i anno.tsv -f filters.txt -o filtered.tsv && barbell trim -i filtered.tsv -r reads.fastq -o trimmed
```

## Evaluation signals

- Every synthetic read is assigned its ground-truth barcode label in anno.tsv (label column matches expected BC## per read)
- Trimmed FASTQ files contain the expected insert sequence with barcode/flank removed at the annotated cut coordinates (cuts column)
- Contaminated (partially trimmed) reads approach zero relative to Dorado/Flexiplex baselines — paper reports reduction from >400 000 to 166 reads (99.96%)
- Inspect output shows only the expected pattern groups (plus explained variants such as Fflank or dual-end concat patterns), with no unexpected Fflank matches at random read locations
- Trimming-error rate is drastically lower than Dorado (>1000× fewer trimming errors reported)

## Limitations

- Only 83% of rapid barcoding reads follow the expected single-barcode configuration (17% show complex attachments), and in native barcoding only 30%–70% of reads have barcodes on both ends, so annotate may miss reads unless --flank-max-errors is relaxed
- Installing from GitHub differs from release versions: the git version (Sassy V2) is ~5x faster but results can slightly differ (<0.01%); the GitHub CLI also accepts multiple files via shell expansion unlike the release
- Too-low flank error thresholds yield random Fflank matches; ulimit file-handle errors were a known limitation when writing many per-barcode outputs, and --use-extended fusion-point search is ~3x slower
- Kit presets were thoroughly tested only for rapid and native kits; RNA and Twist kits are exceptions

## Evidence

- [methods] four-step annotate-inspect-filter-trim pipeline: "Run `annotate` to find matches in reads and output an annotation table (TSV)"
- [methods] trim step: "Trim reads using the `cuts` metadata produced by `filter`:

bash
barbell trim -i filtered.tsv -r reads.fastq -o trimmed"
- [abstract] contamination reduction: "Barbell reduced contaminated reads from >400 000 (Dorado/Flexiplex) to 166 (99.96% reduction)"
- [abstract] complex attachments frequency: "we found that only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments."
- [readme] inspect default behavior: "By default `inspect` shows the top 10 pattern groups; use `-n <amount>` to increase."
- [readme] flank threshold tuning guidance: "Slightly increase `--flank-max-errors` (the automatically derived cutoff is reported)."

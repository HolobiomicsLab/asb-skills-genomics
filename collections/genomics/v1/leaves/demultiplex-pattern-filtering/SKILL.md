---
name: demultiplex-pattern-filtering
description: Use when Nanopore FASTQ reads need demultiplexing/trimming under ONT
  kit presets (e.g. SQK-RBK114-96, SQK-NBD114-96), or when many reads deviate from
  the expected single-barcode configuration (contamination, partial trims, fused double
  barcodes).
when_to_use_negative:
- Reads are not Nanopore kit-barcoded data, or the kit is not in Barbell's supported_kits.txt
  (only rapid/native DNA kits were thoroughly tested; RNA and Twist kits are exceptions)
- You need ultra-strict single-barcode configurations and cannot tolerate the ~3x
  slowdown of --use-extended when fusion-point searching is unnecessary
- The analysis goal is adapter trimming alone with no barcode assignment; a simple
  trimmer may suffice rather than pattern-aware demux
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Barbell
  role: 'Primary tool: pattern-aware annotate, inspect, filter, trim and kit demultiplexing;
    --use-extended enables fusion-pattern detection'
  repo: github.com/rickbeeloo/barbell
- name: Sassy
  role: SIMD-accelerated approximate string matching backend that speeds up Barbell's
    search (git version ~5x faster via Sassy V2)
  repo: github.com/RagnarGrootKoerkamp/sassy
- name: Dorado
  role: Baseline demultiplexer that fails on complex barcode configurations and leaves
    ~10% of rapid-barcoding reads partially trimmed; used for comparison only
  repo: nanoporetech/dorado
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag349
    title: Barbell reveals and resolves demultiplexing and trimming issues in Nanopore
      data
  source_content_hash: sha256:38f3426a181a20a212720cd80e5f4acb4787df41ba033199a8e446f4fe76e9c5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Barbell
  - Dorado
  - Sassy
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# demultiplex-pattern-filtering

## Summary

Pattern-aware demultiplexing and pattern-group filtering of Nanopore reads using Barbell, which detects complex barcode configurations (flank-only, fused double barcodes via --use-extended) that flat-matchers like Dorado miss, and trims reads based on inspected pattern groups.

## When to use

Use when Nanopore FASTQ reads need demultiplexing/trimming under ONT kit presets (e.g. SQK-RBK114-96, SQK-NBD114-96), or when many reads deviate from the expected single-barcode configuration (contamination, partial trims, fused double barcodes).

## When NOT to use

- Reads are not Nanopore kit-barcoded data, or the kit is not in Barbell's supported_kits.txt (only rapid/native DNA kits were thoroughly tested; RNA and Twist kits are exceptions)
- You need ultra-strict single-barcode configurations and cannot tolerate the ~3x slowdown of --use-extended when fusion-point searching is unnecessary
- The analysis goal is adapter trimming alone with no barcode assignment; a simple trimmer may suffice rather than pattern-aware demux

## Inputs

- Nanopore FASTQ reads (optionally gzipped) from a supported ONT kit, e.g. SQK-RBK114-96 rapid or SQK-NBD114-96 native barcoding
- Kit name recognized by Barbell presets (data/supported_kits.txt)
- filters.txt listing pattern groups to keep (one per line) for the filter step

## Outputs

- anno.tsv annotation table with per-read barcode/flank match positions, costs, labels, strand and cuts
- Pattern-group summary from 'barbell inspect' (top 10 by default, -n to increase)
- filtered.tsv subset of reads matching the requested patterns
- Trimmed FASTQ files written to the output folder, optionally gzipped with --gzip
- fusion_barcode_labels.tsv (read_id with first and second barcode labels) when extracting fused double left-end barcodes

## How to apply

Run 'barbell kit -k <kit-name> -i reads.fastq -o out --maximize' for standard demux, adding '--use-extended' to search fusion points and other artefact configurations (~3x slower, worth it for consensus-grade output). For manual analysis use the annotate -> inspect -> filter -> trim workflow: 'barbell annotate --kit SQK-RBK114-96 -i pass_sample.fastq -t 10 -o anno.tsv' produces a per-read TSV with bar/flank positions, match_type, flank_cost, barcode_cost, label, strand and cuts; 'barbell inspect -i anno.tsv' summarizes the top pattern groups (use -n to increase). Write desired patterns (e.g. Ftag[fw, *, @left(0..250)]__Ftag[fw, *, @prev_left(0..250)]) into filters.txt, run 'barbell filter' then 'barbell trim' using the cuts metadata. Adjust '--flank-max-errors' (e.g. 5) when too few reads are annotated, or lower it when too many random Fflank matches appear; the fused second barcode shows a higher match cost (mean edit distance ~7 vs ~3 for an intact barcode+flank match).

## Related tools

- **Barbell** (Primary tool: pattern-aware annotate, inspect, filter, trim and kit demultiplexing; --use-extended enables fusion-pattern detection) — github.com/rickbeeloo/barbell
- **Sassy** (SIMD-accelerated approximate string matching backend that speeds up Barbell's search (git version ~5x faster via Sassy V2)) — github.com/RagnarGrootKoerkamp/sassy
- **Dorado** (Baseline demultiplexer that fails on complex barcode configurations and leaves ~10% of rapid-barcoding reads partially trimmed; used for comparison only) — nanoporetech/dorado

## Examples

```
barbell kit -k SQK-RBK114-96 -i reads.fastq -o output_folder --use-extended --maximize
```

## Evaluation signals

- Inspect output shows the expected pattern groups, e.g. Ftag[fw, *, @left(0..250)]__Ftag[fw, *, @prev_left(0..250)] for fused double left-end barcodes
- Fused second barcode matches carry a higher cost than intact ones (mean edit distance ~7 vs ~3), consistent with loss of the left flank and partial prefix deletion
- Every synthetic read with a known simulation ground truth gets both barcode labels correctly assigned in fusion_barcode_labels.tsv
- Barcode/flank matches occur near read ends; matches at random read positions with many Fflank hits indicate the flank error threshold is too lenient
- Trimmed output should be contamination-free; Dorado/Flexiplex baselines left >400 000 contaminated reads vs 166 for Barbell (99.96% reduction) in the paper's benchmark

## Limitations

- --use-extended is around 3 times slower than the default mode, so reserve it when quality is essential (e.g. generating consensus sequences)
- Results can slightly differ (<0.01%) between the current git version (Sassy V2) and release builds (conda/cargo)
- RNA and Twist kits are exceptions to the well-tested kit presets; unexpected annotations with other kits should be reported as issues
- Changing thresholds such as --flank-max-errors requires re-inspection to avoid random Fflank matches

## Evidence

- [abstract] We developed Barbell, a pattern-aware demultiplexer that is designed to detect complex barcode configurations.: "We developed Barbell, a pattern-aware demultiplexer that is designed to detect complex barcode configurations."
- [abstract] only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments: "we found that only 83% of reads followed the expected single-barcode configuration, while 17% showed complex barcode attachments."
- [readme] there is an option --use-extended which enables searches for fusion points, and other artefacts. This is around 3 times slower: "there is an option `--use-extended` which enables searches for fusion points, and other artefacts. This is around 3 times slower, "
- [readme] Too many Fflank matches: Check annotation.tsv to see if matches occur at unexpected locations (not near read ends). Random locations: Lower --flank-max-errors.: "**Too many `Fflank` matches:**
  1. Check `annotation.tsv` to see if matches occur at unexpected locations (not near read ends).
     - **Random locations:** Lower `--flank-max-errors`."
- [readme] Barbell's typical manual workflow: annotate → inspect → filter → trim.: "Barbell's typical manual workflow: **annotate → inspect → filter → trim**."
- [abstract] Barbell reduced contaminated reads from >400 000 (Dorado/Flexiplex) to 166 (99.96% reduction): "Barbell reduced contaminated reads from >400 000 (Dorado/Flexiplex) to 166 (99.96% reduction)"

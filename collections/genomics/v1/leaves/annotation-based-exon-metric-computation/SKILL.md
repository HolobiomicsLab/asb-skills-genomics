---
name: annotation-based-exon-metric-computation
description: Use when exon predictions from a splice graph or assembler (e.g. EXFI's
  GFA1 output, gfa1_to_fasta exons) must be quantitatively benchmarked against a reference
  GFF3 annotation to obtain precision, recall and F1.
when_to_use_negative:
- No reference genome or GFF3 annotation exists for the organism (non-model species
  with only draft assemblies) — use mappability-based assessment instead
- The question is whether predicted exon sequences are usable for downstream analysis
  rather than coordinate-exact — mappability is a more optimistic metric than exon
  precision
- Evaluating whole-transcript or splice-graph topology rather than individual exon
  spans
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: bedtools
  role: BEDTools intersect enforces the >=95% mutual-overlap criterion between reference
    and predicted exon coordinates
- name: EXFI
  role: Supplies the predicted exon set (gfa1_to_fasta) and the compare_to_gff3 script
    that computes precision/recall/F1 against the GFF3
  repo: https://github.com/jlanga/exfi
- name: Pandas
  role: Tabulates per-k precision/recall/F1 values from the comparison output
- name: BioPython
  role: Handles sequence/coordinate parsing in the comparison scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:0495de027c2ad1f43a6e3409f93861601f044138e8b0357f1ec1e7927ae6171a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - Pandas
  - Python
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# annotation-based-exon-metric-computation

## Summary

Scores a set of predicted exon coordinates against a reference GFF3 annotation by converting both to transcript-based BED format, testing mutual overlap with BEDTools intersect (>=95% in both directions), and computing precision (TP/(TP+FP)), recall (TP/(TP+FN)) and F1 (2PR/(P+R)) per parameter setting.

## When to use

Use when exon predictions from a splice graph or assembler (e.g. EXFI's GFA1 output, gfa1_to_fasta exons) must be quantitatively benchmarked against a reference GFF3 annotation to obtain precision, recall and F1.

## When NOT to use

- No reference genome or GFF3 annotation exists for the organism (non-model species with only draft assemblies) — use mappability-based assessment instead
- The question is whether predicted exon sequences are usable for downstream analysis rather than coordinate-exact — mappability is a more optimistic metric than exon precision
- Evaluating whole-transcript or splice-graph topology rather than individual exon spans

## Inputs

- Predicted exon FASTA/coordinates from gfa1_to_fasta (derived from EXFI's GFA1 splice graph)
- Reference GFF3 exon annotation (e.g. zebrafish Ensembl release 91, GRCz10)
- The k-mer length or other parameter value under test, for per-condition tabulation

## Outputs

- Transcript-based BED files of reference and predicted exon coordinates (strand- and exon-order-aware)
- Per-exon overlap/match classifications (TP, FP, FN) from BEDTools intersect at 95% mutual overlap
- Precision, recall and F1 per parameter setting, tabulated (e.g. with Pandas) across the k-mer sweep

## How to apply

Convert the reference exon coordinates from chromosome-based GFF3 to transcript-based BED format, accounting for strand and exon order, and express predicted exons in the same coordinate space. Run BEDTools intersect requiring a mutual overlap of at least 95% between predicted and reference exons so that only near-identical spans count as true positives. Classify each prediction as TP (matched), FP (unmatched prediction), or FN (missed reference exon) and compute precision TP/(TP+FP), recall TP/(TP+FN), and F1 = 2PR/(P+R); EXFI's compare_to_gff3 script automates this comparison. Tabulate metrics per tested parameter (e.g. per k-mer length using Pandas) and check that the chosen default is justified by stable, high F1 across the tested range.

## Related tools

- **bedtools** (BEDTools intersect enforces the >=95% mutual-overlap criterion between reference and predicted exon coordinates)
- **EXFI** (Supplies the predicted exon set (gfa1_to_fasta) and the compare_to_gff3 script that computes precision/recall/F1 against the GFF3) — https://github.com/jlanga/exfi
- **Pandas** (Tabulates per-k precision/recall/F1 values from the comparison output)
- **BioPython** (Handles sequence/coordinate parsing in the comparison scripts)

## Examples

```
bedtools intersect -f 0.95 -r -a predicted_exons.bed -b reference_exons_transcript.bed > tp_exons.bed; python compare_to_gff3 --gff3 reference.gff3 --input predicted.gfa1
```

## Evaluation signals

- Every counted true positive shows >=95% overlap in BOTH directions under bedtools intersect; near-boundary partial overlaps are excluded
- F1 equals 2PR/(P+R) for the tabulated precision and recall values (arithmetic consistency check)
- Reference BED conversion preserves exon order and strand relative to the original chromosome-based GFF3
- Across a k-mer sweep, F1 should be near-constant within the stable window (k=23-35) at both 4 GB and 60 GB Bloom filter settings, reproducing the paper's justification of k=25
- Precision and recall recomputed by an independent script agree with compare_to_gff3's output table

## Limitations

- Metrics depend on reference annotation and input transcriptome quality: with a single-tissue transcriptome (Atlantic herring), lowly expressed transcripts are fragmented and EXFI was less accurate predicting exons than comparators
- High mappability is more optimistic than exon precision because it covers only interior exon sections, not full exon sequences
- A 95% mutual-overlap threshold penalizes predictions with correct boundaries but slightly different spans; results are anchored to the specific benchmark (zebrafish/human simulations, Ensembl release 91)

## Evidence

- [other] Compare reference and predicted exon coordinates with BEDTools intersect requiring a mutual overlap of at least 95%, and compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1: [evidence span withheld — anchor status: not_found]
- [methods] Exon length filter threshold: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [readme] Readme description of toolchain: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
- [results] Read filtering improves classification metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"

---
name: sv-callset-benchmarking
description: Use when evaluating or reproducing SV caller performance metrics (precision/recall/F1)
  on simulated or GIAB/Wagner-style truth sets using synthetic long reads, hybrid
  SLR+long-read data, or low-coverage (e.g. 5x) long-read benchmarks.
when_to_use_negative:
- The goal is calling or typing variants, not measuring caller performance — use Blackbird
  directly rather than this benchmarking workflow.
- Truth set is unavailable or the events of interest are outside the reliable 50-10,000
  bp indel range Blackbird reports (out.vcf calls are explicitly unreliable).
- Input is short-read-only data without SLR barcodes or long reads — Blackbird and
  its comparison SLR/hybrid callers are not applicable.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Primary SV caller under evaluation; run in SLR-only or hybrid mode to produce
    out_50.vcf calls for benchmarking
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Blackbird was forked from the SPAdes codebase; spades_for_blackbird supplies
    the local-assembly module invoked by Blackbird
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the BWA-indexed reference genome required by Blackbird's -r option
- name: PBSV
  role: State-of-the-art long-read comparison caller at 10x PacBio HiFi coverage in
    benchmarks
- name: Sniffles2
  role: State-of-the-art long-read comparison caller at 10x PacBio HiFi coverage and
    on the Wagner challenging-gene subset
- name: LongRanger
  role: Comparison caller in the simulated 37x SLR chromosome-1 benchmark
- name: Manta
  role: Comparison short-read caller in the simulated SLR benchmark
- name: SvABA
  role: Comparison caller in the simulated SLR benchmark
- name: LEVIATHAN
  role: Comparison SLR caller in the simulated SLR benchmark
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:2f86bb132cfe2aaa119009724e7fa85408fbf0ab99737265fea32c62f23abb3b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - LEVIATHAN
  - LongRanger
  - Manta
  - PBSV
  - SPAdes
  - Sniffles2
  - SvABA
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# sv-callset-benchmarking

## Summary

Benchmark structural variant callers (Blackbird in SLR or hybrid mode against LongRanger, Manta, SvABA, LEVIATHAN, PBSV, Sniffles2) by computing precision, recall, and F1 for insertions and deletions longer than 50 bp on curated truth sets such as HG002 GIAB and the Wagner et al. 2022 challenging medically relevant autosomal gene callset.

## When to use

Use when evaluating or reproducing SV caller performance metrics (precision/recall/F1) on simulated or GIAB/Wagner-style truth sets using synthetic long reads, hybrid SLR+long-read data, or low-coverage (e.g. 5x) long-read benchmarks.

## When NOT to use

- The goal is calling or typing variants, not measuring caller performance — use Blackbird directly rather than this benchmarking workflow.
- Truth set is unavailable or the events of interest are outside the reliable 50-10,000 bp indel range Blackbird reports (out.vcf calls are explicitly unreliable).
- Input is short-read-only data without SLR barcodes or long reads — Blackbird and its comparison SLR/hybrid callers are not applicable.

## Inputs

- Position-sorted, indexed SLR read BAM with BX barcode tags (e.g. NA24385.GRCh37.phased_possorted_bam.bam, GRCh37/hg19)
- BWA-indexed reference genome FASTA (GRCh37/hg19)
- Optional position-sorted, indexed long-read BAM paired with matching long-read FASTQ (e.g. 5x PacBio HiFi for HG002)
- Optional regions file ('chr start end' per row) restricting calling to ~200 target autosomal genes (Wagner et al. 2022)
- Truth-set SV VCFs (HG002 GIAB benchmark; Wagner et al. 2022 curated callset; simulated 37x SLR chromosome-1 truth set)
- VCF outputs from comparison callers (LongRanger, Manta, SvABA, LEVIATHAN, PBSV, Sniffles2)

## Outputs

- out_50.vcf — position-sorted VCF of insertion and deletion calls longer than 50 bp
- hybrid_benchmark_results.csv — precision/recall metrics on the Wagner et al. 2022 challenging gene callset
- blackbird_slr_benchmark_metrics.csv — recall, precision, F1 per caller per SV class on the simulated SLR dataset

## How to apply

Run Blackbird (compiled via ./spades_compile.sh from the SPAdes fork) in SLR-only mode (just -b SLR bam with BX tags, -r BWA-indexed reference, -o output folder, -t threads) or hybrid mode (additionally --long-bam/-l paired with --long-read/-m FASTQ, optionally --regions/-g to restrict to target genes). Collect the primary output out_50.vcf, the position-sorted VCF of insertion and deletion calls longer than 50 bp, and filter comparison callers' VCFs to the same 50-10,000 bp indel range to match Blackbird's reporting. Compare calls against truth sets using Truvari (for the Wagner et al. 2022 HG002 curated callset) or SURVIVOR-style reciprocal-overlap matching (for the simulated 37x SLR chromosome-1 dataset), computing TP/FP/FN counts. Aggregate per-class (insertion/deletion) precision, recall, and F1 into a metrics table; validation anchors include Blackbird hybrid precision ~86% and recall ~71% on the Wagner challenging-gene callset at 5x coverage, and SLR-mode insertions recall 58%, F1 0.71, precision 94% plus deletions recall 72%, F1 0.81, precision 94% on the simulated dataset.

## Related tools

- **Blackbird** (Primary SV caller under evaluation; run in SLR-only or hybrid mode to produce out_50.vcf calls for benchmarking) — https://github.com/1dayac/Blackbird
- **SPAdes** (Blackbird was forked from the SPAdes codebase; spades_for_blackbird supplies the local-assembly module invoked by Blackbird) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the BWA-indexed reference genome required by Blackbird's -r option)
- **PBSV** (State-of-the-art long-read comparison caller at 10x PacBio HiFi coverage in benchmarks)
- **Sniffles2** (State-of-the-art long-read comparison caller at 10x PacBio HiFi coverage and on the Wagner challenging-gene subset)
- **LongRanger** (Comparison caller in the simulated 37x SLR chromosome-1 benchmark)
- **Manta** (Comparison short-read caller in the simulated SLR benchmark)
- **SvABA** (Comparison caller in the simulated SLR benchmark)
- **LEVIATHAN** (Comparison SLR caller in the simulated SLR benchmark)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Blackbird hybrid mode on the Wagner et al. 2022 challenging gene callset at 5x long-read coverage reproduces precision ~86% and recall ~71% within Truvari matching tolerance.
- SLR-mode on the simulated 37x chromosome-1 dataset reproduces insertions recall 58%, F1 0.71, precision 94% and deletions recall 72%, F1 0.81, precision 94%.
- All caller VCFs are filtered to insertion/deletion calls longer than 50 bp so comparisons match Blackbird's out_50.vcf reporting range.
- Metrics are computed from Truvari (or SURVIVOR-style reciprocal-overlap) confusion counts (TP, FP, FN) restricted to the target regions, giving values in [0,1] with F1 consistent with precision and recall.
- out_50.vcf is position-sorted and restricted to the regions file (e.g. the ~200 autosomal genes) when -g is supplied.

## Limitations

- Blackbird performance is lower than Sniffles2 and PBSV on the Wagner et al. 2022 subset of relatively complex events, though still achieving 86% precision and 71% recall at 5x coverage.
- Calls in out.vcf (other than out_50.vcf) are explicitly 'not reliable' and should not be benchmarked.
- 5x coverage meets the reported F1 targets (0.835 deletions, 0.808 insertions) but is lower than the 10x PacBio HiFi used for PBSV/Sniffles2 comparisons, so coverage must be controlled when comparing.

## Evidence

- [abstract] Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 usin"
- [methods] Blackbird assembles the genome into segments and calls insertions and deletions in these segments.: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments."
- [readme] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [intro] In a SLR read mode Blackbird outperforms existing short-read and SLR read methods, especially for insertions.: "In a SLR read mode Blackbird outperforms existing short-read and SLR read methods, especially for insertions."

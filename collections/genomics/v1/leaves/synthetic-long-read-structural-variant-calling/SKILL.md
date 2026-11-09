---
name: synthetic-long-read-structural-variant-calling
description: Use when input is a position-sorted, indexed SLR BAM with BX barcode
  tags aligned to a BWA-indexed reference (optionally paired with low-coverage long-read
  BAM/FASTQ) and the goal is sensitive insertion/deletion calling in the 50–10,000
  bp range.
when_to_use_negative:
- The dataset is short-read-only without BX barcode tags — Blackbird depends on SLR
  barcode information encoded in reads
- The target events are outside the 50–10,000 bp range or calls shorter than 50 bp
  are needed — out_50.vcf only reports calls longer than 50 bp and out.vcf calls are
  'not reliable'
- A whole-genome assembly or non-SV variant calling (SNVs, small indels <50 bp) is
  the goal — Blackbird performs barcode-aware segment assembly for SV detection only
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_3673
tools:
- name: Blackbird
  role: 'Primary caller: barcode-aware sliding-window local assembly to detect insertions/deletions
    in SLR-only or hybrid mode'
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Codebase fork and local-assembly engine; spades_for_blackbird must be installed
    and spades.py supplied via -s
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the required reference genome index (-r expects a BWA-indexed reference);
    also used in MismatchCorrector
- name: PBSV
  role: Comparison long-read caller for hybrid-mode benchmarking
- name: Sniffles2
  role: Comparison long-read caller for hybrid-mode benchmarking
- name: Truvari
  role: Evaluation of caller VCFs against ground-truth SV lists to compute recall
    and precision per SV type
merged_aliases:
- long-read-structural-variant-calling
merged_alias_records:
- alias: structural-variant-calling-from-synthetic-long-reads
  slug: structural-variant-calling-from-synthetic-long-reads
  jaccard_score: 0.7143
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:8e3affc5ad7bae39abf18b41ab1981c396db330c53137a4ad8dbd957d943472e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes
  - SVIM
  - Sniffles
  - Sniffles2
  - Truvari
  - dysgu
  - nanovar
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# synthetic-long-read-structural-variant-calling

## Summary

Call medium-size structural variants (50–10,000 bp insertions and deletions) using synthetic long reads (SLR) with barcode information, optionally augmented with low-coverage PacBio HiFi long reads (hybrid mode), via the barcode-aware sliding-window local assembler Blackbird.

## When to use

Use when input is a position-sorted, indexed SLR BAM with BX barcode tags aligned to a BWA-indexed reference (optionally paired with low-coverage long-read BAM/FASTQ) and the goal is sensitive insertion/deletion calling in the 50–10,000 bp range.

## When NOT to use

- The dataset is short-read-only without BX barcode tags — Blackbird depends on SLR barcode information encoded in reads
- The target events are outside the 50–10,000 bp range or calls shorter than 50 bp are needed — out_50.vcf only reports calls longer than 50 bp and out.vcf calls are 'not reliable'
- A whole-genome assembly or non-SV variant calling (SNVs, small indels <50 bp) is the goal — Blackbird performs barcode-aware segment assembly for SV detection only

## Inputs

- Position-sorted, indexed SLR read BAM with BX barcode tags (e.g. NA24385 10X phased possorted BAM)
- BWA-indexed reference genome (GRCh37/hg19 or GRCh38)
- Optional: position-sorted, indexed long-read BAM (e.g. PacBio HiFi, 5x–30x coverage) with matching FASTQ
- Optional: regions file with 'chr start end' rows

## Outputs

- out_50.vcf — position-sorted VCF of insertion and deletion calls longer than 50 bp
- out.vcf — all other (unreliable) calls
- Benchmark metrics table (recall, precision, F1 per SV type per caller), e.g. blackbird_slr_benchmark_metrics.csv
- Assembly folders (with -d) containing contigs used for SV calling

## How to apply

Compile Blackbird (forked from SPAdes, requiring the separate spades_for_blackbird build) with ./spades_compile.sh, then run the binary with the required -b (BX-tagged SLR BAM), -r (BWA-indexed reference, e.g. GRCh37/GRCh38), and -o (output folder) options, adding -t for multithreading and the -s path to spades.py. Blackbird assembles small genome segments via a barcode-aware sliding window with local assembly (SPAdes-based), calling insertions and deletions per segment without whole-genome assembly; reads longer than 50 bp are written to the position-sorted out_50.vcf. For hybrid mode, additionally supply -l (indexed long-read BAM) and -m (matching long-read FASTQ) to integrate long reads and boost recall. Benchmark against competing callers (LongRanger, Manta, SvABA, LEVIATHAN in SLR mode; PBSV, Sniffles2 in hybrid mode) using Truvari or SURVIVOR-style reciprocal-overlap evaluation to compute recall, precision, and F1 separately for insertions and deletions.

## Related tools

- **Blackbird** (Primary caller: barcode-aware sliding-window local assembly to detect insertions/deletions in SLR-only or hybrid mode) — https://github.com/1dayac/Blackbird
- **SPAdes** (Codebase fork and local-assembly engine; spades_for_blackbird must be installed and spades.py supplied via -s) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the required reference genome index (-r expects a BWA-indexed reference); also used in MismatchCorrector)
- **PBSV** (Comparison long-read caller for hybrid-mode benchmarking)
- **Sniffles2** (Comparison long-read caller for hybrid-mode benchmarking)
- **Truvari** (Evaluation of caller VCFs against ground-truth SV lists to compute recall and precision per SV type)

## Examples

```
./assembler/bin/blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- SLR-mode benchmark row reproduces reported values: insertions recall 58%, precision 94%, F1 0.71; deletions recall 72%, precision 94%, F1 0.81
- Hybrid mode at 5x long-read coverage achieves F1 of 0.835 (deletions) and 0.808 (insertions), comparable to PBSV/Sniffles2 at 10x
- out_50.vcf is position-sorted and contains only insertion/deletion calls longer than 50 bp
- VCF coordinates and alleles match the reference genome indexing used (-r) and calls fall inside supplied regions when -g/--regions is used
- Recall rises with long-read coverage from 5x to 30x (deletions 85–87%, insertions 75–77%, precision 90–93%)

## Limitations

- Calls in out.vcf (shorter than 50 bp or otherwise) are explicitly 'not reliable'
- SLR-only mode requires BX barcode-tagged BAMs; without barcodes the barcode-aware sliding window cannot function
- Low-coverage hybrid input below 5x long reads may lose the reported F1 advantage; results degrade with less long-read data

## Evidence

- [methods] Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"
- [readme] Out_50.vcf is position-sorted: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [abstract] Blackbird requires only 5x coverage: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage."
- [results] SLR mode outperforms existing methods: "In a SLR read mode Blackbird outperforms existing short-read and SLR read methods, especially for insertions."
- [readme] Compile Blackbird via spades_compile.sh: "To compile Blackbird in assembler foder execute:

``n./spades_compile.sh"

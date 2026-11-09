---
name: hybrid-slr-plus-long-read-benchmarking
description: Use when evaluating SV-calling performance of a barcode-aware SLR + long-read
  pipeline against long-read-only callers, or when determining the minimum long-read
  coverage needed to match PBSV/Sniffles2 F1 scores.
when_to_use_negative:
- When the input is short-read-only data without SLR barcode (BX) tags — Blackbird's
  hybrid and SLR modes require barcode-aware sliding window assembly
- When calling SVs smaller than 50 bp or larger than ~10,000 bp — Blackbird targets
  the 50–10,000 bp medium-size range and out_50.vcf only contains calls longer than
  50 bp
- When a whole-genome assembly is already available and preferred — Blackbird is designed
  to avoid computationally expensive whole-genome assembly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3175
- http://edamontology.org/topic_3673
tools:
- name: Blackbird
  role: Hybrid-mode SV caller being benchmarked; assembles genome segments with barcode-aware
    sliding window using SLR BX tags and integrates long reads
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for Blackbird
  role: Local assembly engine (spades.py from spades_for_blackbird) that Blackbird
    forks from and invokes for segment assembly; required dependency compiled via
    spades_compile.sh
  repo: https://github.com/1dayac/spades_for_blackbird
- name: PBSV
  role: State-of-the-art long-read-only SV caller used as the 10x PacBio HiFi coverage
    baseline for F1 comparison
- name: Sniffles2
  role: Second long-read-only SV caller baseline run on the same long-read BAMs at
    each coverage level
- name: BWA
  role: Provides the indexed reference genome required by Blackbird's -r/--reference
    option
- name: Truvari
  role: Compares each caller's VCF against the ground-truth SV list to compute per-type
    recall and precision at every coverage point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:45b5ceaa3163200b8c28c4dd62375a010eb5a25e33a62d9d6170d91f4f2bb997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes for Blackbird
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3175
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# hybrid-slr-plus-long-read-benchmarking

## Summary

Benchmark Blackbird in hybrid mode (synthetic long reads plus low-coverage PacBio HiFi long reads) against state-of-the-art long-read SV callers PBSV and Sniffles2 on simulated and real (HG002 GIAB) human datasets, using Truvari for recall/precision comparison against ground-truth SV callsets.

## When to use

Use when evaluating SV-calling performance of a barcode-aware SLR + long-read pipeline against long-read-only callers, or when determining the minimum long-read coverage needed to match PBSV/Sniffles2 F1 scores.

## When NOT to use

- When the input is short-read-only data without SLR barcode (BX) tags — Blackbird's hybrid and SLR modes require barcode-aware sliding window assembly
- When calling SVs smaller than 50 bp or larger than ~10,000 bp — Blackbird targets the 50–10,000 bp medium-size range and out_50.vcf only contains calls longer than 50 bp
- When a whole-genome assembly is already available and preferred — Blackbird is designed to avoid computationally expensive whole-genome assembly

## Inputs

- Position-sorted, indexed SLR bam-file with BX barcode tags (e.g., NA24385 10X phased possorted bam)
- BWA-indexed reference genome (GRCh38 or GRCh37/hg19 FASTA)
- Position-sorted, indexed PacBio HiFi long-read BAM plus matching FASTQ, downsampled to 5x–30x coverage
- Ground-truth SV list (e.g., 1437 inserted SVs from Huddleston & Eichler 2016; HG002 GIAB benchmark)
- Long-read caller outputs from PBSV and Sniffles2 on the same BAMs

## Outputs

- Position-sorted out_50.vcf per caller with insertion and deletion calls longer than 50 bp
- Per-coverage Truvari recall/precision results for deletions and insertions
- benchmark_results.csv tabulating recall, precision, and F1 per caller per coverage level

## How to apply

Simulate (with LRSIM) or load a 37x SLR chromosome-1 read set carrying BX barcode tags, BWA-index it against GRCh38, and obtain a ground-truth SV list (e.g., 1437 inserted SVs from Huddleston & Eichler 2016). Run Blackbird in hybrid mode at each downsampled PacBio HiFi coverage level from 5x to 30x, supplying the position-sorted indexed SLR bam (-b), BWA-indexed reference (-r), output folder (-o), and the paired long-read bam (-l) and FASTQ (-m). Run PBSV and Sniffles2 on the identical long-read BAMs at each coverage point, then compare every caller's VCF to the ground truth with Truvari to compute per-type (deletion/insertion) recall, precision, and F1. Correct application is judged by checking that Blackbird's hybrid-mode F1 at 5x long-read coverage (0.835 deletions, 0.808 insertions) approaches PBSV/Sniffles2 results at 10x, with 85–87% deletion recall, 75–77% insertion recall, and 90–93% precision across coverage levels.

## Related tools

- **Blackbird** (Hybrid-mode SV caller being benchmarked; assembles genome segments with barcode-aware sliding window using SLR BX tags and integrates long reads) — https://github.com/1dayac/Blackbird
- **SPAdes for Blackbird** (Local assembly engine (spades.py from spades_for_blackbird) that Blackbird forks from and invokes for segment assembly; required dependency compiled via spades_compile.sh) — https://github.com/1dayac/spades_for_blackbird
- **PBSV** (State-of-the-art long-read-only SV caller used as the 10x PacBio HiFi coverage baseline for F1 comparison)
- **Sniffles2** (Second long-read-only SV caller baseline run on the same long-read BAMs at each coverage level)
- **BWA** (Provides the indexed reference genome required by Blackbird's -r/--reference option)
- **Truvari** (Compares each caller's VCF against the ground-truth SV list to compute per-type recall and precision at every coverage point)

## Examples

```
/home/dmm2017/Blackbird/assembler/bin/blackbird -b /local/storage/data/10X/HG002/NA24385.GRCh37.phased_possorted_bam.bam -r /local/workdir/dmm2017/hg37/refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -l hifi_5x.bam -m hifi_5x.fastq
```

## Evaluation signals

- out_50.vcf is produced in the output folder, position-sorted, and contains only insertion/deletion calls longer than 50 bp
- Blackbird hybrid-mode F1 at 5x long-read coverage reaches 0.835 (deletions) and 0.808 (insertions), comparable to PBSV/Sniffles2 at 10x
- Deletion recall falls in the 85–87% range and insertion recall in the 75–77% range across all 5x–30x coverage levels
- Precision across coverage levels falls in the 90–93% range
- benchmark_results.csv contains recall, precision, and F1 rows for every caller and every downsampled coverage point (5x–30x)

## Limitations

- out.vcf contains all other calls but they are not reliable; only out_50.vcf calls (SVs > 50 bp) should be trusted
- Blackbird targets the 50–10,000 bp medium-size SV range; most SVs in this range cannot be resolved with short-read sequencing, so short-read callers are not valid baselines
- The SLR bam must be position-sorted and indexed with BX tags; the long-read bam and FASTQ must be paired options and position-sorted/indexed
- The --regions option accepts a regions file in 'chr start end' space-separated format; region restriction is optional
- Blackbird was forked from SPAdes, so the separate spades_for_blackbird package must be installed and compiled

## Evidence

- [abstract] Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi long-read coverage.: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2"
- [readme] Hybrid mode CLI options: "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [methods] Sliding window assembly rationale: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [readme] Output specification: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [methods] Coverage requirement claim: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads to achieve same results."

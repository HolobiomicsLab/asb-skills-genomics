---
name: hybrid-slr-longread-variant-calling
description: Use when you have a position-sorted, BX-tagged SLR BAM (e.g. TELL-Seq
  or stLFR) plus low-coverage long-read BAM/FASTQ (e.g. PacBio HiFi at 5–34x) and
  need sensitive SV calling with reduced long-read depth.
when_to_use_negative:
- You only have short-read sequencing without SLR barcode information — most SVs in
  the 50–10,000 bp range cannot be resolved with short reads
- You have long reads but no synthetic long-read (BX-tagged) dataset — Blackbird's
  core barcode-aware sliding-window assembly depends on SLR data
- Calls shorter than 50 bp are needed — Blackbird's reliable output is restricted
  to events longer than 50 bp; shorter calls land in out.vcf and are unreliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: Hybrid SLR + long-read SV caller producing out_50.vcf
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Blackbird was forked from the SPAdes codebase; spades_for_blackbird supplies
    the local-assembly engine invoked via --spades
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the required indexed reference genome for Blackbird (-r)
- name: Truvari
  role: Benchmark out_50.vcf against GIAB/Wagner truth sets to compute precision,
    recall, and F1
- name: Sniffles2
  role: Long-read-only SV caller used as the state-of-the-art comparison at each coverage
    level
- name: PBSV
  role: Long-read-only SV caller used as the state-of-the-art comparison at each coverage
    level
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:414caeaaff180be3da54d8b348e184003deb5d70ee819897bb2f6ecac59d4376
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - PBSV
  - SPAdes
  - Sniffles2
  - Truvari
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# hybrid-slr-longread-variant-calling

## Summary

Call structural variants (insertions/deletions >50 bp) by running Blackbird in hybrid mode, combining barcode-tagged synthetic long reads (SLR) with downsampled long-read data, to match state-of-the-art long-read caller quality with less long-read coverage.

## When to use

Use when you have a position-sorted, BX-tagged SLR BAM (e.g. TELL-Seq or stLFR) plus low-coverage long-read BAM/FASTQ (e.g. PacBio HiFi at 5–34x) and need sensitive SV calling with reduced long-read depth.

## When NOT to use

- You only have short-read sequencing without SLR barcode information — most SVs in the 50–10,000 bp range cannot be resolved with short reads
- You have long reads but no synthetic long-read (BX-tagged) dataset — Blackbird's core barcode-aware sliding-window assembly depends on SLR data
- Calls shorter than 50 bp are needed — Blackbird's reliable output is restricted to events longer than 50 bp; shorter calls land in out.vcf and are unreliable

## Inputs

- Position-sorted and indexed SLR read BAM with BX tags (e.g. TELL-Seq 42x or stLFR 35x)
- BWA-indexed reference genome (e.g. GRCh37/hg19 FASTA)
- Position-sorted and indexed long-read BAM (e.g. downsampled PacBio HiFi at 5x, 10x, 20x, 34x)
- FASTQ file with the same long reads (paired with the long-read BAM)
- Optional regions file with 'chr start end' rows restricting SV calling to target loci

## Outputs

- out_50.vcf — position-sorted VCF of insertion and deletion calls longer than 50 bp
- out.vcf — all other calls (not reliable)
- Truvari-based precision/recall/F1 metrics per SV class (e.g. hybrid_benchmark_results.csv, blackbird_coverage_saturation.csv)

## How to apply

Compile Blackbird with spades_compile.sh (it was forked from the SPAdes codebase, so also install SPAdes for blackbird and point --spades at its spades.py). Run Blackbird with the required inputs: -b for the position-sorted, indexed SLR BAM with BX tags, -r for the BWA-indexed reference genome (e.g. GRCh37/hg19), and -o for the output folder; add hybrid options -l (position-sorted indexed long-read BAM) paired with -m (matching long-read FASTQ), and use -t for multithreading since default is 1. Optionally restrict calling to targets via -g/--regions with 'chr start end' rows (as done for the ~200 Wagner et al. 2022 medically relevant autosomal genes). Blackbird performs barcode-aware sliding-window local assembly of genome segments rather than whole-genome assembly, emitting out_50.vcf (position-sorted VCF of insertions and deletions >50 bp) plus the less reliable out.vcf. Evaluate by comparing out_50.vcf against truth sets (GIAB HG002_SVs_Tier1_v0.6 or Wagner et al. curated callset) with Truvari restricted to indels >50 bp, computing precision, recall, and F1 per SV class.

## Related tools

- **Blackbird** (Hybrid SLR + long-read SV caller producing out_50.vcf) — https://github.com/1dayac/Blackbird
- **SPAdes** (Blackbird was forked from the SPAdes codebase; spades_for_blackbird supplies the local-assembly engine invoked via --spades) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the required indexed reference genome for Blackbird (-r))
- **Truvari** (Benchmark out_50.vcf against GIAB/Wagner truth sets to compute precision, recall, and F1)
- **Sniffles2** (Long-read-only SV caller used as the state-of-the-art comparison at each coverage level)
- **PBSV** (Long-read-only SV caller used as the state-of-the-art comparison at each coverage level)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_hybrid_5x -l hifi_5x.bam -m hifi_5x.fastq -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the output folder and is position-sorted, containing only insertions and deletions longer than 50 bp
- On HG002 GIAB, hybrid Blackbird reaches F1 ≈ 0.835 (deletions) and ≈ 0.808 (insertions) at only 5x PacBio HiFi coverage, matching Sniffles2/PBSV at 10x
- Truvari confusion counts (TP, FP, FN) reproduce precision ≈ 86% and recall ≈ 71% on the Wagner et al. 2022 challenging gene callset with 5x long-read coverage
- Saturation curves show Blackbird saturating earlier (at lower long-read coverage) than Sniffles2 and PBSV, which only saturate at 10–20x
- Long-read BAM and FASTQ inputs refer to the same reads and both are position-sorted and indexed, as required by the paired -l/-m options

## Limitations

- Calls outside out_50.vcf (out.vcf) are explicitly described as 'not reliable'
- Performance drops on complex events: on the Wagner et al. 2022 challenging medically relevant gene subset, Blackbird underperforms Sniffles2 and PBSV
- Default thread count is 1; larger jobs need explicit -t for acceptable runtime
- Blackbird is for SVs in the 50–10,000 bp range; larger rearrangements are outside its demonstrated scope

## Evidence

- [abstract] We propose a novel hybrid alignment- and local-assembly-based algorithm, Blackbird, that uses synthetic long reads and low-coverage long reads to improve structural variant detection.: "We propose a novel hybrid alignment- and local-assembly-based algorithm, Blackbird, that uses synthetic long reads and low-coverage long reads"
- [abstract] Coverage threshold finding: "Blackbird requires only 5× coverage to achieve F1-scores (0.835 and 0.808 for deletions and insertions) similar to PBSV and Sniffles2 using 10× PacBio Hi-Fi"
- [readme] SLR BAM input requirement: "--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Output format: "out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] Hybrid options: "--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [methods] Algorithm rationale: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [methods] Hybrid-mode comparison: "In a hybrid-mode Blackbird demonstrated results similar to state-of-the-art long read tools, but requires less long reads"

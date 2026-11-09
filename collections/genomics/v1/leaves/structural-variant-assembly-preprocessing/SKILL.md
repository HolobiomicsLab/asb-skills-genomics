---
name: structural-variant-assembly-preprocessing
description: Use when preparing medium-size (50–10,000 bp) insertion/deletion SV calls
  from position-sorted, BX-tagged SLR BAMs (optionally with paired long-read BAM/FASTQ)
  and a BWA-indexed reference genome.
when_to_use_negative:
- Whole-genome assembly is desired — Blackbird explicitly avoids computationally expensive
  whole-genome assembly in favor of local segment assembly
- Target SVs are shorter than 50 bp — Blackbird's primary output out_50.vcf contains
  only insertion/deletion calls longer than 50 bp, and shorter calls in out.vcf are
  described as not reliable
- The SLR bam lacks position-sorted, indexed BX tags or the reference is not BWA-indexed
  — required input assumptions are violated
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: Segments the genome into barcode-aware sliding windows, collects barcode-linked
    reads per segment, and orchestrates local assembly and SV calling
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes (spades_for_blackbird)
  role: Modified barcode- and long-read-aware assembler run per 50 kbp segment to
    produce local contig sets
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the indexed reference genome required by Blackbird's alignment-based
    stage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:eade6a72b725e93961956b2b8edd791dd5457c7f387604ae398e600f028a5d0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes (spades_for_blackbird)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# structural-variant-assembly-preprocessing

## Summary

Segment a BWA-indexed reference genome into overlapping 50 kbp sliding windows and build barcode-aware local-assembly read sets for structural variant calling, as implemented in Blackbird for synthetic long read (SLR) and hybrid SLR+long-read SV detection.

## When to use

Use when preparing medium-size (50–10,000 bp) insertion/deletion SV calls from position-sorted, BX-tagged SLR BAMs (optionally with paired long-read BAM/FASTQ) and a BWA-indexed reference genome.

## When NOT to use

- Whole-genome assembly is desired — Blackbird explicitly avoids computationally expensive whole-genome assembly in favor of local segment assembly
- Target SVs are shorter than 50 bp — Blackbird's primary output out_50.vcf contains only insertion/deletion calls longer than 50 bp, and shorter calls in out.vcf are described as not reliable
- The SLR bam lacks position-sorted, indexed BX tags or the reference is not BWA-indexed — required input assumptions are violated

## Inputs

- Position-sorted and indexed SLR read BAM with BX barcode tags (via --bam/-b)
- BWA-indexed reference genome FASTA (via --reference/-r)
- Optional position-sorted and indexed long-read BAM paired with matching FASTQ (hybrid mode, --long-bam/-l and --long-read/-m)
- Optional regions file with 'chr start end' rows (--regions)

## Outputs

- Position-sorted VCF of insertion and deletion calls longer than 50 bp (out_50.vcf)
- VCF of additional less-reliable calls (out.vcf)
- Per-segment local assembly contig sets (retained with the hidden -d option for inspection)

## How to apply

Divide the BWA-indexed reference genome into overlapping 50 kbp sliding-window segments so events near boundaries fall fully within at least one window. For each segment s, build the barcode list B(s) by keeping SLR barcodes with at least 3 read pairs aligning to the segment whose leftmost-to-rightmost aligned read span exceeds 5 kbp. From the position-sorted, BX-tagged SLR bam, extract all reads aligning to the segment plus all reads sharing any barcode in M(B(s)) to capture barcode-linked reads outside the window. In hybrid mode, additionally supply a position-sorted, indexed long-read bam paired with its matching FASTQ via the --long-bam/-l and --long-read/-m options. Run the barcode- and long-read-aware modified SPAdes assembler (spades_for_blackbird fork) on each segment's read set, using the hidden -d flag to retain assembly folders so the contigs feeding SV calling can be inspected.

## Related tools

- **Blackbird** (Segments the genome into barcode-aware sliding windows, collects barcode-linked reads per segment, and orchestrates local assembly and SV calling) — https://github.com/1dayac/Blackbird
- **SPAdes (spades_for_blackbird)** (Modified barcode- and long-read-aware assembler run per 50 kbp segment to produce local contig sets) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the indexed reference genome required by Blackbird's alignment-based stage)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s spades_for_blackbird/assembler/spades.py -d
```

## Evaluation signals

- Assembly folders containing per-segment contigs are produced and remain available for inspection when run with the hidden -d option
- out_50.vcf is emitted as a position-sorted VCF containing only insertion and deletion calls longer than 50 bp
- Each segment's barcode list B(s) retains only barcodes with ≥3 read pairs spanning more than 5 kbp across the segment
- Sliding-window segments are 50 kbp with overlap, so boundary-proximal SVs are fully contained in at least one segment
- In hybrid mode, --long-bam and --long-read are supplied as a consistent paired long-read BAM/FASTQ set

## Limitations

- Calls in out.vcf (shorter than 50 bp) are described as not reliable; only out_50.vcf calls longer than 50 bp should be trusted
- SPAdes-based assembly is not intended for large mammalian-size genomes at whole-genome scale; Blackbird mitigates this via local segment assembly only
- The README notes default single-thread operation — more threads (--threads/-t) are beneficial for throughput
- The 50 kbp window and 5 kbp barcode-span criteria are SLR/10X-specific; other linked-read chemistries may need recalibrated thresholds

## Evidence

- [abstract] abstract: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments, integrating long reads to improve"
- [methods] methods: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [readme] readme: "Blackbird is a novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads"

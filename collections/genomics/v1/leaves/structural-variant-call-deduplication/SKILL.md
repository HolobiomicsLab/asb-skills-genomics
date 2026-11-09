---
name: structural-variant-call-deduplication
description: Use when locally assembled contigs (e.g. from Blackbird's barcode-aware
  sliding-window SPAdes assembly) have been aligned to a reference with minimap2 and
  raw CIGAR-derived indel events must be deduplicated and filtered into a reliable
  VCF.
when_to_use_negative:
- The goal is whole-genome-assembly-based SV calling — Blackbird deliberately uses
  a barcode-aware sliding-window local assembly instead of a whole-genome assembly
- Only short-read sequencing data are available without SLR barcodes or long reads
  — most SVs in the 50–10,000 bp range cannot be resolved with short reads
- Variants shorter than 50 bp are the target — they are routed to out.vcf, which Blackbird
  reports as not reliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: End-to-end SV caller that performs the local assembly, minimap2 alignment,
    CIGAR indel calling, deduplication, 50 bp filtering, and position-sorted VCF output
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Base codebase of Blackbird; SPAdes for Blackbird provides spades.py used for
    the local segment assembly whose contigs feed SV calling
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the required index of the reference genome that contigs are aligned
    against
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:853fd76258b85524cee689f3fe563b763e85c66483eff3fd9776e0402d4c18c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# structural-variant-call-deduplication

## Summary

Deduplicates insertion and deletion calls produced by Blackbird's SV identification stage by collapsing variants sharing an identical breakpoint and event length, then filters to indels longer than 50 bp and writes a position-sorted VCF.

## When to use

Use when locally assembled contigs (e.g. from Blackbird's barcode-aware sliding-window SPAdes assembly) have been aligned to a reference with minimap2 and raw CIGAR-derived indel events must be deduplicated and filtered into a reliable VCF.

## When NOT to use

- The goal is whole-genome-assembly-based SV calling — Blackbird deliberately uses a barcode-aware sliding-window local assembly instead of a whole-genome assembly
- Only short-read sequencing data are available without SLR barcodes or long reads — most SVs in the 50–10,000 bp range cannot be resolved with short reads
- Variants shorter than 50 bp are the target — they are routed to out.vcf, which Blackbird reports as not reliable

## Inputs

- Position-sorted and indexed SLR read BAM with BX tags (Blackbird --bam)
- BWA-indexed reference genome FASTA (Blackbird -r/--rerefence)
- Locally assembled barcode-aware contigs from the SPAdes-derived Blackbird assembler
- minimap2 contig-to-reference alignments with CIGAR strings

## Outputs

- out_50.vcf — position-sorted VCF containing only insertion and deletion calls longer than 50 bp
- out.vcf — VCF of all other (less reliable) calls

## How to apply

After the barcode-aware locally assembled contigs are aligned to the BWA-indexed reference genome with minimap2, parse each contig-to-reference alignment CIGAR string to extract insertion and deletion events within the assembled segments. Collapse events that share an identical breakpoint and event length to remove duplicates arising from multiple contigs or windows. Retain only insertion and deletion calls longer than 50 bp, sort the remaining calls by genomic position, and write them to out_50.vcf in the user-specified output folder. Calls that fall below the 50 bp threshold are written to out.vcf, which Blackbird marks as not reliable. In Blackbird this stage runs automatically on the segments assembled from SLR reads (optionally integrating long reads in hybrid mode).

## Related tools

- **Blackbird** (End-to-end SV caller that performs the local assembly, minimap2 alignment, CIGAR indel calling, deduplication, 50 bp filtering, and position-sorted VCF output) — https://github.com/1dayac/Blackbird
- **SPAdes** (Base codebase of Blackbird; SPAdes for Blackbird provides spades.py used for the local segment assembly whose contigs feed SV calling) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the required index of the reference genome that contigs are aligned against)

## Examples

```
./assembler/bin/blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /path/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the user-specified output folder and is a valid, position-sorted VCF
- Every record in out_50.vcf is an insertion or deletion call longer than 50 bp
- No two retained calls share an identical breakpoint and event length (deduplication invariant)
- Calls shorter than 50 bp appear only in out.vcf, not in out_50.vcf
- SV calls match the reference set on HG002 GIAB benchmarking (F1 ~0.835 deletions / 0.808 insertions at 5x long-read coverage)

## Limitations

- Calls written to out.vcf are explicitly 'not reliable'; only the >50 bp indels in out_50.vcf are considered trustworthy
- Blackbird targets medium-size events (50–10,000 bp); SVs outside this range are not the intended target
- Event deduplication only collapses identical breakpoint-and-length pairs — near-duplicate events at slightly shifted breakpoints may persist

## Evidence

- [readme] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] out.vcf contains all other calls, though they are not reliable.: "`out.vcf` contains all other calls, though they are not reliable."
- [methods] Blackbird assembles the genome into segments and calls insertions and deletions in these segments.: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments."
- [abstract] novel integrated alignment- and local-assembly-based algorithm hat employs the barcode information encoded in SLR reads: "We propose a novel hybrid alignment- and local-assembly-based algorithm, Blackbird, that uses synthetic long reads and low-coverage long reads to improve structural variant detection."
- [intro] Blackbird uses a barcode-aware sliding window approach to assemble small segments of the target genome and sensitively call SVs in these segments: "Without the need for a computationally expensive whole genome assembly, Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs"

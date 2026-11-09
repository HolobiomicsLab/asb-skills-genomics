---
name: barcode-aware-genome-segmentation-sliding-window
description: Use when performing local assembly of medium-size SV regions (50–10,000
  bp) from position-sorted, BX-tagged SLR BAMs, and you must define 50 kbp segments
  and decide which barcodes support each segment before assembly.
when_to_use_negative:
- The goal is a whole-genome assembly — Blackbird explicitly avoids whole-genome assembly
  in favor of local sliding-window segments
- The SLR BAM is not position-sorted/indexed or lacks BX barcode tags, so B(s) selection
  and barcode-linked read extraction cannot be performed
- Only short reads without barcodes are available and no SLR data exists — Blackbird
  requires SLR barcodes for this stage
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Blackbird
  role: Implements the sliding-window segmentation, B(s) barcode selection (≥3 read
    pairs spanning >5 kbp), and barcode-linked read collection feeding local assembly
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes (spades_for_blackbird fork)
  role: Barcode- and long-read-aware modified assembler run on each segment's collected
    read set; path supplied via -s/--spades
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the BWA-indexed reference genome used to align SLR read pairs to
    segments for barcode assignment
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:01882048794f464aa3198f68cb39bf62b54c41144914816d6d97e6f3c4425fdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes (spades_for_blackbird fork)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# barcode-aware-genome-segmentation-sliding-window

## Summary

Blackbird's local assembly stage: instead of whole-genome assembly, the BWA-indexed reference is divided into overlapping 50 kbp sliding-window segments, and for each segment a barcode list B(s) of synthetic long-read (SLR) barcodes is built to pull in barcode-linked reads for a modified SPAdes (spades_for_blackbird) assembly.

## When to use

Use when performing local assembly of medium-size SV regions (50–10,000 bp) from position-sorted, BX-tagged SLR BAMs, and you must define 50 kbp segments and decide which barcodes support each segment before assembly.

## When NOT to use

- The goal is a whole-genome assembly — Blackbird explicitly avoids whole-genome assembly in favor of local sliding-window segments
- The SLR BAM is not position-sorted/indexed or lacks BX barcode tags, so B(s) selection and barcode-linked read extraction cannot be performed
- Only short reads without barcodes are available and no SLR data exists — Blackbird requires SLR barcodes for this stage

## Inputs

- Position-sorted, indexed SLR read BAM with BX barcode tags (e.g. 10X NA24385 GRCh37 phased possorted bam)
- BWA-indexed reference genome (e.g. GRCh37/hg19 FASTA)
- Optional position-sorted, indexed long-read BAM paired with a FASTQ of the same long reads (hybrid mode)
- Optional regions file with 'chr start end' rows restricting SV calling

## Outputs

- Per-segment local contig sets from the modified SPAdes assembler
- Assembly folders retained per segment when the hidden -d option is used
- Downstream position-sorted out_50.vcf with insertion and deletion calls longer than 50 bp

## How to apply

Divide the BWA-indexed reference genome into overlapping 50 kbp sliding-window segments (10 kbp overlap) so events near boundaries fall fully inside at least one segment. For each segment s, build the barcode list B(s) by retaining an SLR barcode b only if at least three read pairs with that barcode align to the segment AND the distance between the leftmost and rightmost aligned reads exceeds 5 kbp; discard barcodes failing these thresholds. From the position-sorted BX-tagged SLR BAM, extract all reads aligning to the segment plus all reads sharing any barcode in the barcode-to-reads mapping M(B(s)), capturing barcode-linked reads outside the window. Optionally merge a position-sorted long-read BAM with its matching FASTQ (paired --long-bam/--long-read options) in hybrid mode. Run the barcode- and long-read-aware modified SPAdes assembler (spades_for_blackbird fork) on the collected per-segment read set, using the hidden -d flag to retain assembly folders so the contigs used for SV calling can be inspected.

## Related tools

- **Blackbird** (Implements the sliding-window segmentation, B(s) barcode selection (≥3 read pairs spanning >5 kbp), and barcode-linked read collection feeding local assembly) — https://github.com/1dayac/Blackbird
- **SPAdes (spades_for_blackbird fork)** (Barcode- and long-read-aware modified assembler run on each segment's collected read set; path supplied via -s/--spades) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the BWA-indexed reference genome used to align SLR read pairs to segments for barcode assignment)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /path/to/spades_for_blackbird/assembler/spades.py -d
```

## Evaluation signals

- Each segment's B(s) contains only barcodes with at least 3 aligned read pairs whose leftmost-to-rightmost aligned read distance exceeds 5 kbp
- Every candidate SV near a segment boundary is fully contained in at least one 50 kbp window due to the 10 kbp overlap
- Extracted per-segment read sets include reads sharing barcodes in M(B(s)) even when those reads fall outside the window
- With the hidden -d flag, assembly folders containing per-segment contigs are produced and remain available for inspection
- The full run emits a position-sorted out_50.vcf restricted to insertions and deletions longer than 50 bp

## Limitations

- The paper reports no numeric acceptance threshold for this intermediate assembly stage; validation relies on producing inspectable per-segment contig folders
- out.vcf contains all other calls but they 'are not reliable'; only out_50.vcf calls longer than 50 bp are the intended output
- The hidden -n option ('Don't collect poorly aligned reads') changes barcode/read collection behavior; default collection differs if it is not set
- SPAdes was 'initially designed for small genomes' and is not intended for mammalian-size whole genomes — this is why local segments rather than WGA are used

## Evidence

- [abstract] Sliding-window barcode-aware segmentation rationale: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments"
- [methods] Segmentation method statement: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [readme] Segment assembly output statement: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments."
- [readme] Hidden -d option for retaining assembly folders: "`-d` - Don't delete assembly folders. It allows to inspect contigs that were used for SV calling"
- [readme] Required BX-tagged SLR BAM input: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"

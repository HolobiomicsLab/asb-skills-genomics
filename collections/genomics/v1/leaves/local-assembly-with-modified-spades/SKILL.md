---
name: local-assembly-with-modified-spades
description: Use when calling medium-size structural variants (50–10,000 bp) from
  position-sorted, BX-tagged SLR bams (optionally plus low-coverage long reads) where
  a full whole-genome assembly is unnecessary or too expensive.
when_to_use_negative:
- Input is short-read-only data without SLR barcode tags (BX) — Blackbird relies on
  encoded SLR barcode information
- Goal is a whole-genome assembly; Blackbird explicitly avoids whole-genome assembly
  and only assembles small segments for SV calling
- The event of interest is shorter than 50 bp — reliable calls are emitted only for
  indels longer than 50 bp in out_50.vcf
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: Blackbird
  role: Performs barcode-aware sliding-window segmentation, per-segment read collection
    via barcode lists B(s), and SV calling from assembled contigs
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for Blackbird (spades_for_blackbird)
  role: Barcode- and long-read-aware modified SPAdes assembler executed per segment
    to produce the local contig set
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the indexed reference genome against which segments and reads are
    aligned
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:3073ee5cba912b0f1112bd0d5cc0e86fe7d9b76dc2aa6c341eb48ed02a2598e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes
  - SPAdes for Blackbird (spades_for_blackbird)
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# local-assembly-with-modified-spades

## Summary

Barcode-aware sliding-window local assembly of synthetic long-read (SLR) datasets for structural variant detection, implemented in Blackbird with a barcode- and long-read-aware SPAdes fork. Segments the BWA-indexed reference into 50 kbp overlapping windows, collects barcode-linked reads per segment, and assembles each segment locally to call insertions and deletions in the 50–10,000 bp range.

## When to use

Use when calling medium-size structural variants (50–10,000 bp) from position-sorted, BX-tagged SLR bams (optionally plus low-coverage long reads) where a full whole-genome assembly is unnecessary or too expensive.

## When NOT to use

- Input is short-read-only data without SLR barcode tags (BX) — Blackbird relies on encoded SLR barcode information
- Goal is a whole-genome assembly; Blackbird explicitly avoids whole-genome assembly and only assembles small segments for SV calling
- The event of interest is shorter than 50 bp — reliable calls are emitted only for indels longer than 50 bp in out_50.vcf

## Inputs

- Position-sorted and indexed SLR read BAM with BX barcode tags (--bam/-b, required)
- BWA-indexed reference genome FASTA (--reference/-r, required)
- Optional position-sorted and indexed long-read BAM (--long-bam/-l)
- FASTQ file with the same long reads (--long-read/-m, must be paired with -l)
- Optional regions file with 'chr start end' rows (--regions/-g)

## Outputs

- Per-segment local contig sets assembled by the modified SPAdes
- out_50.vcf: position-sorted VCF of insertion and deletion calls longer than 50 bp
- out.vcf: additional (less reliable) calls
- Assembly folders retained for inspection when the hidden -d option is used

## How to apply

Divide the BWA-indexed reference genome into overlapping 50 kbp sliding-window segments so boundary events fall fully within at least one segment. For each segment s, build its barcode list B(s) by retaining SLR barcodes with at least 3 read pairs aligning to the segment whose leftmost-to-rightmost aligned read span exceeds 5 kbp. Extract from the position-sorted, BX-tagged SLR bam all reads aligning to the segment plus all reads sharing any barcode in M(B(s)), capturing barcode-linked reads outside the window. In hybrid mode, merge the position-sorted long-read bam with its matching FASTQ via the paired --long-bam/-l and --long-read/-m options. Run the modified assembler (spades.py from the spades_for_blackbird repository, passed via --spades/-s) on the collected per-segment read set to produce local contigs per segment, which downstream SV calling consumes to emit insertions and deletions longer than 50 bp in out_50.vcf.

## Related tools

- **Blackbird** (Performs barcode-aware sliding-window segmentation, per-segment read collection via barcode lists B(s), and SV calling from assembled contigs) — https://github.com/1dayac/Blackbird
- **SPAdes for Blackbird (spades_for_blackbird)** (Barcode- and long-read-aware modified SPAdes assembler executed per segment to produce the local contig set) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the indexed reference genome against which segments and reads are aligned)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- Assembly folders containing per-segment contigs are produced and retained (via the hidden -d option) and can be inspected for barcode-linked local sequence
- out_50.vcf is a position-sorted VCF containing only insertion and deletion calls longer than 50 bp
- Each assembled segment's read set reflects barcode support: every retained barcode in B(s) has at least 3 read pairs spanning more than 5 kbp across the segment
- In hybrid mode, the long-read BAM and FASTQ inputs are paired consistently (-l with -m) and long reads are integrated into the per-segment assemblies
- On the HG002 GIAB benchmark, hybrid-mode results should be comparable to PBSV/Sniffles2 at 10x PacBio HiFi while Blackbird uses ~5x long-read coverage (F1 ≈ 0.835 deletions, 0.808 insertions)

## Limitations

- out.vcf calls (other than those in out_50.vcf) are described as 'not reliable'
- Barcode selection discards barcodes with fewer than 3 spanning read pairs or spans ≤ 5 kbp, which may miss low-support events
- SPAdes was initially designed for small genomes and 'is not intended for larger genomes'; Blackbird mitigates this by assembling only small segments, not whole genomes
- Low long-read coverage affects hybrid-mode performance; published benchmarking used ~5x coverage

## Evidence

- [methods] Blackbird uses a barcode-aware sliding window approach: "Blackbird uses a barcode-aware
sliding window approach to assemble small segments of the target genome and sensitively call
SVs in these segments."
- [readme] Blackbird assembles genome segments and calls indels: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments."
- [readme] README options for SLR bam and reference: "`--bam` or `-b` [required] - position-sorted and indexed SLR Read bam-file with BX tags"
- [readme] Hidden -d option retains assembly folders: "`-d` - Don't delete assembly folders. It allows to inspect contigs that were used for SV calling"
- [readme] out_50.vcf output description: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [abstract] Sliding window and barcode information in abstract: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments"
- [methods] SPAdes fork requirement: "you should install and compile [SPAdes for blackbird](https://github.com/1dayac/spades_for_blackbird)"

---
name: sliding-window-local-assembly-sv-calling
description: Use when input is a position-sorted, indexed SLR bam with BX tags plus
  a BWA-indexed reference, and the goal is insertion/deletion calls in the 50-10,000
  bp range without whole-genome assembly.
when_to_use_negative:
- The goal requires resolving SVs beyond the 50-10,000 bp range that Blackbird targets,
  or whole-genome-scale assembly is expected
- Input reads are standard short reads without SLR barcode (BX tag) information, since
  the method depends on barcode-aware sliding-window assembly
- No BWA-indexed reference genome is available, since the required reference must
  be BWA-indexed
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: 'Main pipeline: barcode-aware sliding-window local assembly of genome segments,
    contig-to-reference alignment, and insertion/deletion calling from CIGAR strings'
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for Blackbird
  role: Forked SPAdes codebase providing spades.py used by Blackbird for local segment
    assembly
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Required to index the reference genome before SV calling
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:b4db935342d951d78fadc68c3db605309dd25511dd7fe476e9069f8626cb41da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes for Blackbird
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sliding-window-local-assembly-sv-calling

## Summary

Detect medium-size (50-10,000 bp) insertions and deletions by barcode-aware sliding-window local assembly of synthetic long reads (optionally hybrid with long reads), aligning locally assembled contigs to the reference and calling SVs from CIGAR strings, as implemented in Blackbird.

## When to use

Use when input is a position-sorted, indexed SLR bam with BX tags plus a BWA-indexed reference, and the goal is insertion/deletion calls in the 50-10,000 bp range without whole-genome assembly.

## When NOT to use

- The goal requires resolving SVs beyond the 50-10,000 bp range that Blackbird targets, or whole-genome-scale assembly is expected
- Input reads are standard short reads without SLR barcode (BX tag) information, since the method depends on barcode-aware sliding-window assembly
- No BWA-indexed reference genome is available, since the required reference must be BWA-indexed

## Inputs

- Position-sorted and indexed SLR read bam-file with BX tags (--bam)
- BWA-indexed reference genome (--reference)
- Optional: position-sorted and indexed long-read bam (--long-bam) paired with a FASTQ of the same long reads (--long-read)
- Optional: regions file with 'chr start end' rows (--regions)

## Outputs

- out_50.vcf: position-sorted VCF containing insertion and deletion calls longer than 50 bp
- out.vcf: all other (less reliable) calls

## How to apply

Run Blackbird on the position-sorted, BX-tagged SLR bam against a BWA-indexed reference; it uses a barcode-aware sliding window to locally assemble small genome segments (SPAdes-derived codebase; use spades.py from spades_for_blackbird). Optionally enable hybrid mode by pairing --long-bam with a FASTQ of the same long reads, which improves results while requiring less long-read coverage than tools like PBSV or Sniffles2. Align the assembled contigs to the reference with minimap2 and parse each contig-to-reference CIGAR string to extract insertion and deletion events. Deduplicate events sharing identical breakpoint and length, retain only calls longer than 50 bp, sort by genomic position, and write out_50.vcf.

## Related tools

- **Blackbird** (Main pipeline: barcode-aware sliding-window local assembly of genome segments, contig-to-reference alignment, and insertion/deletion calling from CIGAR strings) — https://github.com/1dayac/Blackbird
- **SPAdes for Blackbird** (Forked SPAdes codebase providing spades.py used by Blackbird for local segment assembly) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Required to index the reference genome before SV calling)

## Examples

```
/home/dmm2017/Blackbird/assembler/bin/blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r refdata-hg19-2.1.0/fasta/genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s /home/dmm2017/spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the user-specified output folder and is a valid position-sorted VCF
- All records in out_50.vcf are insertion or deletion calls longer than 50 bp; shorter/unreliable calls appear only in out.vcf
- No duplicate SV events remain: variants sharing an identical breakpoint and event length are collapsed
- On HG002 GIAB, hybrid-mode results should be comparable to PBSV/Sniffles2 at 10x PacBio HiFi, with Blackbird reaching F1 ~0.835 (deletions) and ~0.808 (insertions) at only 5x long-read coverage
- Hidden option -d keeps assembly folders so contigs used for SV calling can be inspected

## Limitations

- Calls reported in out.vcf (below the 50 bp threshold) are explicitly 'not reliable'
- Only insertion and deletion calls longer than 50 bp are written to the primary output; other SV classes are not emitted
- The --long-bam option must be paired with --long-read FASTQ of the same long reads; unpaired usage is invalid
- SV calling is restricted to regions provided in the optional regions file if used; otherwise whole genome

## Evidence

- [abstract] Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments, integrating long reads to improve: "Instead of relying on whole-genome assembly, Blackbird uses a sliding window approach and synthetic long-read barcode information to assemble local segments, integrating long reads to improve"
- [methods] `out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] Position-sorted and indexed long read file (optional, should be paired with long-read option): "`--long-bam` or `-l` - Position-sorted and indexed long read file (optional, should be paired with long-read option)"
- [readme] out.vcf contains all other calls, though they are not reliable.: "`out.vcf` contains all other calls, though they are not reliable."

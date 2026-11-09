---
name: contig-to-reference-alignment
description: Use when locally assembled contigs (e.g., from Blackbird/SPAdes sliding-window
  assembly of synthetic long-read barcode data) must be converted into insertion/deletion
  calls against a reference genome.
when_to_use_negative:
- You already have long reads at ~10x coverage and can call SVs directly with a long-read
  caller such as PBSV or Sniffles2 instead of local assembly
- The goal is whole-genome assembly rather than segment-local insertion/deletion calling;
  Blackbird explicitly avoids whole-genome assembly
- 'Only short-read data are available: most SVs in the 50-10,000 bp range cannot be
  resolved with short-read sequencing'
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: Blackbird
  role: Runs the sliding-window local assembly, contig-to-reference alignment, CIGAR-based
    indel calling, deduplication, 50 bp filtering, and VCF output
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes for Blackbird
  role: Forked SPAdes assembler providing spades.py used for the local assembly of
    genome segments whose contigs are then aligned
  repo: https://github.com/1dayac/spades_for_blackbird
- name: minimap2
  role: Long-read-capable aligner used to align locally assembled contigs to the reference
    genome; indels are called from its CIGAR output
- name: BWA
  role: Used to index the reference genome required by the --rerefence/-r input
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:8a2b7ace41257f3b5354b7aa38aa194a82ecdbedec2188c607753eb27233099c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/Blackbird
  tools:
  - BWA
  - Blackbird
  - SPAdes for Blackbird
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Contig-to-reference alignment for SV identification

## Summary

Aligns Blackbird's barcode-aware locally assembled contigs against a BWA-indexed reference genome using minimap2, parses the CIGAR strings to call insertions and deletions within assembled segments, and emits a position-sorted VCF of events longer than 50 bp.

## When to use

Use when locally assembled contigs (e.g., from Blackbird/SPAdes sliding-window assembly of synthetic long-read barcode data) must be converted into insertion/deletion calls against a reference genome.

## When NOT to use

- You already have long reads at ~10x coverage and can call SVs directly with a long-read caller such as PBSV or Sniffles2 instead of local assembly
- The goal is whole-genome assembly rather than segment-local insertion/deletion calling; Blackbird explicitly avoids whole-genome assembly
- Only short-read data are available: most SVs in the 50-10,000 bp range cannot be resolved with short-read sequencing

## Inputs

- Barcode-aware locally assembled contigs from Blackbird's sliding-window segments (SPAdes-derived assembler)
- BWA-indexed reference genome FASTA (e.g., GRCh37/hg19 genome.fa)
- Position-sorted and indexed SLR read BAM with BX tags
- Optional position-sorted, indexed long-read BAM and matching long-read FASTQ for hybrid mode

## Outputs

- out_50.vcf: position-sorted VCF containing only insertion and deletion calls longer than 50 bp
- out.vcf: VCF with all other calls (not reliable)
- Optionally retained per-segment assembly folders (hidden -d option) for inspecting contigs used in SV calling

## How to apply

First run Blackbird's barcode-aware sliding window local assembly (forked from the SPAdes codebase) to produce contigs for genome segments, using a position-sorted, indexed SLR BAM with BX tags and a BWA-indexed reference. Align each contig against the reference genome with minimap2, then parse each contig-to-reference CIGAR string to extract insertion and deletion events within the assembled segments. Deduplicate events by collapsing variants sharing an identical breakpoint and event length, then filter to retain only insertions and deletions longer than 50 bp. Finally, sort the retained calls by genomic position and write them to out_50.vcf in the user-specified output folder; less reliable shorter calls go to out.vcf.

## Related tools

- **Blackbird** (Runs the sliding-window local assembly, contig-to-reference alignment, CIGAR-based indel calling, deduplication, 50 bp filtering, and VCF output) — https://github.com/1dayac/Blackbird
- **SPAdes for Blackbird** (Forked SPAdes assembler providing spades.py used for the local assembly of genome segments whose contigs are then aligned) — https://github.com/1dayac/spades_for_blackbird
- **minimap2** (Long-read-capable aligner used to align locally assembled contigs to the reference genome; indels are called from its CIGAR output)
- **BWA** (Used to index the reference genome required by the --rerefence/-r input)

## Examples

```
blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the user-specified output folder and is a valid, position-sorted VCF
- All records in out_50.vcf are insertion or deletion calls longer than 50 bp
- No two retained calls share an identical breakpoint and event length (deduplication invariant)
- Contigs used for SV calling can be inspected by re-running with the hidden -d option, confirming each call traces to an assembled segment
- On HG002 GIAB hybrid mode, results should be comparable to PBSV/Sniffles2 at 10x PacBio HiFi coverage when using only ~5x long-read coverage (F1 ~0.835 deletions, ~0.808 insertions)

## Limitations

- Calls written to out.vcf (below 50 bp or otherwise filtered) are described as 'not reliable'
- Blackbird targets medium-size events (50-10,000 bp); SVs outside this range are not the focus
- Hybrid mode requires the long-read BAM and FASTQ options to be supplied as a pair
- A typo exists in the documented CLI flag for the reference (--rerefence/-r), which is also reused for the optional --regions file

## Evidence

- [readme] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] Blackbird assembles the genome into segments and calls insertions and deletions in these segments.: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments."
- [readme] BWA-indexed reference genome is the required -r input.: "`--rerefence` or `-r` [required] - BWA-indexed reference genome"
- [methods] Blackbird uses barcode-aware sliding window assembly of small segments to sensitively call SVs.: "Blackbird uses a barcode-aware sliding window approach to assemble small segments of the target genome and sensitively call SVs in these segments."

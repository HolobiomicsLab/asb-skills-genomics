---
name: cigar-string-indel-extraction
description: Use when locally assembled contigs must be converted into insertion/deletion
  variant calls by aligning them to a reference genome and mining the CIGAR alignment
  operations.
when_to_use_negative:
- Input is raw short-read sequencing without locally assembled contigs — the CIGAR-based
  indel extraction operates on contig-to-reference alignments, not short-read pileups
- The goal is whole-genome assembly-based SV calling — Blackbird deliberately avoids
  expensive whole-genome assembly in favor of a barcode-aware sliding window approach
- Variants smaller than the 50 bp structural variant threshold are the target — these
  are routed to out.vcf and considered unreliable by the tool
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: Blackbird
  role: 'Hosts the SV identification stage: aligns assembled contigs to the reference
    and calls insertions/deletions from CIGAR strings, then writes the position-sorted
    VCF'
  repo: https://github.com/1dayac/Blackbird
- name: SPAdes
  role: Blackbird was forked from the SPAdes codebase; spades_for_blackbird performs
    the barcode-aware local assembly whose contigs feed the CIGAR-based indel extraction
  repo: https://github.com/1dayac/spades_for_blackbird
- name: BWA
  role: Provides the index for the reference genome supplied via the --rerefence/-r
    option against which contigs are aligned
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioadv/vbaf151
    title: 'Blackbird: structural variant detection using synthetic and low-coverage
      long-reads'
  source_content_hash: sha256:a08895cb5d0af11dd4138eb98bb20da55ad8fb9ab96db2f32c1f09c9dc2fe02b
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
yes_rate: 0.5
n_claims: 8
---

# cigar-string-indel-extraction

## Summary

Extracts insertion and deletion calls by parsing contig-to-reference alignment CIGAR strings, as done in Blackbird's SV identification stage where locally assembled (barcode-aware, SPAdes-derived) contigs are aligned to the reference with minimap2.

## When to use

Use when locally assembled contigs must be converted into insertion/deletion variant calls by aligning them to a reference genome and mining the CIGAR alignment operations.

## When NOT to use

- Input is raw short-read sequencing without locally assembled contigs — the CIGAR-based indel extraction operates on contig-to-reference alignments, not short-read pileups
- The goal is whole-genome assembly-based SV calling — Blackbird deliberately avoids expensive whole-genome assembly in favor of a barcode-aware sliding window approach
- Variants smaller than the 50 bp structural variant threshold are the target — these are routed to out.vcf and considered unreliable by the tool

## Inputs

- Barcode-aware locally assembled contigs from Blackbird's sliding-window assembly (SPAdes for blackbird)
- BWA-indexed reference genome FASTA
- Position-sorted and indexed SLR read BAM with BX tags (--bam)

## Outputs

- out_50.vcf: position-sorted VCF containing insertion and deletion calls longer than 50 bp
- out.vcf: VCF of all other (shorter, not reliable) calls

## How to apply

Align the barcode-aware locally assembled contigs (produced by the SPAdes-derived Blackbird assembler) against the BWA-indexed reference genome using the long-read-capable aligner minimap2. Parse each contig-to-reference alignment's CIGAR string to extract insertion (I) and deletion (D) events within the assembled segments, since Blackbird calls insertions and deletions in these segments. Deduplicate the called events by collapsing variants that share an identical breakpoint and event length. Filter to retain only insertion and deletion calls longer than 50 bp, then sort by genomic position and write them to the position-sorted VCF out_50.vcf in the user-specified output folder. Calls shorter than 50 bp go to out.vcf but are flagged as not reliable by the tool's own documentation.

## Related tools

- **Blackbird** (Hosts the SV identification stage: aligns assembled contigs to the reference and calls insertions/deletions from CIGAR strings, then writes the position-sorted VCF) — https://github.com/1dayac/Blackbird
- **SPAdes** (Blackbird was forked from the SPAdes codebase; spades_for_blackbird performs the barcode-aware local assembly whose contigs feed the CIGAR-based indel extraction) — https://github.com/1dayac/spades_for_blackbird
- **BWA** (Provides the index for the reference genome supplied via the --rerefence/-r option against which contigs are aligned)

## Examples

```
./blackbird -b NA24385.GRCh37.phased_possorted_bam.bam -r genome.fa -o blackbird_10X_chr1 -g genome.txt -t 32 -s spades_for_blackbird/assembler/spades.py
```

## Evaluation signals

- out_50.vcf exists in the user-specified output folder and is a valid, position-sorted VCF
- Every record in out_50.vcf is an insertion or deletion call longer than 50 bp, per the paper's stated output format
- Deduplicated events: no two retained calls share an identical breakpoint and event length
- Contigs used for SV calling can be inspected when running with the hidden -d flag (assembly folders retained), allowing spot-checks of contig alignments underlying calls
- On the HG002 GIAB benchmark, hybrid-mode results should be comparable to PBSV and Sniffles2 run at 10× PacBio Hi-Fi coverage

## Limitations

- Only insertion and deletion calls longer than 50 bp are retained in out_50.vcf; other calls in out.vcf 'are not reliable'
- Blackbird relies on synthetic long-read barcode (BX tag) information, so input BAMs must be position-sorted, indexed, and carry BX tags
- Requires the separately installed and compiled spades_for_blackbird fork; the reference option is spelled '--rerefence' in the manual, a potential CLI pitfall

## Evidence

- [methods] In the SV identification stage, locally assembled contigs are aligned to the reference genome using minimap2, and insertions and deletions are called from the CIGAR string, which encodes the alignment operations between the contig and the reference.: [evidence span withheld — anchor status: not_found]
- [methods] Blackbird assembles the genome into segments and calls insertions and deletions in these segments.: "Blackbird assembles the genome into segments and calls insertions and deletions in these segments"
- [readme] out_50.vcf is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp.: "`out_50.vcf` is a position sorted VCF file, that contains insertion and deletion calls longer than 50 bp."
- [readme] out.vcf contains all other calls, though they are not reliable.: "`out.vcf` contains all other calls, though they are not reliable."
- [readme] `--rerefence` or `-r` [required] - BWA-indexed reference genome: "`--rerefence` or `-r` [required] - BWA-indexed reference genome"

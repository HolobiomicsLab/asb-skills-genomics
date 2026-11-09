---
name: barcode-guided-local-reassembly
description: Use when detecting and genotyping non-reference sequence insertions in
  synthetic long-read (linked-read) datasets where short-read methods fail to resolve
  insertions larger than 300 bp and whole-genome assembly is too expensive.
when_to_use_negative:
- Small insertions whose number of associated barcodes is small and coverage insufficient
  — Novel-X needs a substantial number of distinct barcodes to assemble anchors
- A whole-genome de novo assembly is already available or required for other SV classes
  — whole-genome-assembly-based synthetic long-read methods (e.g. Supernova-based
  approaches) would apply instead
- Complex SV classes beyond NRS insertions — the article notes assembly of other classes
  of complex SVs was not yet explored
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: Novel-X
  role: Snakemake pipeline orchestrating the four-step barcode-guided local reassembly
    workflow
  repo: github.com/1dayac/Novel-X
- name: Velvet
  role: Assembles the unaligned read set U (k=63, compiled via MAXKMERLENGTH=63 for
    10x data) into orphan contigs
  repo: https://github.com/dzerbino/velvet
- name: SPAdes
  role: Reassembles barcode-selected reads (k=77, cov-cutoff=3) to obtain anchors
    for each candidate insertion
- name: Minimap2
  role: Aligns reassembled contigs to the reference genome; used to identify best
    matches of reassembled contigs to orphan contigs and to locate anchors
- name: BLASTN
  role: 'Optional contamination removal: discards non-human orphan contigs against
    nt/nr via the --nt flag'
- name: Samtools
  role: BAM handling for extracting unaligned/poorly-aligned reads and barcode-linked
    reads
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:75e49eddb83b253bc5975ad04e48b838e3743de5a8deb5667eb97ebca6f21430
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTN
  - Minimap2
  - Novel-X
  - SPAdes
  - Samtools
  - Velvet
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# barcode-guided-local-reassembly

## Summary

A four-step local assembly strategy (Novel-X) that detects and places non-reference sequence (NRS) insertions >300 bp from 10x/TELL-Seq/stLFR linked-read data by first assembling unaligned reads into orphan contigs with Velvet, then extracting barcodes B(c), reassembling barcode-linked reads with SPAdes, and locating anchors on the reference with Minimap2 — all without a whole-genome de novo assembly.

## When to use

Use when detecting and genotyping non-reference sequence insertions in synthetic long-read (linked-read) datasets where short-read methods fail to resolve insertions larger than 300 bp and whole-genome assembly is too expensive.

## When NOT to use

- Small insertions whose number of associated barcodes is small and coverage insufficient — Novel-X needs a substantial number of distinct barcodes to assemble anchors
- A whole-genome de novo assembly is already available or required for other SV classes — whole-genome-assembly-based synthetic long-read methods (e.g. Supernova-based approaches) would apply instead
- Complex SV classes beyond NRS insertions — the article notes assembly of other classes of complex SVs was not yet explored

## Inputs

- LongRanger-aligned BAM of synthetic long-read (10x Chromium, TELL-Seq, or stLFR) paired-end reads with barcode tags
- Reference genome FASTA (e.g. GRCh38)

## Outputs

- VCF file (named after the input BAM) containing NRS insertion calls >300 bp with anchor lengths summing to >300 bp, placed on the reference genome

## How to apply

First build the unaligned read set U from a LongRanger-aligned BAM by keeping paired-end reads with at least one unaligned end, MAPQ < 10, or >20% soft-clipped bases, and average base quality > 20; assemble U with Velvet (k-mer length 63, MAXKMERLENGTH=63 for 10x data) to produce orphan contigs, optionally discarding non-human contigs via BLASTN (--nt flag). Second, for each orphan contig align reads in U to it, filter alignments with low quality or ≥20% soft/hard-clipped sequence, collect barcodes B(c), and discard contigs recruiting fewer than t=5 barcodes (default t=5). Third, extract both aligned and unaligned original reads whose barcodes are in B(c) from the original BAM and reassemble each barcode set separately with SPAdes (k=77, cov-cutoff=3), retaining reassembled contigs whose best Minimap2 match is an existing orphan contig. Fourth, align reassembled contigs to the reference (e.g. GRCh38) with Minimap2, apply a QUAST-like filtering that chooses best alignment subsets maximizing continuously covered base pairs, and call insertions where adjacent anchor alignments are close on the reference but far on the contig; output only insertions longer than 300 bp with summed anchor lengths exceeding 300 bp into a VCF named after the input BAM.

## Related tools

- **Novel-X** (Snakemake pipeline orchestrating the four-step barcode-guided local reassembly workflow) — github.com/1dayac/Novel-X
- **Velvet** (Assembles the unaligned read set U (k=63, compiled via MAXKMERLENGTH=63 for 10x data) into orphan contigs) — https://github.com/dzerbino/velvet
- **SPAdes** (Reassembles barcode-selected reads (k=77, cov-cutoff=3) to obtain anchors for each candidate insertion)
- **Minimap2** (Aligns reassembled contigs to the reference genome; used to identify best matches of reassembled contigs to orphan contigs and to locate anchors)
- **BLASTN** (Optional contamination removal: discards non-human orphan contigs against nt/nr via the --nt flag)
- **Samtools** (BAM handling for extracting unaligned/poorly-aligned reads and barcode-linked reads)

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- Output VCF exists in the outdir, is named after the input BAM, and parses as valid VCF
- Every VCF record is an insertion longer than 300 bp whose summed anchor lengths exceed 300 bp
- Demo/expected run on the provided dataset yields a single chr1 record around position 29503 with the insertion sequence present
- No orphan contig with fewer than t=5 recruited barcodes contributes a call
- Reassembled contigs are retained only when their best Minimap2 match is an existing orphan contig

## Limitations

- Novel-X needs a substantial number of distinct barcodes per insertion; for small insertions the number of associated barcodes is usually small and coverage is insufficient for anchor assembly
- A 'filter_target_contig' stage problem is acknowledged in the methods ('There is a problem on filter_target_contig stage at the moment')
- The repository must be cloned recursively (git submodule update --init --recursive) or Novel-X will not work
- A dedicated local assembler for SV detection in synthetic long-read data does not yet exist; a general-purpose local assembler is used as a workaround

## Evidence

- [abstract] We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads: "We propose a novel integrated alignment-based and local assembly-based algorithm, Novel-X, that uses the barcode information encoded in synthetic long reads"
- [methods] NRS insertion assembly step: [evidence span withheld — anchor status: not_found]
- [methods] barcode threshold: [evidence span withheld — anchor status: not_found]
- [methods] SPAdes parameters: "k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences)"
- [methods] output filter: [evidence span withheld — anchor status: not_found]
- [readme] clone instruction: "If you clone repository non-recursively Novel-X will not work. To fix this run from Novel-X folder:"
- [methods] read selection filter: "we choose only reads with the average base quality score > 20"

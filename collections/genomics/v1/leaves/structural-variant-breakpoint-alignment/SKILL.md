---
name: structural-variant-breakpoint-alignment
description: Use when a LongRanger-aligned 10x (or TELL-Seq/stLFR) BAM is available
  and the goal is to discover and genotype non-reference insertions longer than 300
  bp that short-read methods fail to assemble and place.
when_to_use_negative:
- The goal requires assembling full genomes or SV classes other than NRS insertions
  — Novel-X targets NRS insertion discovery and does not perform whole-genome de novo
  assembly nor assembly of other complex SV classes.
- 'Insertions are small (<300 bp): Novel-X needs a substantial number of distinct
  barcodes, and for small insertions the number of associated barcodes is usually
  small and coverage insufficient for anchor assembly.'
- Input is raw FASTQ without barcode-aware alignment — the pipeline expects a LongRanger-aligned
  BAM as its starting point.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
tools:
- name: Novel-X
  role: Integrated alignment-based and local assembly-based Snakemake pipeline that
    detects and genotypes NRS insertions using read alignment signatures and synthetic
    long-read barcode information
  repo: github.com/1dayac/Novel-X
- name: Velvet
  role: Assembles the unaligned/poorly-aligned read set U into orphan contigs (k-mer
    length 63, MAXKMERLENGTH=63 for 10x data)
  repo: https://github.com/dzerbino/velvet
- name: SPAdes
  role: Reassembles reads whose barcodes are in B(c) per barcode set (k=77, cov-cutoff=3)
    to produce anchored insertion contigs
- name: Minimap2
  role: Aligns reassembled contigs to the reference genome to locate insertion anchors
    and detect insertion signatures
- name: BLASTN
  role: 'Optional contamination removal: discards non-human orphan contigs via BLASTN
    against nt/nr (--nt flag)'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:8925c93c9cbadec130a6125d94d1ae02dc93261be6a5127bff88edf0a2646f45
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5882352941176471
n_claims: 17
---

# structural-variant-breakpoint-alignment

## Summary

Detects and places non-reference sequence (NRS) insertions >300 bp from 10x Genomics linked-read data via Novel-X's four-step pipeline: orphan contig assembly, barcode extraction, barcode-guided reassembly with SPAdes, and Minimap2 anchor alignment to the reference, without whole-genome de novo assembly.

## When to use

Use when a LongRanger-aligned 10x (or TELL-Seq/stLFR) BAM is available and the goal is to discover and genotype non-reference insertions longer than 300 bp that short-read methods fail to assemble and place.

## When NOT to use

- The goal requires assembling full genomes or SV classes other than NRS insertions — Novel-X targets NRS insertion discovery and does not perform whole-genome de novo assembly nor assembly of other complex SV classes.
- Insertions are small (<300 bp): Novel-X needs a substantial number of distinct barcodes, and for small insertions the number of associated barcodes is usually small and coverage insufficient for anchor assembly.
- Input is raw FASTQ without barcode-aware alignment — the pipeline expects a LongRanger-aligned BAM as its starting point.

## Inputs

- LongRanger-aligned 10x Genomics BAM with barcode information (also compatible with TELL-Seq and stLFR linked-read data)
- Reference genome FASTA (e.g. GRCh38)

## Outputs

- VCF file of NRS insertions longer than 300 bp with summed anchor lengths exceeding 300 bp, named after the input BAM, written inside the output directory

## How to apply

Run Novel-X as a Snakemake pipeline (clone recursively with git clone --recursive). Step 1: build the unaligned read set U from paired-end reads with at least one unaligned end, mapping quality <10, or >20% soft-clipped bases, keeping only reads with average base quality >20, then assemble U into orphan contigs with Velvet (k-mer length 63, compiled via MAXKMERLENGTH=63 for 10x data). Step 2: for each orphan contig, realign reads from U, discard alignments with low quality or >=20% soft/hard-clipped sequence, collect barcodes B(c), and drop contigs recruiting fewer than t=5 barcodes. Step 3: reassemble, per barcode set, both aligned and unaligned original reads with SPAdes (k=77, cov-cutoff=3), retaining reassembled contigs whose best Minimap2 match is an existing orphan contig. Step 4: align reassembled contigs to the reference (e.g. GRCh38) with Minimap2, apply a QUAST-like filter choosing alignment subsets that maximize continuously covered base pairs, and call insertions where adjacent anchors are close on the reference but far on the contig.

## Related tools

- **Novel-X** (Integrated alignment-based and local assembly-based Snakemake pipeline that detects and genotypes NRS insertions using read alignment signatures and synthetic long-read barcode information) — github.com/1dayac/Novel-X
- **Velvet** (Assembles the unaligned/poorly-aligned read set U into orphan contigs (k-mer length 63, MAXKMERLENGTH=63 for 10x data)) — https://github.com/dzerbino/velvet
- **SPAdes** (Reassembles reads whose barcodes are in B(c) per barcode set (k=77, cov-cutoff=3) to produce anchored insertion contigs)
- **Minimap2** (Aligns reassembled contigs to the reference genome to locate insertion anchors and detect insertion signatures)
- **BLASTN** (Optional contamination removal: discards non-human orphan contigs via BLASTN against nt/nr (--nt flag))

## Examples

```
git clone --recursive git@github.com:1dayac/Novel-X.git
```

## Evaluation signals

- The output VCF exists in the outdir, is named after the input BAM, and parses as valid VCF.
- Every VCF record encodes an insertion longer than 300 bp whose summed anchor lengths exceed 300 bp (thresholds chosen to prevent false calls).
- Demo run expectation: a single chr1 record around position 29503 with the insertion sequence present.
- Orphan contigs that recruited fewer than t=5 barcodes were discarded, and SPAdes reassembly used cov-cutoff=3 to remove low-coverage spurious sequences.
- For benchmarking, compare against the supplementary VCFs (NA12878, NA19240, CHM1, CHM13, HG002 10x Chromium sets) in the novel_insertions_supplementary repository.

## Limitations

- Novel-X requires a substantial number of distinct barcodes per insertion; for small insertions the number of associated barcodes is usually small and coverage is insufficient for anchor assembly.
- A known problem exists on the filter_target_contig stage of the pipeline.
- SuperNova-based comparisons offer no user control over assembler parameters; defaults are maximally tuned for whole human genome diploid assembly.
- Optional non-human contig discarding via BLASTN against nt/nr (--nt flag) is available for contamination removal, similar to Pamir.
- No changelog and no DOI provided in the source repository; future work includes a dedicated local assembler for SV detection in synthetic long-read data.

## Evidence

- [readme] Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information.: "Novel-X detects and genotypes novel sequence insertions in 10X sequencing dataset using non-trivial read alignment signatures and barcode information."
- [methods] We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases: "We choose paired-end reads with at least one end not aligned to the reference genome, or aligned but with a mapping quality below 10, or that have > 20% of soft-clipped bases"
- [methods] k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences): "k=77 (that empirically works better than other standard single k-mer lengths) and cov-cutoff = 3 (to remove low-covered spurious sequences)"
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [intro] All of the existing short-read methods fail to correctly assemble and locate the vast majority of insertions that are > 300 bp.: "All of the existing short-read methods fail to correctly assemble and locate the vast majority of insertions that are > 300 bp."

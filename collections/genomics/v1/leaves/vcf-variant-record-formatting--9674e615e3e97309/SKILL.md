---
name: vcf-variant-record-formatting
description: Use when final insertion candidates from a local-assembly pipeline (e.g.
  orphan contigs with anchor alignments) must be serialized as variant records, with
  length and anchor thresholds applied before writing the VCF.
when_to_use_negative:
- Input is raw orphan contigs that still need BlastN contamination screening, Minimap2
  reference alignment, or the QUAST-like best-alignment-subset filtering — formatting
  to VCF comes only after those filter steps
- The caller's outputs are short variants or already-standardized VCFs from another
  genotyper; this skill applies to Novel-X-style NRS insertion calls with the paper's
  300 bp thresholds
- Small insertions lacking sufficient barcode coverage (Novel-X needs a substantial
  number of distinct barcodes; small insertions rarely yield anchor assemblies and
  would produce few/no records)
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Novel-X
  role: Snakemake-based pipeline whose final stage writes qualifying NRS insertion
    calls (>300 bp) to filtered_insertion_candidates.vcf
  repo: https://github.com/1dayac/novel_insertions
- name: Minimap2
  role: Aligns surviving contigs to the reference genome so anchor positions feeding
    the VCF records can be located
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:721a0b5d9324e55bad139b1496baa3012e58b31b2b9238eb2263b3e99b206a6f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/novel_insertions
  tools:
  - Minimap2
  - Novel-X
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# vcf-variant-record-formatting

## Summary

Formats final non-reference sequence (NRS) insertion calls as VCF records after post-assembly filtering, keeping only insertions longer than 300 bp with summed anchor lengths exceeding 300 bp, as done by Novel-X.

## When to use

Use when final insertion candidates from a local-assembly pipeline (e.g. orphan contigs with anchor alignments) must be serialized as variant records, with length and anchor thresholds applied before writing the VCF.

## When NOT to use

- Input is raw orphan contigs that still need BlastN contamination screening, Minimap2 reference alignment, or the QUAST-like best-alignment-subset filtering — formatting to VCF comes only after those filter steps
- The caller's outputs are short variants or already-standardized VCFs from another genotyper; this skill applies to Novel-X-style NRS insertion calls with the paper's 300 bp thresholds
- Small insertions lacking sufficient barcode coverage (Novel-X needs a substantial number of distinct barcodes; small insertions rarely yield anchor assemblies and would produce few/no records)

## Inputs

- Filtered insertion candidates: orphan contigs with detected insertion signatures (adjacent anchor alignments close on reference, far apart on contig)
- Anchor alignments to the reference genome (e.g. GRCh38, via Minimap2)
- Non-qualifying placements from the QUAST-like filtering step

## Outputs

- filtered_insertion_candidates.vcf — VCF records for qualifying NRS insertions (>300 bp insertion, >300 bp summed anchor lengths)
- Separate file storing non-qualifying insertion placements
- Benchmark-ready VCFs per sample (e.g. NA12878, HG002, CHM13) as distributed in the supplementary repository

## How to apply

After the QUAST-like best-alignment-subset filtering and insertion-signature detection (adjacent anchors close on the reference but far apart on the contig), apply the final call thresholds: only output insertions longer than 300 bp whose sum of anchor lengths exceeds 300 bp — a cutoff the authors set to prevent false calls. Write qualifying calls to a VCF file (in Novel-X, filtered_insertion_candidates.vcf) and store non-qualifying placements in a separate file rather than discarding them, so downstream re-examination is possible. Format each record with the insertion location on the reference genome (e.g. GRCh38) and the assembled non-reference sequence. Validate the output by confirming every record satisfies both the >300 bp insertion-length and >300 bp summed-anchor criteria; on the paper's simulated benchmark this filtering supported precision of 0.999 for Novel-X.

## Related tools

- **Novel-X** (Snakemake-based pipeline whose final stage writes qualifying NRS insertion calls (>300 bp) to filtered_insertion_candidates.vcf) — https://github.com/1dayac/novel_insertions
- **Minimap2** (Aligns surviving contigs to the reference genome so anchor positions feeding the VCF records can be located)

## Evaluation signals

- Every VCF record satisfies both thresholds: insertion length > 300 bp and sum of the anchors' lengths > 300 bp
- Non-qualifying placements are present in the separate storage file, not silently dropped
- On the paper's simulated benchmark, downstream precision reaches 0.999 for Novel-X
- Records reference positions on the chosen reference genome (e.g. GRCh38) and encode assembled non-reference sequence consistent with reassembled contigs
- Output VCFs are benchmark-comparable, as in the supplementary repository where VCFs for NA12878, NA19240, CHM1, CHM13, HG002 (10X, Tell-Seq, stLFR) generated Figures 4-5 and Tables S1-S3

## Limitations

- The 300 bp cutoffs exclude shorter events: the analysis explicitly focuses on insertions longer than 300 bp, and small insertions often lack sufficient barcode coverage for anchor assembly
- Known pipeline issue noted in the code: 'There is a problem on filter_target_contig stage at the moment' — some non-qualifying placements may need manual inspection
- BlastN contamination removal against nt/nr may also drop known viral sequences of interest, so upstream filtering choices are reflected in what reaches the VCF

## Evidence

- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [methods] QUAST-like filtering described in methods: "a filtering procedure that resembles the QUAST (37) procedure, i.e. choosing best alignment subsets from a given alignment set that maximizes the number of continuously covered base pairs"
- [readme] Supplementary VCF files used for benchmarks: [evidence span withheld — anchor status: external_doc]
- [abstract] Novel-X output scale in Polaris cohort: "Novel-X discovered 16 691 NRS insertions of size > 300 bp (total length 18.2 Mb)."

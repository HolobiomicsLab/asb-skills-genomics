---
name: mappability-track-overlap-analysis
description: Use when validating or annotating a set of insertion/SV breakpoints on
  GRCh38 by checking whether each breakpoint lies in a non-uniquely mappable region,
  using UCSC Umap k100 and k50 tracks.
when_to_use_negative:
- The insertion calls lack reference-genome breakpoint coordinates (e.g., unplaced
  contigs), since Umap overlap requires positioned breakpoints
- The reference build is not GRCh38, as the UCSC Umap k100/k50 mappability tracks
  used here are built for GRCh38 and overlap counts will not transfer
- The goal is sensitivity against a truth set rather than characterizing where calls
  land — use simulated/benchmark comparison instead of mappability annotation
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Novel-X
  role: Produces the insertion VCF whose breakpoint positions are intersected with
    mappability tracks
  repo: github.com/1dayac/Novel-X
- name: Umap
  role: UCSC Umap Multitrack k100 and k50 mappability tracks used to flag breakpoints
    in non-uniquely mappable reference regions
- name: RepeatMasker
  role: Screens 300 bp upstream/downstream breakpoint-flanking sequences for repetitive
    reference sequence as a complementary annotation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:da0fb187f6ba8018f79d305dfc07a9b5bc0c3d2d0e928495e35a5976e52e22e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  - RepeatMasker
  - Umap
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mappability-track-overlap-analysis

## Summary

Characterizes detected sequence insertions (e.g., Novel-X NRS insertion VCF records) by intersecting their reference breakpoints with UCSC Umap Multitrack mappability tracks (k100, k50), classifying which calls fall in repetitive, non-uniquely mappable reference regions.

## When to use

Use when validating or annotating a set of insertion/SV breakpoints on GRCh38 by checking whether each breakpoint lies in a non-uniquely mappable region, using UCSC Umap k100 and k50 tracks.

## When NOT to use

- The insertion calls lack reference-genome breakpoint coordinates (e.g., unplaced contigs), since Umap overlap requires positioned breakpoints
- The reference build is not GRCh38, as the UCSC Umap k100/k50 mappability tracks used here are built for GRCh38 and overlap counts will not transfer
- The goal is sensitivity against a truth set rather than characterizing where calls land — use simulated/benchmark comparison instead of mappability annotation

## Inputs

- Novel-X VCF of NRS insertions (>300 bp, anchor-length sum >300 bp) on GRCh38
- UCSC Umap Multitrack mappability tracks for k100 and k50 (GRCh38)
- RepeatMasker annotation/screen of ±300 bp breakpoint-flanking sequences
- GENCODE v.24 gene annotation for optional gene-overlap cross-check

## Outputs

- Per-insertion classification of breakpoint as uniquely vs non-uniquely mappable at k100 and k50
- Counts of insertions in repetitive/non-unique regions per track (e.g., 24 k100, 93 k50, 40 RepeatMasker for CHM1)
- Optionally, counts of insertions overlapping GENCODE-annotated genes/exons

## How to apply

Start from the Novel-X output VCF, which reports only insertions longer than 300 bp whose anchor-length sum exceeds 300 bp (in the CHM1 10x benchmark this yielded 226 insertion records from the LongRanger-aligned, downsampled BAM). Take each reported insertion breakpoint position on the GRCh38 reference and intersect it with the UCSC Umap Multitrack mappability tracks at k=100 and k=50; breakpoints falling in non-uniquely mappable regions are counted separately per k, since mappability is k-dependent (24 insertions for k100 vs 93 for k50 in the CHM1 benchmark). Complement this with a RepeatMasker screen of the 300 bp sequences extracted upstream and downstream of each breakpoint (40 insertions confirmed in repetitive regions for CHM1), and optionally intersect breakpoint positions with GENCODE v.24 annotation to count gene-overlapping calls (111 of 226). Report counts per annotation source so mappability confounding can be separated from true novel sequence.

## Related tools

- **Novel-X** (Produces the insertion VCF whose breakpoint positions are intersected with mappability tracks) — github.com/1dayac/Novel-X
- **Umap** (UCSC Umap Multitrack k100 and k50 mappability tracks used to flag breakpoints in non-uniquely mappable reference regions)
- **RepeatMasker** (Screens 300 bp upstream/downstream breakpoint-flanking sequences for repetitive reference sequence as a complementary annotation)

## Evaluation signals

- The VCF record count matches the expected benchmark size (226 insertions for CHM1 at 0.8-downsampled ~40x coverage)
- The number of insertions in non-uniquely mappable regions matches the reported track-specific counts (24 for k100 and 93 for k50), confirming k-dependence of the overlap
- The RepeatMasker screen of ±300 bp breakpoint flanks reproduces the reported 40 insertions in repetitive reference regions
- Each breakpoint flank extraction yields exactly 300 bp upstream and 300 bp downstream of the reported breakpoint
- GENCODE v.24 intersection reproduces 111/226 gene-overlapping insertions with only two in exonic regions of known protein-coding genes

## Limitations

- Umap k100 and k50 give different overlap counts (24 vs 93 for CHM1), so mappability classification is k-dependent and must be reported per track
- Novel-X needs a substantial number of distinct barcodes; small insertions often lack sufficient coverage for anchor assembly, so the insertion set being annotated may itself be incomplete for small events
- Mappability annotation characterizes where calls fall in the reference but does not by itself validate the correctness of the insertion sequence

## Evidence

- [other] Cross-check insertion placement against UCSC Umap Multitrack mappability tracks for k100 and k50, confirming 24 and 93 insertions in repetitive (non-uniquely mappable) regions, respectively.: [evidence span withheld — anchor status: not_found]
- [other] Extract the 300 bp sequences upstream and downstream of each reported insertion breakpoint and screen them for repeats using RepeatMasker, confirming 40 insertions located in repetitive regions of the reference genome.: [evidence span withheld — anchor status: not_found]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [readme] Results folder contains all vcf files that we used for benchmarks that generated Figures 4-5 and Tables S1-S3 from the article.: [evidence span withheld — anchor status: external_doc]
- [other] Intersect the reported breakpoint positions with GENCODE annotation v.24 and verify that 111 of 226 insertions fall inside known gene sequences, with only two in exonic regions of known protein-coding genes.: [evidence span withheld — anchor status: not_found]

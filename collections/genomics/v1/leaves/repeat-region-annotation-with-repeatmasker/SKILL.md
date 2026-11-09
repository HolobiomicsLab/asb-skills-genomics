---
name: repeat-region-annotation-with-repeatmasker
description: Use when a set of novel-insertion breakpoint coordinates (e.g. from a
  Novel-X VCF) needs classification by whether the surrounding reference sequence
  is repetitive, to characterize detection sensitivity in repeat-rich loci.
when_to_use_negative:
- Input is raw linked-read FASTQ or BAM — RepeatMasker here applies only to extracted
  flanking sequences of already-called insertion breakpoints, not to reads or assemblies
- The insertion set is not from Novel-X's >300 bp anchor-sum-filtered VCF; this skill
  was characterized only on Novel-X output records meeting those thresholds
- The goal is de novo discovery of repeat consensus sequences rather than binary repeat/non-repeat
  annotation of known reference coordinates
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: RepeatMasker
  role: Screens the 300 bp upstream/downstream flanking sequences of each insertion
    breakpoint for reference repeats
- name: Novel-X
  role: Produces the VCF of >300 bp NRS insertions whose breakpoints are annotated
    by this skill
  repo: github.com/1dayac/Novel-X
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkac653
    title: Efficient detection and assembly of non-reference DNA sequences with synthetic
      long reads
  source_content_hash: sha256:3e34e3f1e6195270aef93db5db32c1704ab0cf495da12b0b959ef49437230094
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - LongRanger
  - Novel-X
  - RepeatMasker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Repeat-region annotation with RepeatMasker

## Summary

Screens the 300 bp sequences flanking each non-reference insertion breakpoint (reported in VCF by Novel-X on GRCh38) for repetitive reference sequence using RepeatMasker, to quantify how many detected insertions land in repeat regions of the reference genome.

## When to use

Use when a set of novel-insertion breakpoint coordinates (e.g. from a Novel-X VCF) needs classification by whether the surrounding reference sequence is repetitive, to characterize detection sensitivity in repeat-rich loci.

## When NOT to use

- Input is raw linked-read FASTQ or BAM — RepeatMasker here applies only to extracted flanking sequences of already-called insertion breakpoints, not to reads or assemblies
- The insertion set is not from Novel-X's >300 bp anchor-sum-filtered VCF; this skill was characterized only on Novel-X output records meeting those thresholds
- The goal is de novo discovery of repeat consensus sequences rather than binary repeat/non-repeat annotation of known reference coordinates

## Inputs

- VCF of non-reference sequence (NRS) insertion calls with breakpoint positions on GRCh38 (Novel-X output)
- GRCh38 reference genome FASTA for extracting 300 bp upstream/downstream flanking sequences
- CHM1 10x Genomics linked-read BAM aligned with LongRanger (downsampled to 40x) as the upstream calling input

## Outputs

- Per-insertion annotation of whether 300 bp flanks overlap reference repeats (RepeatMasker output with repeat class/family)
- Count of insertions in repetitive regions (40 of 226 in the CHM1 benchmark)

## How to apply

First run the Novel-X four-step pipeline (Velvet orphan-contig assembly with barcode threshold t = 5, SPAdes reassembly with k = 77 and cov-cutoff = 3, Minimap2 anchor placement) to produce a VCF of insertions longer than 300 bp with anchor-length sums exceeding 300 bp. For each VCF record, extract the 300 bp of reference sequence upstream and the 300 bp downstream of the reported insertion breakpoint. Submit these flanking sequences to RepeatMasker (human species settings) and record which insertion records have at least one flank flagged as repetitive. In the CHM1 10x benchmark (LongRanger-aligned BAM downsampled to 40x, run with novel-x.py run --10x --high-coverage against GRCh38), this yielded 40 of 226 insertions located in repetitive regions of the reference genome, comparable to Umap mappability cross-checks (24 insertions non-uniquely mappable at k100, 93 at k50).

## Related tools

- **RepeatMasker** (Screens the 300 bp upstream/downstream flanking sequences of each insertion breakpoint for reference repeats)
- **Novel-X** (Produces the VCF of >300 bp NRS insertions whose breakpoints are annotated by this skill) — github.com/1dayac/Novel-X

## Examples

```
python novel-x.py run --10x --high-coverage  # CHM1 40x LongRanger BAM vs GRCh38; then RepeatMasker the 300 bp flanks of each VCF breakpoint
```

## Evaluation signals

- The RepeatMasker-positive insertion count for the CHM1 10x callset equals 40 of the 226 VCF records, matching the reported benchmark value
- Each screened sequence is exactly 300 bp extracted immediately upstream and downstream of the VCF breakpoint coordinate
- Every VCF record output by Novel-X satisfies the length filter (insertion > 300 bp, anchor-length sum > 300 bp) before flanking extraction
- Cross-check counts are consistent: 24 insertions non-uniquely mappable at Umap k100, 93 at k50, and 111 of 226 overlapping GENCODE v.24 genes, with the RepeatMasker count as an independent repeat-annotation signal

## Limitations

- Only the reference (GRCh38) side of the breakpoint is screened; the inserted novel sequence itself is not classified by RepeatMasker in this workflow
- Novel-X needs a substantial number of distinct barcodes per insertion, so small insertions often lack sufficient coverage for anchor assembly and never reach the annotation stage
- Mappability-based counts differ from RepeatMasker counts (24 k100 / 93 k50 vs. 40), so repeat annotation and non-unique mappability are non-equivalent notions of 'repetitive region'

## Evidence

- [methods] Extract the 300 bp sequences upstream and downstream of each reported insertion breakpoint and screen them for repeats using RepeatMasker, confirming 40 insertions located in repetitive regions of the reference genome.: [evidence span withheld — anchor status: not_found]
- [methods] We only output insertions longer than 300 bp with the sum of the anchors' lengths exceeding 300 bp to prevent false calls: [evidence span withheld — anchor status: not_found]
- [readme] Results folder contains all vcf files that we used for benchmarks that generated Figures 4-5 and Tables S1-S3 from the article.: [evidence span withheld — anchor status: external_doc]

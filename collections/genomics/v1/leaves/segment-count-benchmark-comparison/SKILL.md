---
name: segment-count-benchmark-comparison
description: Use when evaluating whether nanoranger or PacBio skera should be used
  to deconcatenate and quantify MAS-Seq (Kinnex-style) long-read single-cell cDNA
  data, or when reproducing a published head-to-head benchmark on PacBio Revio MAS-Seq
  runs.
when_to_use_negative:
- Input is short-read single-cell data (e.g. standard 10x Genomics scRNA-seq FASTQ)
  rather than concatenated MAS-Seq long reads — deconcatenation benchmarking does
  not apply
- Input is ONT long-read data from targeted amplification of 10x cDNA libraries, where
  skera (a PacBio tool) is not the comparator
- The goal is downstream variant/isoform calling from nanoranger output rather than
  quantifying deconcatenation and molecule-counting performance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: nanoranger
  role: Long-read single-cell processing pipeline run end-to-end on MAS-Seq reads
    with TruSeq-adapter barcode search to produce segment and molecule counts
  repo: https://github.com/mehdiborji/nanoranger
- name: Pacific Biosciences
  role: Source of the Revio MAS-Seq data and of skera, the commercial deconcatenation
    tool used as the benchmark comparator
- name: minimap2
  role: Spliced long-read aligner available in the workflow for aligning long cDNA
    reads against the transcriptome reference
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44137-7
    title: Integrative genotyping of cancer and immune phenotypes by long-read sequencing
  source_content_hash: sha256:7032e2dea7bb99b211a9ee42970a176783e9cc7c80c74313b69dce864c52ba7f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mehdiborji/nanoranger
  tools:
  - Pacific Biosciences
  - minimap2
  - nanoranger
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# segment-count-benchmark-comparison

## Summary

Benchmarking protocol for comparing nanoranger's deconcatenation and molecule quantification of PacBio MAS-ISO-seq single-cell reads against PacBio's skera tool, using per-read segment counts and per-gene/per-cell molecule counts as the comparative metrics.

## When to use

Use when evaluating whether nanoranger or PacBio skera should be used to deconcatenate and quantify MAS-Seq (Kinnex-style) long-read single-cell cDNA data, or when reproducing a published head-to-head benchmark on PacBio Revio MAS-Seq runs.

## When NOT to use

- Input is short-read single-cell data (e.g. standard 10x Genomics scRNA-seq FASTQ) rather than concatenated MAS-Seq long reads — deconcatenation benchmarking does not apply
- Input is ONT long-read data from targeted amplification of 10x cDNA libraries, where skera (a PacBio tool) is not the comparator
- The goal is downstream variant/isoform calling from nanoranger output rather than quantifying deconcatenation and molecule-counting performance

## Inputs

- PacBio Revio MAS-Seq run (e.g. DATA-Revio-PBMC-2) from downloads.pacbcloud.com (long-read single-cell cDNA sequencing data)
- GENCODE v44 transcriptome reference (alignment target)
- nanoranger workflow from github.com/mehdiborji/nanoranger
- Published skera benchmark values (median 12 vs 15 segments/read; 141 vs 55 molecules/gene)

## Outputs

- Per-read segment counts from the nanoranger output
- Per-gene and per-cell molecule counts from the nanoranger output
- Side-by-side comparison file nanoranger_vs_skera_comparison.csv (rows: segments-per-read and molecules-per-gene, with nanoranger and skera benchmark columns)

## How to apply

Download the public PacBio Revio MAS-Seq run (e.g. DATA-Revio-PBMC-2 from downloads.pacbcloud.com) containing long-read single-cell cDNA sequencing data from healthy donor PBMCs. Run the nanoranger workflow end-to-end (github.com/mehdiborji/nanoranger) with a GENCODE v44 transcriptome reference as the alignment target, configuring the TruSeq-adapter barcode search as described in the Methods. Extract per-read segment counts and per-gene/per-cell molecule counts from the nanoranger output and compute the median segments per read plus mean molecules per gene and per cell. Compare these against skera-based reference values — in the published benchmark, nanoranger found a median of 12 segments/read (range 0–37) versus skera's 15 (range 0–16), while detecting mean molecules per gene of 141 vs 55 and per cell of 2580 vs 1650, with Pearson correlations of r = 0.88 (per gene) and r = 0.97 (per cell). Write the side-by-side comparison to a CSV and validate that both metrics rows contain the reference skera values in the benchmark columns.

## Related tools

- **nanoranger** (Long-read single-cell processing pipeline run end-to-end on MAS-Seq reads with TruSeq-adapter barcode search to produce segment and molecule counts) — https://github.com/mehdiborji/nanoranger
- **Pacific Biosciences** (Source of the Revio MAS-Seq data and of skera, the commercial deconcatenation tool used as the benchmark comparator)
- **minimap2** (Spliced long-read aligner available in the workflow for aligning long cDNA reads against the transcriptome reference) — https://github.com/lh3/minimap2

## Examples

```
nextflow run mehdiborji/nanoranger --reads DATA-Revio-PBMC-2.bam --genome gencode.v44.transcripts.fa
```

## Evaluation signals

- nanoranger_vs_skera_comparison.csv exists and is well-formed CSV with rows for both metrics (segments-per-read and molecules-per-gene)
- The reported skera reference values (12 vs 15 segments/read; 141 vs 55 molecules/gene) are present as the benchmark columns
- Median segments per read for nanoranger falls in the reported range (median ~12, range 0–37)
- Per-gene and per-cell molecule counts correlate strongly with skera values (expected r ≈ 0.88 and r ≈ 0.97) while being consistently higher
- Per-cell molecule counts fall in the reported magnitude (nanoranger ~2580 vs skera ~1650 molecules per cell)

## Limitations

- nanoranger consistently finds fewer segments per read than skera (median 12 vs 15) yet reports higher molecule counts, so segment counts and molecule counts must be interpreted together rather than as independent quality measures
- skera reference values come from previously reported paper results rather than a re-run, so the comparison mixes freshly computed nanoranger metrics with published skera numbers
- The benchmark is specific to PacBio Revio MAS-Seq PBMC data with GENCODE v44; results may not transfer to other chemistries or references

## Evidence

- [results] nanoranger found consistently fewer segments per read (median 12; range 0–37) than skera (median 15; range 0–16): [evidence span withheld — anchor status: not_found]
- [results] molecules per gene and per cell correlations: [evidence span withheld — anchor status: not_found]
- [abstract] nanoranger pipeline description: "we devise a flexible long-read sequencing workflow and analysis pipeline, termed nanoranger, that starts from intermediate single-cell cDNA libraries"
- [readme] input data description from README: "The input data can be obtained through sequencing of 10x Genomics whole-transcriptome cDNA libraries or amplicons obtained through targeted amplification, with Oxford Nanopore Technologies (ONT) or"

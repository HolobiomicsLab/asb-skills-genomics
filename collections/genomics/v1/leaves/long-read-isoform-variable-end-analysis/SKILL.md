---
name: long-read-isoform-variable-end-analysis
description: Use when deciding how a collapsing/merging tool (isoseq collapse, gffcompare)
  treats transcripts with identical intron chains but variable 5'/3' ends, e.g., when
  variable ends across samples risk being discarded into a single full-length isoform
  entry.
when_to_use_negative:
- Input transcripts differ in intron chain, not just in 5'/3' end coordinates — the
  skill specifically probes variable ends on a shared intron chain.
- The goal is cross-sample isoform comparison by matched IDs (PB.X.Y), which isoSeQL
  addresses rather than collapse-behavior benchmarking.
- Only one collapsing tool is of interest and no tool-to-tool asymmetry comparison
  is needed.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoseq
  role: Collapses synthetic truncated isoform variants using default 50 bp 5'/100
    bp 3' deviation cutoffs (v4.2); recommended replacement for cDNA_Cupcake's collapse_isoforms_by_sam.py
  repo: https://isoseq.how
- name: gffcompare
  role: Merges the identical variant GFF to test whether it discards start/end variability,
    reporting only the full-length transcript
- name: cDNA_Cupcake
  role: Legacy alternative for collapsing isoforms, superseded by official isoseq
    collapse in the pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: isoSeQL
  role: Downstream context tool for compiling long-read transcriptomic data and identifying
    common and unique isoforms across multiple samples after SQANTI3 annotation
  repo: https://github.com/christine-liu/isoSeQL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:bdc5ed15fc6636c5e6e35c3c3962a4c4ed97233c3fbe39faf33c034de316915f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cDNA_Cupcake
  - gffcompare
  - isoSeQL
  - isoseq
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# long-read-isoform-variable-end-analysis

## Summary

A benchmark-style skill for testing how transcript collapsing/merging tools handle 5'/3' end variability when transcripts share an intron chain: synthetic truncated variants of a reference isoform (e.g., LPAR3 ENST00000440086.1) are collapsed with isoseq collapse and merged with gffcompare, and their outputs are compared to determine whether start/end coordinate variability is retained.

## When to use

Use when deciding how a collapsing/merging tool (isoseq collapse, gffcompare) treats transcripts with identical intron chains but variable 5'/3' ends, e.g., when variable ends across samples risk being discarded into a single full-length isoform entry.

## When NOT to use

- Input transcripts differ in intron chain, not just in 5'/3' end coordinates — the skill specifically probes variable ends on a shared intron chain.
- The goal is cross-sample isoform comparison by matched IDs (PB.X.Y), which isoSeQL addresses rather than collapse-behavior benchmarking.
- Only one collapsing tool is of interest and no tool-to-tool asymmetry comparison is needed.

## Inputs

- Reference transcript annotation in GTF format containing the target isoform (e.g., ENST00000440086.1)
- Synthetic GFF of 5'/3' truncated variant transcripts sharing the same intron chain

## Outputs

- Collapsed isoform set from isoseq collapse (GFF) retaining start/end coordinate variability
- Merged transcript output from gffcompare (GTF/GFF) containing only the full-length isoform entry
- variant_collapse_behavior_report.tsv with one row per tool: tool name, version, number of collapsed/merged isoforms, retained coordinate variability

## How to apply

Parse the reference GTF to extract the target transcript model (e.g., ENST00000440086.1 of LPAR3) and its intron chain. Programmatically generate synthetic 5'/3' truncated variants (Python/pandas) that retain the identical single intron chain but differ only in start/end coordinates, and write them to a GFF file. Collapse the variants with isoseq collapse (v4.2), which uses default 50 bp 5' deviation and 100 bp 3' deviation cutoffs, and separately merge the identical input with gffcompare. Then compare the two outputs: isoseq collapse retains multiple start/end-coordinate entries for the shared intron chain, whereas gffcompare reports only a single full-length transcript entry. Assemble the per-tool comparison into a TSV (tool name, version, number of collapsed/merged isoforms, retained coordinate variability) so the asymmetry between tools is documented explicitly.

## Related tools

- **isoseq** (Collapses synthetic truncated isoform variants using default 50 bp 5'/100 bp 3' deviation cutoffs (v4.2); recommended replacement for cDNA_Cupcake's collapse_isoforms_by_sam.py) — https://isoseq.how
- **gffcompare** (Merges the identical variant GFF to test whether it discards start/end variability, reporting only the full-length transcript)
- **cDNA_Cupcake** (Legacy alternative for collapsing isoforms, superseded by official isoseq collapse in the pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)) — https://github.com/Magdoll/cDNA_Cupcake
- **isoSeQL** (Downstream context tool for compiling long-read transcriptomic data and identifying common and unique isoforms across multiple samples after SQANTI3 annotation) — https://github.com/christine-liu/isoSeQL

## Examples

```
isoseq collapse lpar3_variants.gff lpar3_collapsed.gff
```

## Evaluation signals

- variant_collapse_behavior_report.tsv contains one row each for isoseq collapse and gffcompare with tool name, version, isoform count, and retained coordinate variability.
- isoseq collapse output retains multiple start/end-coordinate entries for the shared single intron chain.
- gffcompare output contains only a single full-length isoform entry, documenting the asymmetry.
- All synthetic variants share the identical intron chain extracted from the reference GTF for ENST00000440086.1.
- Truncations exceed the isoseq collapse default deviation cutoffs (50 bp 5', 100 bp 3') so the behavior test is meaningful.

## Limitations

- Collapsing behavior is tool- and cutoff-dependent: gffcompare discards start/end variability for identical intron chains, so merged transcriptomes may under-report variable ends.
- The mega-sample concatenation approach does not scale well with increased sample numbers or as SMRTcell throughput has increased.
- cDNA_Cupcake collapse scripts are deprecated; the maintainer removed redundant scripts in favor of official isoseq equivalents and advises transitioning at your own risk.

## Evidence

- [other] finding on gffcompare behavior: [evidence span withheld — anchor status: not_found]
- [other] workflow on isoseq collapse cutoffs: [evidence span withheld — anchor status: not_found]
- [intro] pipeline ordering: "running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2) on that single "mega sample""
- [readme] cDNA_Cupcake deprecation note: [evidence span withheld — anchor status: external_doc]
- [intro] cross-sample motivation: "I was working with multiple samples that I wanted to compare to each other, but quickly realized that the isoform ID #s (PB.X.Y) in each sample couldn't be matched."

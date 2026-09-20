---
name: intron-chain-annotation-parsing
description: Use when you need to extract intron chains from a reference GTF annotation
  and test whether transcript collapsing or merging tools retain or discard start/end
  variability among transcripts that share the same intron chain.
when_to_use_negative:
- Input is raw long-read sequencing data (HiFi/FLNC reads) rather than an existing
  annotation/transcript model to parse
- The goal is collapsing actual Iso-Seq cluster output (PB.X.Y isoforms) across samples
  rather than benchmarking collapse behavior on synthetic variants
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: isoseq
  role: Collapses the synthetic variant GFF into unique isoforms using default 50
    bp 5' deviation and 100 bp 3' deviation cutoffs; also part of the standard Iso-Seq
    pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)
  repo: https://isoseq.how
- name: gffcompare
  role: Merges the same variant GFF input to test whether start/end variability for
    the shared intron chain is discarded in the merged transcriptome
- name: Python
  role: Parses the reference GTF to extract the transcript model and intron chain
    and generates the synthetic truncated variant transcripts (isoSeQL itself was
    developed using Python 3.9.6)
- name: isoSeQL
  role: Downstream tool for compiling long-read transcriptomic data and identifying
    common and unique isoforms across samples using SQANTI3-annotated output
  repo: https://github.com/christine-liu/isoSeQL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:ce4e240feaddb15ed9a7dc69ec876d6a58f1ab1cfa56db69c9bed92347750dc5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - gffcompare
  - isoSeQL
  - isoseq
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# intron-chain-annotation-parsing

## Summary

A skill for parsing reference GTF/GFF annotations to extract transcript models and their intron chains (e.g. LPAR3 isoform ENST00000440086.1), then programmatically generating synthetic truncated 5'/3' variants sharing an identical intron chain to benchmark how collapse/merge tools (isoseq collapse vs gffcompare) treat start/end-coordinate variability.

## When to use

Use when you need to extract intron chains from a reference GTF annotation and test whether transcript collapsing or merging tools retain or discard start/end variability among transcripts that share the same intron chain.

## When NOT to use

- Input is raw long-read sequencing data (HiFi/FLNC reads) rather than an existing annotation/transcript model to parse
- The goal is collapsing actual Iso-Seq cluster output (PB.X.Y isoforms) across samples rather than benchmarking collapse behavior on synthetic variants

## Inputs

- Reference genome annotation in GTF format containing the transcript of interest (e.g. LPAR3 ENST00000440086.1)
- Programmatically generated GFF file of synthetic 5'/3'-truncated variant transcripts sharing one intron chain

## Outputs

- Collapsed isoform set from isoseq collapse (GFF/GTF)
- Merged transcript output from gffcompare (GTF)
- variant_collapse_behavior_report.tsv with one row per tool: tool name, version, number of collapsed/merged isoforms, and retained coordinate variability

## How to apply

Parse the reference GTF (e.g. with Python/pandas) to extract the transcript model of interest and its intron chain, such as LPAR3 isoform ENST00000440086.1. Programmatically generate synthetic variant transcripts truncated on the 5' or 3' end that retain the identical single intron chain but differ in start/end coordinates, emulating variable ends across samples, and write them to a GFF file. Collapse the variants with isoseq collapse (v4.2) using the default 50 bp 5' deviation and 100 bp 3' deviation cutoffs, and merge the same GFF with gffcompare. Compare the outputs: record whether isoseq collapse retains multiple start/end-coordinate entries for the shared intron chain versus gffcompare reporting a single full-length transcript entry. Assemble the per-tool comparison into variant_collapse_behavior_report.tsv with one row per tool (tool name, version, number of collapsed/merged isoforms, retained coordinate variability).

## Related tools

- **isoseq** (Collapses the synthetic variant GFF into unique isoforms using default 50 bp 5' deviation and 100 bp 3' deviation cutoffs; also part of the standard Iso-Seq pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)) — https://isoseq.how
- **gffcompare** (Merges the same variant GFF input to test whether start/end variability for the shared intron chain is discarded in the merged transcriptome)
- **Python** (Parses the reference GTF to extract the transcript model and intron chain and generates the synthetic truncated variant transcripts (isoSeQL itself was developed using Python 3.9.6))
- **isoSeQL** (Downstream tool for compiling long-read transcriptomic data and identifying common and unique isoforms across samples using SQANTI3-annotated output) — https://github.com/christine-liu/isoSeQL

## Examples

```
isoseq collapse variants.gff collapsed.gff --dev-mono-5p 50 --dev-mono-3p 100
```

## Evaluation signals

- variant_collapse_behavior_report.tsv contains rows for both isoseq collapse and gffcompare with tool names and versions recorded
- All synthetic variant transcripts share the identical single intron chain of the reference isoform (only start/end coordinates differ)
- isoseq collapse output retains multiple start/end-coordinate entries for the shared intron chain
- gffcompare output contains only the single full-length isoform entry, documenting the asymmetry between the two tools

## Limitations

- Merging truncated variants with gffcompare discards start/end variability: only the full-length isoform is reported in the merged transcriptome
- isoseq collapse behavior depends on its default deviation cutoffs (50 bp 5', 100 bp 3'); variants truncated beyond these windows may be treated differently
- cDNA_Cupcake collapsing scripts are deprecated in favor of official isoseq software (e.g. isoseq3 collapse is now preferred over collapse_isoforms_by_sam.py)

## Evidence

- [methods] isoseq cluster2 generates clusters of isoforms with the support of at least 2 FLNC reads: "isoseq cluster2 generates clusters of isoforms with the support of at least 2 FLNC reads"
- [intro] Iso-Seq pipeline ordering: "running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)"
- [readme] cDNA_Cupcake deprecation in favor of isoseq: [evidence span withheld — anchor status: external_doc]
- [abstract] isoSeQL purpose statement: "We developed isoSeQL, a tool for compiling long-read transcriptomic data, identifying common and unique isoforms across multiple samples"

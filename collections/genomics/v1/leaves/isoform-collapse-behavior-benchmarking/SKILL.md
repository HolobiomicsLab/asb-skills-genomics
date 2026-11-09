---
name: isoform-collapse-behavior-benchmarking
description: Decision support for choosing an isoform collapsing/merging tool when
  5'/3' end variability across samples matters — e.g. simulated or real truncated
  isoforms sharing the same intron chain must be collapsed and you must know if start/end
  coordinates survive.
when_to_use_negative:
- Input transcripts have distinct intron chains — the benchmark specifically probes
  collapsing of variants that share the same intron chain, so differing splice junctions
  test a different property.
- The goal is cross-sample isoform ID matching over existing SQANTI3 output (PB.X.Y
  IDs) — use isoSeQL for that instead of re-running collapse benchmarks.
- You need to collapse raw FLNC read clusters (isoseq cluster2) rather than compare
  how pre-built transcript models with variable ends are merged.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: isoseq
  role: Collapses the synthetic truncated variant GFF/BAM into a unique isoform set
    using default 50 bp 5' deviation and 100 bp 3' deviation cutoffs; its output is
    one arm of the behavior comparison.
  repo: https://isoseq.how
- name: gffcompare
  role: Merges the identical variant GFF input; its merged transcriptome output is
    the second arm of the comparison (documented to report only the full-length isoform).
- name: cDNA_Cupcake
  role: Legacy alternative collapsing route (collapse_isoforms_by_sam.py) superseded
    by official isoseq software; relevant when reproducing historical collapsing pipelines.
  repo: https://github.com/Magdoll/cDNA_Cupcake
- name: isoSeQL
  role: Downstream tool for compiling and comparing collapsed long-read isoforms across
    samples, motivating awareness of whether collapsing preserves end-coordinate variability.
  repo: https://github.com/christine-liu/isoSeQL
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf680
    title: 'isoSeQL: comparing long-read isoforms across multiple datasets'
  source_content_hash: sha256:7348be8bb2644f0eac05b5e51506839778d9c4010c7774ed52c57ed30ef46bd6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - cDNA_Cupcake
  - gffcompare
  - isoSeQL
  - isoseq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# isoform-collapse-behavior-benchmarking

## Summary

A controlled benchmarking skill that probes how different long-read isoform collapsing tools (isoseq collapse vs. gffcompare) treat synthetic transcript variants that share an intron chain but differ in start/end coordinates, revealing whether transcript-end variability is retained or discarded during collapsing.

## When to use

Decision support for choosing an isoform collapsing/merging tool when 5'/3' end variability across samples matters — e.g. simulated or real truncated isoforms sharing the same intron chain must be collapsed and you must know if start/end coordinates survive.

## When NOT to use

- Input transcripts have distinct intron chains — the benchmark specifically probes collapsing of variants that share the same intron chain, so differing splice junctions test a different property.
- The goal is cross-sample isoform ID matching over existing SQANTI3 output (PB.X.Y IDs) — use isoSeQL for that instead of re-running collapse benchmarks.
- You need to collapse raw FLNC read clusters (isoseq cluster2) rather than compare how pre-built transcript models with variable ends are merged.

## Inputs

- Reference annotation GTF containing the transcript of interest (e.g. ENST00000440086.1 with its intron chain)
- Synthetic truncated 5'/3' variant transcripts sharing the identical intron chain, written as a GFF file
- Tool executables: isoseq collapse (v4.2) and gffcompare

## Outputs

- Collapsed/merged transcript sets from isoseq collapse and gffcompare for identical input
- variant_collapse_behavior_report.tsv with one row per tool (tool name, version, number of collapsed/merged isoforms, retained coordinate variability)

## How to apply

Parse the reference GTF to extract the target transcript model (e.g. LPAR3 isoform ENST00000440086.1) and its intron chain. Programmatically generate synthetic 5'/3' truncated variants (Python/pandas) that retain the identical intron chain but differ in start/end coordinates, and write them to a GFF file. Run the same variant GFF through two collapsing paths: isoseq collapse (v4.2, default 50 bp 5' deviation and 100 bp 3' deviation cutoffs) and gffcompare merge. Diff the two outputs to record whether isoseq collapse retains multiple start/end-coordinate entries for the shared intron chain while gffcompare reports only a single full-length transcript. Assemble the per-tool behavior into a TSV report with tool name, version, number of collapsed/merged isoforms, and retained coordinate variability.

## Related tools

- **isoseq** (Collapses the synthetic truncated variant GFF/BAM into a unique isoform set using default 50 bp 5' deviation and 100 bp 3' deviation cutoffs; its output is one arm of the behavior comparison.) — https://isoseq.how
- **gffcompare** (Merges the identical variant GFF input; its merged transcriptome output is the second arm of the comparison (documented to report only the full-length isoform).)
- **cDNA_Cupcake** (Legacy alternative collapsing route (collapse_isoforms_by_sam.py) superseded by official isoseq software; relevant when reproducing historical collapsing pipelines.) — https://github.com/Magdoll/cDNA_Cupcake
- **isoSeQL** (Downstream tool for compiling and comparing collapsed long-read isoforms across samples, motivating awareness of whether collapsing preserves end-coordinate variability.) — https://github.com/christine-liu/isoSeQL

## Examples

```
isoseq collapse lpar3_truncation_variants.bam hg38.annotation.gtf lpar3_collapsed.gff; gffcompare -r hg38.annotation.gtf -o gffcompare_lpar3 lpar3_truncation_variants.gff
```

## Evaluation signals

- variant_collapse_behavior_report.tsv contains a row for both isoseq collapse and gffcompare.
- The isoseq collapse row shows retained start/end coordinate variability for the shared intron chain (multiple coordinate entries).
- The gffcompare row shows only a single full-length isoform entry for the same input (start/end variability discarded).
- Every synthetic variant in the input GFF retains the identical intron chain as ENST00000440086.1 — verify before collapsing.
- isoseq collapse uses default cutoffs (50 bp 5' deviation, 100 bp 3' deviation) so the comparison reflects tool defaults, not tuned parameters.

## Limitations

- The finding is demonstrated on a single-intron simulated locus (LPAR3 ENST00000440086.1); behavior on multi-intron or more complex loci is not established by this benchmark.
- gffcompare's collapsing discards start/end variability here, so downstream tools that expect end-coordinate distinctions (e.g. cross-sample PB.X.Y comparisons) may see information loss.
- The comparison depends on tool versions (isoseq v4.2) and default cutoffs; other versions or parameterizations may yield different collapsing behavior.

## Evidence

- [intro] running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2): "running the analysis pipeline (isoseq > minimap2 > cDNA_cupcake > SQANTI2)"
- [methods] isoseq cluster2 generates clusters of isoforms with the support of at least 2 FLNC reads: "isoseq cluster2 generates clusters of isoforms with the support of at least 2 FLNC reads"
- [readme] isoseq3 collapse` is now the preferred tool over `collapse_isoforms_by_sam.py: [evidence span withheld — anchor status: external_doc]

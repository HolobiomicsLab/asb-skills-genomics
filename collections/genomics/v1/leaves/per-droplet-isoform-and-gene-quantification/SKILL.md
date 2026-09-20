---
name: per-droplet-isoform-and-gene-quantification
description: Use when you have barcoded long-read alignments to a reference genome
  and need per-cell/per-droplet gene and isoform count matrices, including novel isoforms,
  with explicit handling of multimapping reads.
when_to_use_negative:
- Input is bulk long-read data without 10x droplet barcodes and no per-cell quantification
  is required — IsoQuant alone can quantify without the per-droplet machinery
- Reads are unaligned FASTQ — alignment (minimap2_splice) and transcript reconstruction
  must run first
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scywalker
  role: orchestrates read assignment and per-droplet weighted/unique/strict count
    generation from IsoQuant transcript models
  repo: https://github.com/derijkp/scywalker
- name: IsoQuant
  role: reconstructs known and novel transcript models to which reads are assigned
    for quantification
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: implementation framework for scywalker; provides tools for querying the tsv
    count and read_assignment tables
  repo: https://github.com/derijkp/genomecomb
- name: minimap2
  role: splice-preset aligner producing the barcoded long-read alignments used as
    quantification input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:8cd3d5e7d24e01d21bc9244907c4cd39ba4eb359808c1e13d2ecc55ae7e2743d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/derijkp/scywalker
  tools:
  - IsoQuant
  - genomecomb
  - minimap2
  - scywalker
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# per-droplet isoform and gene quantification

## Summary

scywalker assigns barcoded long reads to IsoQuant-reconstructed transcript models and generates per-droplet (per-cell) isoform and gene count tables in weighted, unique, strict, polyA-restricted, and nicount variants, in _raw and _filtered versions. This turns full-length nanopore/PacBio alignments into single-cell quantification comparable to short-read count matrices.

## When to use

Use when you have barcoded long-read alignments to a reference genome and need per-cell/per-droplet gene and isoform count matrices, including novel isoforms, with explicit handling of multimapping reads.

## When NOT to use

- Input is bulk long-read data without 10x droplet barcodes and no per-cell quantification is required — IsoQuant alone can quantify without the per-droplet machinery
- Reads are unaligned FASTQ — alignment (minimap2_splice) and transcript reconstruction must run first

## Inputs

- droplet-barcoded long-read alignments (minimap2 splice-preset BAM from 10x ONT or PacBio reads)
- merged IsoQuant transcript models (known and novel, per-region merged)
- reference directory (refdir) with genome, GTF/gene tsv, and minimap2 indexes
- emptydrops cell approvals (is_cell flags from the cell-calling step)

## Outputs

- read_assignment table (ambiguity, gambiguity, umicount, covered_pct, polyA, completeness fields)
- per-droplet isoform count tables: weighted, unique, strict (>= 90% coverage), polyA-restricted, in _raw and _filtered versions
- per-droplet gene count tables: weighted (1/N multimapping), unique, strict, polyA-restricted, nicount, in _raw and _filtered versions
- bulk count tables alongside per-cell counts

## How to apply

After region-split IsoQuant reconstruction and merging of per-region transcript models via unique location-based transcript IDs, assign reads to the merged models and build a read_assignment table with ambiguity, gambiguity, umicount, covered_pct, polyA, and completeness fields. From these assignments produce per-droplet count tables: weighted counts give a read mapping to N genes 1/N count per gene; unique counts use only unambiguous reads; strict counts require unique reads covering >= 90% of the transcript. Additionally emit polyA-restricted and gene nicount variants, each as _raw (all detected cells) and _filtered (emptydrops-approved cells only). Validate by checking that _filtered files contain only emptydrops-approved cells (is_cell = 1) and that strict counts are restricted to unique reads with >= 90% transcript coverage.

## Related tools

- **scywalker** (orchestrates read assignment and per-droplet weighted/unique/strict count generation from IsoQuant transcript models) — https://github.com/derijkp/scywalker
- **IsoQuant** (reconstructs known and novel transcript models to which reads are assigned for quantification) — https://github.com/ablab/IsoQuant
- **genomecomb** (implementation framework for scywalker; provides tools for querying the tsv count and read_assignment tables) — https://github.com/derijkp/genomecomb
- **minimap2** (splice-preset aligner producing the barcoded long-read alignments used as quantification input)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- sc_gene_counts_filtered files contain only emptydrops-approved cells (is_cell = 1); _raw files contain all detected cells
- counts_strict columns include only unique reads that cover >= 90% of the transcript
- Weighted gene counts for a read mapping to N genes sum to 1 across those genes (1/N each)
- read_assignment table exposes all required fields: ambiguity, gambiguity, umicount, covered_pct, polyA, completeness
- Per-cell counts correlate well with matched short-read gene quantification (as benchmarked on human brain, Arabidopsis, and mixed cell line datasets)

## Limitations

- IsoQuant-based code can hang or crash on organelle chromosomes (e.g. chrM, chrPt), which is why organelles are excluded via the refdir organelles configuration
- Strict counts are conservative — only unique reads covering >= 90% of the transcript are counted, reducing counts for multimapping or partially covered isoforms
- Cell-type assignment (pseudobulk) is not possible without a markerset; on limited datasets celltyping may be unreliable

## Evidence

- [results] In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells".: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [results] counts_strict: only unique reads that cover >= 90% of the transcript: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [results] multimapping reads are weighed (if maps to N genes -> each gets 1/N count): "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [intro] reconstruct (based on [IsoQuant](https://github.com/ablab/IsoQuant)) and quantify isoforms and genes: "reconstruct (based on [IsoQuant](https://github.com/ablab/IsoQuant)) and quantify isoforms and genes"
- [methods] The isoquant based code often hangs or crashes on the very different organelle data.: "The isoquant based code often hangs or crashes on the
very different organelle data."

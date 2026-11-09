---
name: multimapping-read-weighting
description: Use when building per-cell gene/isoform count tables from droplet-barcoded
  long-read alignments and reads must be assigned to reconstructed (known or novel)
  transcript models despite gene- or isoform-level multimapping ambiguity.
when_to_use_negative:
- Input is short-read single-cell data or lacks droplet cell barcodes — scywalker
  targets 10x-style long-read (ONT/PacBio) full-length cDNA
- Only bulk-level abundance is needed and no per-cell/per-droplet counting is required
  — weighting across cells adds overhead without benefit
- The reference contains organelle sequences not excluded via the refdir organelles
  configuration, since the IsoQuant-based code often hangs or crashes on organelle
  data
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scywalker
  role: Executes region-split IsoQuant calling, merges transcript models, assigns
    reads, and emits weighted/unique/strict per-droplet count tables
  repo: https://github.com/derijkp/scywalker
- name: IsoQuant
  role: Underlying engine for reconstructing known and novel transcript models to
    which reads are assigned and counted
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Framework in which scywalker's counting algorithms and genomecomb tooling
    (tsv querying, refdir handling) are implemented
  repo: https://github.com/derijkp/genomecomb
- name: minimap2
  role: Default aligner (minimap2_splice, splice preset) producing the barcoded alignments
    that feed read assignment and counting
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:f8be667edc30df36af06a1422fe742d18070d8e6794e46e1eac464c247bcf380
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
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multimapping-read-weighting

## Summary

scywalker's counting scheme for per-droplet gene and isoform tables from long-read single-cell data, in which reads mapping to multiple genes are fractionated (a read mapping to N genes contributes 1/N to each) and complemented by unique and strict count variants.

## When to use

Use when building per-cell gene/isoform count tables from droplet-barcoded long-read alignments and reads must be assigned to reconstructed (known or novel) transcript models despite gene- or isoform-level multimapping ambiguity.

## When NOT to use

- Input is short-read single-cell data or lacks droplet cell barcodes — scywalker targets 10x-style long-read (ONT/PacBio) full-length cDNA
- Only bulk-level abundance is needed and no per-cell/per-droplet counting is required — weighting across cells adds overhead without benefit
- The reference contains organelle sequences not excluded via the refdir organelles configuration, since the IsoQuant-based code often hangs or crashes on organelle data

## Inputs

- Droplet-barcoded long-read alignments to the reference genome (minimap2 splice preset, sorted BAM from the refdir)
- Merged per-region transcript models (known and novel) with unique location-based transcript IDs
- Reference directory (refdir) with genome fasta, gtf transcripts, and minimap2 indexes
- emptydrops cell-calling results used to distinguish _raw from _filtered count tables

## Outputs

- Per-droplet isoform count tables: counts_weighted, counts_unique, counts_strict (>= 90% transcript coverage), and polyA-restricted variants, each as _raw and _filtered versions
- Per-droplet gene count tables including weighted gene counts with 1/N multimapping and gene nicount variants, as _raw and _filtered versions
- read_assignment table with fields: ambiguity, gambiguity, umicount, covered_pct, polyA, completeness

## How to apply

After IsoQuant-based transcript reconstruction (per-region models merged via unique location-based transcript IDs), assign each barcoded read to the merged transcript models and record ambiguity metadata (ambiguity, gambiguity, umicount, covered_pct, polyA, completeness) in the read_assignment table. Generate weighted counts by weighing multimapping reads: a read mapping to N genes gives each gene a 1/N count. In parallel emit unique counts (unambiguous reads only) and strict counts, where strict counts retain only unique reads that cover >= 90% of the transcript. Also produce polyA-restricted and gene nicount variants, and emit each table in _raw (all detected cells) and _filtered (emptydrops-approved cells only) versions.

## Related tools

- **scywalker** (Executes region-split IsoQuant calling, merges transcript models, assigns reads, and emits weighted/unique/strict per-droplet count tables) — https://github.com/derijkp/scywalker
- **IsoQuant** (Underlying engine for reconstructing known and novel transcript models to which reads are assigned and counted) — https://github.com/ablab/IsoQuant
- **genomecomb** (Framework in which scywalker's counting algorithms and genomecomb tooling (tsv querying, refdir handling) are implemented) — https://github.com/derijkp/genomecomb
- **minimap2** (Default aligner (minimap2_splice, splice preset) producing the barcoded alignments that feed read assignment and counting)

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -cellmarkerfile markers_chr17.tsv -threads 6 test10x
```

## Evaluation signals

- A read mapping to N genes contributes exactly 1/N to each gene in the weighted count table
- counts_strict contains only unique reads covering >= 90% of the transcript
- _filtered count files contain only emptydrops-approved cells (is_cell = 1), while _raw files cover all detected cells
- read_assignment table exposes all required fields: ambiguity, gambiguity, umicount, covered_pct, polyA, completeness
- Transcript models derive from per-region IsoQuant calls merged with unique location-based transcript IDs

## Limitations

- The IsoQuant-based code often hangs or crashes on organelle data, so organelle chromosomes must be excluded via the refdir organelles configuration
- Weighted counts fractionate gene-multimapping reads, so per-gene weighted values are fractional and should be interpreted differently from unique counts
- Strict counts discard multimapping reads entirely and require >= 90% transcript coverage, reducing sensitivity for partially covered transcripts

## Evidence

- [results] Gene multimapping reads are weighed so that a read mapping to N genes gives each gene 1/N count: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [results] Strict counts definition: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [results] Filtered vs raw count files: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."
- [intro] IsoQuant-based reconstruction and quantification in the end-to-end pipeline: "it will find and assign cellbarcodes, align reads, and reconstruct (based on [IsoQuant](https://github.com/ablab/IsoQuant)) and quantify isoforms and genes"

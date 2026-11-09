---
name: single-cell-transcript-reconstruction-with-isoquant
description: Use when starting from barcoded, long-read (ONT or PacBio) alignments
  to a reference genome and needing known/novel transcript models plus per-droplet
  isoform and gene count matrices.
when_to_use_negative:
- Input is already a count matrix or read_assignment table — this skill covers alignment-to-counts
  reconstruction, not downstream quantification
- Analysis targets organelle chromosomes without the refdir organelles configuration
  — IsoQuant hangs or crashes on organelle data
- Only short-read data is available without long reads — scywalker targets full-length
  single-cell/nuclei cDNA long-read data
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: scywalker
  role: 'End-to-end driver: splits alignments into regions, runs IsoQuant per region,
    merges transcript models, and emits per-cell count tables'
  repo: https://github.com/derijkp/scywalker
- name: minimap2
  role: Aligns barcoded long reads to the reference genome using the splice preset
    (minimap2_splice default; minimap2_splicesmall experimental)
- name: IsoQuant
  role: Reconstructs known and novel transcript models per genomic region and assigns
    reads to the merged models
  repo: https://github.com/ablab/IsoQuant
- name: genomecomb
  role: Implementation framework in which scywalker's region distribution, counting,
    and tsv handling are built
  repo: https://github.com/derijkp/genomecomb/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae549
    title: 'Scywalker: scalable end-to-end data analysis workflow for long-read single-cell
      transcriptome sequencing'
  source_content_hash: sha256:b11c1ec11b8d048a5ebb561d327c8a5bcd039b45936b3826f5bfca50f176b8a3
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-transcript-reconstruction-with-isoquant

## Summary

Reconstruct and quantify genes and isoforms from droplet-barcoded long-read single-cell alignments by splitting the genome into ~5 Mb regions, running IsoQuant per region, merging models, and producing weighted, unique, and strict per-cell count tables.

## When to use

Use when starting from barcoded, long-read (ONT or PacBio) alignments to a reference genome and needing known/novel transcript models plus per-droplet isoform and gene count matrices.

## When NOT to use

- Input is already a count matrix or read_assignment table — this skill covers alignment-to-counts reconstruction, not downstream quantification
- Analysis targets organelle chromosomes without the refdir organelles configuration — IsoQuant hangs or crashes on organelle data
- Only short-read data is available without long reads — scywalker targets full-length single-cell/nuclei cDNA long-read data

## Inputs

- Barcoded long-read FASTQ files in a sample/project directory (fastq subdirectory)
- Reference directory (refdir) with genome FASTA, GTF/gene tsv, minimap2 indexes, and organelles configuration (built by scywalker_makerefdir)
- Droplet cell barcodes (e.g. 737K-august-2016 10x whitelist)

## Outputs

- Merged transcript models with unique location-based transcript IDs
- read_assignment table with ambiguity, gambiguity, umicount, covered_pct, polyA, completeness fields
- Per-droplet isoform and gene count tables: weighted, unique, strict (>=90% transcript coverage), polyA-restricted, and gene nicount variants
- _raw and _filtered versions of each count table (filtered restricted to emptydrops-approved cells)

## How to apply

Align barcoded FASTQ reads to the reference directory (refdir) with minimap2 using the splice preset (scywalker default minimap2_splice; minimap2_splicesmall only experimentally for small-exon detection). Distribute alignment and transcript reconstruction over genomic regions with the default distrreg setting g5000000 (~5 Mb target, splits only in 250kbase gene-free regions) to reduce peak memory use, excluding organelle chromosomes via the refdir organelles configuration because IsoQuant hangs on organelle data. Reconstruct known and novel transcript models per region with IsoQuant, merge per-region models using unique location-based transcript IDs, and assign reads to the merged models to build a read_assignment table with ambiguity, gambiguity, umicount, covered_pct, polyA, and completeness fields. Emit per-droplet isoform and gene counts in weighted form (multimapping reads weighed 1/N across genes), unique form, and strict form (unique reads covering >= 90% of the transcript), plus polyA-restricted and gene nicount variants, each as _raw (all detected cells) and _filtered (emptydrops-approved cells) versions.

## Related tools

- **scywalker** (End-to-end driver: splits alignments into regions, runs IsoQuant per region, merges transcript models, and emits per-cell count tables) — https://github.com/derijkp/scywalker
- **minimap2** (Aligns barcoded long reads to the reference genome using the splice preset (minimap2_splice default; minimap2_splicesmall experimental))
- **IsoQuant** (Reconstructs known and novel transcript models per genomic region and assigns reads to the merged models) — https://github.com/ablab/IsoQuant
- **genomecomb** (Implementation framework in which scywalker's region distribution, counting, and tsv handling are built) — https://github.com/derijkp/genomecomb/

## Examples

```
scywalker -v 1 -d 8 -dmaxmem 64G -refdir g17 -sc_expectedcells 183 -threads 6 test10x
```

## Evaluation signals

- _filtered count files contain only emptydrops-approved cells (is_cell=1); _raw files contain counts for all detected cells
- counts_strict is restricted to unique reads that cover >= 90% of the transcript
- read_assignment table exposes all required fields: ambiguity, gambiguity, umicount, covered_pct, polyA, completeness
- Multimapping reads are weighed 1/N across N genes in weighted counts
- Region splitting occurs only in >=250kbase gene-free regions near the ~5 Mb target, and per-region models merge to unique location-based transcript IDs

## Limitations

- IsoQuant-based code often hangs or crashes on organelle data, so organelle chromosomes must be excluded via the refdir organelles configuration
- Default minimap2_splice preset may miss very small exons; minimap2_splicesmall is provided only as an experimental alternative
- Region splitting granularity and default settings (distrreg g5000000) are tuned for scalability; deviating requires explicit reconfiguration

## Evidence

- [other] scywalker's gene/isoform calling splits the alignments into smaller regions (default target 5Mbase, with splits only in 250kbase gene-free regions) that are processed separately, improving efficiency and reducing peak memory use.: [evidence span withheld — anchor status: not_found]
- [results] Isoform counting distinguishes weighed, unique, and strict counts, with strict requiring >= 90% transcript coverage: "counts_strict: only unique reads that cover >= 90% of the transcript"
- [results] Gene multimapping reads are weighed so that a read mapping to N genes gives each gene 1/N count: "multimapping reads are weighed (if maps to N genes -> each gets 1/N count)"
- [methods] The default aligner is minimap2_splice (minimap2 with the splice preset): "The default aligner is minimap2_splice (minimap2 with the splice preset)"
- [readme] The isoquant based code often hangs or crashes on the very different organelle data.: "The isoquant based code often hangs or crashes on the
very different organelle data."
- [results] scywalker _filtered files contain only emptydrops approved cells, _raw files provide counts for all detected cells: "In this file (_filtered) only information on the emptydrops approved cells is given, the file with _raw provides counts for all detected "cells"."

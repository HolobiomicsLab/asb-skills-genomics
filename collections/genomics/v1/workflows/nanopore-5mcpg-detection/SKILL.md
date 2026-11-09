---
name: nanopore-5mcpg-detection-workflow
description: 'Use when nanopore POD5 or FAST5 signal must be basecalled with move
  tables, aligned while preserving move tags, analysed for 5mCpG with DeepMod2, aggregated
  with modkit, and checked against orthogonal methylation measurements.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - basecalling-with-move-tables
  - dorado-move-table-basecalling
  - parallel-basecalling-of-pod5-batches
  - ont-read-alignment-to-reference-genome
  - bam-file-processing-and-indexing
  - sam-to-bam-conversion-and-indexing
  - sam-bam-alignment-file-handling
  - signal-based-cpg-methylation-detection
  - nanopore-modified-basecalling
  - nanopore-methylation-calling--ab6e4dfe90fd7ce1
  - reference-anchored-cpg-methylation-aggregation
  - modkit-modification-pileup
  - per-site-methylation-frequency-aggregation
  - modkit-modification-site-counting
  - bedmethyl-frequency-aggregation
  - low-coverage-site-filtering
  - bisulfite-methylation-ground-truth-matching
  - cross-platform-methylation-correlation-analysis--50dd8f2e672836d6
  - bedtools-genomic-interval-intersection
  - pearson-correlation-of-per-site-methylation-levels
  member_tools:
  - DeepMod2
  - Dorado
  - Guppy
  - minimap2
  - modkit
  - samtools
  - Dogme
  - bedtools
  coverage_gaps: []
  derived_from_workflows: []
  bound_by: perspicacite-semantic
  collection: https://w3id.org/holobiomicslab/asb-skill/collection/genomics/v1
schema_version: 0.3.0
attribution:
  generator: AgenticScienceBuilder
  promoter: Louis-Félix Nothias
  sponsor: CNRS & Université Côte d'Azur
---

# Nanopore 5mCpG Detection and Validation

## Summary

Produce tag-preserving nanopore alignments, infer per-read 5mCpG, aggregate CpG-site frequencies, and validate the methylation profile.


## When to use

Use when nanopore POD5 or FAST5 signal must be basecalled with move tables, aligned while preserving move tags, analysed for 5mCpG with DeepMod2, aggregated with modkit, and checked against orthogonal methylation measurements.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — basecall_with_move_tables

**Goal:** basecall raw nanopore signal while retaining signal-to-base moves

**EDAM operation:** http://edamontology.org/operation_0292

**Inputs:** nanopore-signal/pod5-fast5 · **Outputs:** bam/move-table-unaligned

**Candidate leaf skills:** `basecalling-with-move-tables` (primary), `dorado-move-table-basecalling`, `parallel-basecalling-of-pod5-batches`

**Tools (primary):** DeepMod2, Dorado, Guppy, minimap2, modkit, samtools

**Other candidate tools:** Dogme, Nextflow

**Grounding:** 2 KB(s); DOIs: 10.1038/s41467-024-45778-y, 10.1093/bioinformatics/btag066

### Stage 2 — align_move_table_bam

**Goal:** align basecalls without dropping signal-to-base move tags

**EDAM operation:** http://edamontology.org/operation_3198

**Inputs:** bam/move-table-unaligned, fasta/reference-genome · **Outputs:** bam/move-table-aligned

**Candidate leaf skills:** `ont-read-alignment-to-reference-genome` (primary), `bam-file-processing-and-indexing`, `sam-to-bam-conversion-and-indexing`, `sam-bam-alignment-file-handling`

**Tools (primary):** DeepMod2, Dorado, Guppy, minimap2, modkit, samtools

**Other candidate tools:** AgileStructure, EpiNano, NanoFilt

**Grounding:** 3 KB(s); DOIs: 10.1038/s41467-019-11713-9, 10.1038/s41467-024-45778-y, 10.1093/bioinformatics/btag294

### Stage 3 — detect_5mcpg

**Goal:** infer per-read 5mCpG probabilities from signal-aligned basecalls

**EDAM operation:** http://edamontology.org/operation_3202

**Inputs:** nanopore-signal/pod5-fast5, bam/move-table-aligned, fasta/reference-genome · **Outputs:** modbam/5mcpg-per-read

**Candidate leaf skills:** `signal-based-cpg-methylation-detection` (primary), `nanopore-modified-basecalling`, `nanopore-methylation-calling--ab6e4dfe90fd7ce1`, `reference-anchored-cpg-methylation-aggregation`

**Tools (primary):** DeepMod2, Dorado, Guppy, minimap2, modkit, samtools

**Other candidate tools:** DeepMod, DeepSignal, METEORE, Megalodon, Nanopolish, Remora, Snakemake, Tombo, gcf52ref, ont_fast5_api, bedtools

**Grounding:** 2 KB(s); DOIs: 10.1038/s41467-021-23778-6, 10.1038/s41467-024-45778-y

### Stage 4 — aggregate_5mcpg_sites

**Goal:** aggregate per-read 5mCpG calls into filtered CpG-site frequencies

**EDAM operation:** http://edamontology.org/operation_3802

**Inputs:** modbam/5mcpg-per-read · **Outputs:** bed-methyl/5mcpg-per-site

**Candidate leaf skills:** `modkit-modification-pileup` (primary), `per-site-methylation-frequency-aggregation`, `modkit-modification-site-counting`, `bedmethyl-frequency-aggregation`, `low-coverage-site-filtering`

**Tools (primary):** Dogme, Dorado, minimap2, modkit

**Other candidate tools:** DeepSignal-plant, Guppy, PyTorch, Tombo, Nextflow, DSS, Singularity, ccsmeth, ccsmethphase, clair3, pbmm2, whatshap, DeepMod2, bedtools, samtools

**Grounding:** 4 KB(s); DOIs: 10.1038/s41467-021-26278-9, 10.1038/s41467-023-39784-9, 10.1038/s41467-024-45778-y, 10.1093/bioinformatics/btag066

### Stage 5 — validate_methylation_profile

**Goal:** compare nanopore 5mCpG frequencies with orthogonal CpG measurements

**EDAM operation:** http://edamontology.org/operation_2409

**Inputs:** bed-methyl/5mcpg-per-site, methylation-table/orthogonal-cpg-reference · **Outputs:** validation-report/5mcpg-correlation

**Candidate leaf skills:** `bisulfite-methylation-ground-truth-matching` (primary), `cross-platform-methylation-correlation-analysis--50dd8f2e672836d6`, `bedtools-genomic-interval-intersection`, `pearson-correlation-of-per-site-methylation-levels`

**Tools (primary):** DeepMod2, Dorado, bedtools, modkit

**Other candidate tools:** minimap2, samtools

**Grounding:** 1 KB(s); DOIs: 10.1038/s41467-024-45778-y

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.

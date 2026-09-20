---
name: long-read-structural-variant-workflow
description: 'Use when long-read FASTQ data must be aligned, called by multiple structural-variant
  methods, verified per caller for read-depth support of copy-number events, merged,
  and compared with an external truth or high-confidence call set.

  '
license: CC-BY-4.0
metadata:
  kind: composite-workflow
  techniques: []
  stage_count: 5
  member_skills:
  - long-read-genome-alignment--8cec567f57b0cb03
  - long-read-alignment-processing
  - long-read-genome-alignment-minimap2
  - invocation-and-execution-of-long-read-aligners
  - nanopore-structural-variant-calling
  - structural-variant-calling-long-reads
  - structural-variant-calling-from-bam
  - structural-variant-calling-from-long-read-sequencing
  - coverage-based-sv-filtering
  - cnv-call-filtering-by-read-depth
  - read-depth-coverage-filtering
  - deletion-duplication-coverage-filtering
  - multi-caller-sv-callset-merging
  - sv-call-filtering-by-caller-support
  - sv-variant-overlap-collapse
  - structural-variant-vcf-merging
  - truvari-parameter-tuning-evaluation
  - truvari-callset-comparison-against-giab
  - vcf-callset-precision-recall-computation
  - structural-variant-truth-set-benchmarking
  - truvari-benchmark-vcf-comparison
  member_tools:
  - CuteSV
  - NGMLR
  - NanoVar
  - SVIM
  - Sniffles
  - disCoverage
  - lra
  - minimap2
  - nanopore_pipeline_21
  - mosdepth
  - SURVIVOR
  - DeBreak
  - MAMnet
  - PBSIM
  - SVDSS
  - SVIM-asm
  - Sniffles2
  - Truvari
  - VISOR
  - VolcanoSV
  - pbsv
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

# Long-Read Structural-Variant Calling and Comparison

## Summary

Align long reads, call per-caller variants, verify depth support of copy-number calls, merge a consensus call set, and report truth-aware benchmark metrics.


## When to use

Use when long-read FASTQ data must be aligned, called by multiple structural-variant methods, verified per caller for read-depth support of copy-number events, merged, and compared with an external truth or high-confidence call set.


## When NOT to use

- The data is not this platform.
- You need a single atomic step, not the full pipeline (use the leaf skill directly via the router).

## Stages

### Stage 1 — align_long_reads

**Goal:** align long reads to the reference genome for structural-variant calling

**EDAM operation:** http://edamontology.org/operation_2928

**Inputs:** fastq/long-read, fasta/reference-genome · **Outputs:** bam/long-read-alignment

**Candidate leaf skills:** `long-read-genome-alignment--8cec567f57b0cb03` (primary), `long-read-alignment-processing`, `long-read-genome-alignment-minimap2`, `invocation-and-execution-of-long-read-aligners`

**Tools (primary):** CuteSV, NGMLR, NanoVar, SVIM, Sniffles, disCoverage, lra, minimap2, nanopore_pipeline_21

**Other candidate tools:** AgileStructure, MakeSVGenome, Samtools, WINE, LongRanger, Nextflow, StringTie, fastp, stlfr2supernova, PBSIM, PBSIM2, Winnowmap2, invMap

**Grounding:** 4 KB(s); DOIs: 10.1016/j.jare.2022.10.012, 10.1093/bioinformatics/btad726, 10.1093/bioinformatics/btag294, 10.1093/gigascience/giac099

### Stage 2 — call_structural_variants

**Goal:** generate separate structural-variant call sets from the alignments

**EDAM operation:** http://edamontology.org/operation_3227

**Inputs:** bam/long-read-alignment, fasta/reference-genome · **Outputs:** vcf/per-caller-sv

**Candidate leaf skills:** `nanopore-structural-variant-calling` (primary), `structural-variant-calling-long-reads`, `structural-variant-calling-from-bam`, `structural-variant-calling-from-long-read-sequencing`

**Tools (primary):** CuteSV, NGMLR, NanoVar, SVIM, Sniffles, disCoverage, lra, minimap2, mosdepth, nanopore_pipeline_21

**Other candidate tools:** RetroInspector, SAMtools, SURVIVOR, Sniffles2, Delly, FUDGE, LAST, Lumpy, Manta, NanoFG, NanoSV, NanoSim, Wtdbg2, bedtools, DASV, PBSV

**Grounding:** 4 KB(s); DOIs: 10.1016/j.jare.2022.10.012, 10.1038/s41467-020-16641-7, 10.1038/s41598-025-98847-7, 10.1093/bib/bbae269

### Stage 3 — filter_per_caller_calls

**Goal:** verify read-depth support of the copy-number calls in each per-caller call set

**EDAM operation:** http://edamontology.org/operation_3227

**Inputs:** vcf/per-caller-sv, bam/long-read-alignment · **Outputs:** vcf/filtered-per-caller-sv, support-table/per-caller-sv

**Candidate leaf skills:** `coverage-based-sv-filtering` (primary), `cnv-call-filtering-by-read-depth`, `read-depth-coverage-filtering`, `deletion-duplication-coverage-filtering`

**Tools (primary):** CuteSV, Mosdepth, NGMLR, NanoVar, SVIM, Sniffles, disCoverage, lra, minimap2

**Other candidate tools:** Delly, HapCUT2, Longshot

**Grounding:** 2 KB(s); DOIs: 10.1016/j.jare.2022.10.012, 10.1038/s41467-019-12493-y

### Stage 4 — merge_consensus_calls

**Goal:** consolidate the filtered per-caller structural variants

**EDAM operation:** http://edamontology.org/operation_3227

**Inputs:** vcf/filtered-per-caller-sv · **Outputs:** vcf/consensus-sv

**Candidate leaf skills:** `multi-caller-sv-callset-merging` (primary), `sv-call-filtering-by-caller-support`, `sv-variant-overlap-collapse`, `structural-variant-vcf-merging`

**Tools (primary):** CuteSV, NGMLR, NanoVar, SURVIVOR, SVIM, Sniffles, disCoverage, lra, minimap2

**Other candidate tools:** Delly, Lumpy, Pindel, cn.MOPs, ConsensuSV-ONT, Docker, Nextflow, Truvari

**Grounding:** 3 KB(s); DOIs: 10.1016/j.jare.2022.10.012, 10.1038/ncomms14061, 10.1038/s41598-025-01486-1

### Stage 5 — benchmark_callset

**Goal:** compare consensus calls with truth under truth-set-specific metric limits

**EDAM operation:** http://edamontology.org/operation_3227

**Inputs:** vcf/consensus-sv, vcf/truth-sv · **Outputs:** benchmark-table/structural-variant

**Candidate leaf skills:** `truvari-parameter-tuning-evaluation` (primary), `truvari-callset-comparison-against-giab`, `vcf-callset-precision-recall-computation`, `structural-variant-truth-set-benchmarking`, `truvari-benchmark-vcf-comparison`

**Tools (primary):** DeBreak, MAMnet, NanoVar, PBSIM, SURVIVOR, SVDSS, SVIM-asm, Sniffles2, Truvari, VISOR, VolcanoSV, cuteSV, pbsv

**Other candidate tools:** BWA, Blackbird, SVIM, Sniffles, dysgu, svbench, RetroInspector, ConsensuSV-ONT, Docker, Nextflow

**Grounding:** 5 KB(s); DOIs: 10.1038/s41467-024-51282-0, 10.1038/s41598-025-01486-1, 10.1038/s41598-025-98847-7, 10.1093/bioadv/vbaf151 …

## Grounding

Each stage carries the `kb_slugs`/`dois` of the leaves it draws on. Ground any stage against its source paper with the collection's `/ground` command or `bin/perspicacite_kb_bind.py` (Perspicacité KB; serverless local-clone fallback).

## Verification contract

`workflow.yaml` declares the stage graph and its typed outputs; the final stage emits the master deliverable. Automatic grading of that graph (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages as an outline — the structure is validated, the execution is not.

## Provenance

Generated by `compose_workflows.py` (semantic binding + EDAM-aware primary selection). `derived_from_workflows` lists the ASB per-paper workflows whose structure corroborated this pipeline; it is a provenance record, and no ablation experiment consuming it is released. Validated structurally by `validate_workflows.py` through `release_gate.py`: the collection is the hard-gated artefact and this workflows layer is additive.

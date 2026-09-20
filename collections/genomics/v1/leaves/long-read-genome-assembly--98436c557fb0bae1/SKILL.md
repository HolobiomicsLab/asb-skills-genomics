---
name: long-read-genome-assembly
description: Use when assembling low-coverage (~5x) PacBio HiFi reads de novo and
  a high-quality reference genome (e.g., CHM13 v2.0) is available to partition reads
  chromosome-by-chromosome before assembly.
when_to_use_negative:
- Input is high-coverage (>~25-30x) HiFi data where hifiasm alone already resolves
  the genome without reference-guided binning.
- No high-quality, closely related reference genome is available — HiFiCCL is exclusively
  reference-guided and relies on mapping reads to a reference to partition chromosomes.
- 'The goal is chromosome-level scaffolding that must preserve true SVs: RagTag scaffolding
  overlooked true SVs (only 62.5% of inversion SVs detected), and Hi-C/nanopore integration
  is not yet implemented.'
edam_operation: http://edamontology.org/operation_0524
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HiFiCCL
  role: Reference-guided framework that bins reads chromosome-by-chromosome and invokes
    hifiasm on each chromosomal bin
  repo: https://github.com/zjjbuqi/HiFiCCL.git
- name: hifiasm
  role: Base de novo assembler run in --primary mode on each chromosomal cluster
  repo: https://github.com/chhylp123/hifiasm
- name: minimap2
  role: Maps HiFi reads to the reference genome for chromosomal class assignment
  repo: https://github.com/lh3/minimap2
- name: seqtk
  role: Downsamples HiFi read sets to ~5x coverage before assembly
  repo: https://github.com/lh3/seqtk
- name: pysam
  role: Reads/manipulates SAM/BAM alignments during label-graph construction (v0.21.0)
  repo: https://github.com/pysam-developers/pysam
- name: QUAST
  role: Evaluates contigs against high-quality published assemblies (v5.2.0)
- name: BUSCO
  role: Assesses gene completeness with the vertebrate single-copy ortholog database
    (v5.4.7)
- name: Merqury
  role: Assesses assembly completeness and QV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:67d4ce517091e19ea81b7f6bdbc3c4ae2a2a09c4eb890496b0c079670da863ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL.git
  tools:
  - BUSCO
  - HiFiCCL
  - Hifiasm
  - Merqury
  - QUAST
  - hifiasm
  - minimap2
  - pysam
  - seqtk
  edam_operation: http://edamontology.org/operation_0524
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# long-read-genome-assembly

## Summary

Reference-guided, chromosome-by-chromosome de novo assembly of low-coverage (~5x) PacBio HiFi reads using HiFiCCL combined with hifiasm, which partitions whole-genome reads into chromosomal bins with a high-quality reference and assembles each bin de novo to reduce misassembled contig length relative to hifiasm alone.

## When to use

Use when assembling low-coverage (~5x) PacBio HiFi reads de novo and a high-quality reference genome (e.g., CHM13 v2.0) is available to partition reads chromosome-by-chromosome before assembly.

## When NOT to use

- Input is high-coverage (>~25-30x) HiFi data where hifiasm alone already resolves the genome without reference-guided binning.
- No high-quality, closely related reference genome is available — HiFiCCL is exclusively reference-guided and relies on mapping reads to a reference to partition chromosomes.
- The goal is chromosome-level scaffolding that must preserve true SVs: RagTag scaffolding overlooked true SVs (only 62.5% of inversion SVs detected), and Hi-C/nanopore integration is not yet implemented.

## Inputs

- PacBio HiFi read set in FASTQ/FASTA (e.g., HG002 SRR10382244, NA19240 SRR14611231, downsampled to ~5x with seqtk)
- High-quality reference genome FASTA for read partitioning (e.g., CHM13 v2.0)
- Optional pangenome graph / chromosomal classes for Pangenome Label Graph (PLG) construction
- Published high-quality assemblies to serve as QUAST evaluation references (e.g., GRCh38, CHM13)

## Outputs

- Per-chromosome de novo contig assemblies combined into a primary assembly (hifiasm .bp.p_ctg GFA/FASTA contigs)
- Chromosomal read bins based on final assigned classes
- QUAST metrics: contig size, number of contigs, misassembled contig length, NG50, NGA50
- BUSCO 5.4.7 vertebrate ortholog completeness scores and Merqury completeness/QV values
- assembly_evaluation_metrics.csv with MCL reduction percentages, BUSCO/Merqury/NG50 deltas, and runtime ratios

## How to apply

Map the HiFi reads to a high-quality reference (e.g., CHM13 v2.0) with minimap2 and use pysam to process alignments; HiFiCCL then builds an Alignment Label Graph (ALG), corrects sequence classes via a heuristic Z-drop-inspired ALG search and translocation class correction, harmonizes classes with a Pangenome Label Graph (PLG) using bidirectional DFS, and clusters reads into chromosomal bins. Assemble each chromosomal bin de novo with hifiasm run in --primary mode (HiFiCCL invokes hifiasm as the base assembler). As a baseline, assemble the same read set with hifiasm alone (0.19.5-r592, -t20 --primary); reads can be downsampled to ~5x with seqtk 1.3-r106 first. Evaluate with QUAST v5.2.0 (contig size, number of contigs, misassembled contig length, NG50, NGA50), BUSCO 5.4.7 with the vertebrate single-copy ortholog database, and Merqury for completeness/QV; record wall time and maximum resident set size for both pipelines. Success criterion: HiFiCCL-hifiasm reduces misassembled contig length versus hifiasm alone (reported average 21.19%, up to 38.58% across 45 ~5x human datasets) with improved BUSCO, Merqury and NG50 values, written to assembly_evaluation_metrics.csv.

## Related tools

- **HiFiCCL** (Reference-guided framework that bins reads chromosome-by-chromosome and invokes hifiasm on each chromosomal bin) — https://github.com/zjjbuqi/HiFiCCL.git
- **hifiasm** (Base de novo assembler run in --primary mode on each chromosomal cluster) — https://github.com/chhylp123/hifiasm
- **minimap2** (Maps HiFi reads to the reference genome for chromosomal class assignment) — https://github.com/lh3/minimap2
- **seqtk** (Downsamples HiFi read sets to ~5x coverage before assembly) — https://github.com/lh3/seqtk
- **pysam** (Reads/manipulates SAM/BAM alignments during label-graph construction (v0.21.0)) — https://github.com/pysam-developers/pysam
- **QUAST** (Evaluates contigs against high-quality published assemblies (v5.2.0))
- **BUSCO** (Assesses gene completeness with the vertebrate single-copy ortholog database (v5.4.7))
- **Merqury** (Assesses assembly completeness and QV)

## Examples

```
hificcl.py -f HG002_5x.hifi.fq -r CHM13v2.0.fasta -t 20
```

## Evaluation signals

- QUAST misassembled contig length (MCL) for HiFiCCL-hifiasm is lower than the hifiasm-alone baseline on the same reads (e.g., HG002 ~5x: 331.84 -> 274.16 Mb, a 17.38% reduction; NA19240: 245.96 -> 179.15 Mb, 27.16%).
- Across the 45 ~5x human datasets, average MCL reduction ~21.19% and up to 38.58%; a deviation far below this range suggests binning or class correction failed.
- BUSCO vertebrate ortholog completeness and Merqury completeness/QV should improve over hifiasm alone, not regress.
- NG50/NGA50 from QUAST v5.2.0 should be higher than the hifiasm-alone baseline.
- Each chromosomal bin yields a de novo assembly based on overlap relationships; per-bin contigs concatenate into a genome-scale primary assembly, with wall time and max RSS recorded for the runtime ratio.

## Limitations

- At ~5x coverage, many genomic regions remain missing; completeness gaps are unresolved by this approach.
- RagTag scaffolding of HiFiCCL-hifiasm assemblies relies on reference alignments and may overlook true SVs — only 62.5% of inversion SVs were detected.
- Plant genomes with higher genetic diversity were tested on only two samples; broader plant evaluation has not been performed, and integration of Hi-C or nanopore data is not yet implemented.
- Requires a high-quality reference genome (e.g., CHM13 v2.0) for chromosome partitioning; reference choice affects binning quality.

## Evidence

- [abstract] HiFiCCL combined with hifiasm reduces the length of misassembled contigs relative to hifiasm: "HiFiCCL combined with hifiasm reduces the length of misassembled contigs relative to hifiasm"
- [intro] whole-genome reads are partitioned by chromosome using the high-quality reference genome and then assembled individually: "whole‐genome reads are partitioned by chromosome using the high‐quality reference genome and then assembled individually"
- [abstract] HiFiCCL is proposed, the first assembly framework specifically designed for low-coverage high-fidelity reads: "HiFiCCL is proposed, the first assembly framework specifically designed for low-coverage high-fidelity reads"
- [methods] hificcl.py -f <reads> -r CHM13v2.0.fasta -t 20: [evidence span withheld — anchor status: not_found]
- [readme] Assemble inbred/homozygous genomes (-l0 disables duplication purging): "# Assemble inbred/homozygous genomes (-l0 disables duplication purging)
hifiasm -o CHM13.asm"

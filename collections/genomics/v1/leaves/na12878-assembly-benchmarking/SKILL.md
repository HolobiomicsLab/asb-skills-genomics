---
name: na12878-assembly-benchmarking
description: Use when comparing one or more genome assemblies (e.g., DFS hybrid scaffolds
  of NA12878) against a reference genome and public assemblies to quantify continuity
  (N50) and structural errors (misjoins, relocations, translocations, inversions).
when_to_use_negative:
- Input is raw reads or contig-level data that has not been assembled/scaffolded into
  an assembly to be evaluated against a reference
- No reference genome is available, since the benchmarking relies on 1-to-1 whole-genome
  alignment of each assembly against the human reference
- The goal is gene or variant annotation of an assembly rather than structural accuracy/continuity
  comparison across assemblies
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Nucmer
  role: Whole-genome alignment of the DFS scaffolds and comparison assemblies against
    the human reference to obtain pairwise nucleotide alignments
- name: Gage
  role: 'Computation of assembly quality statistics (N50, alignment identity, structural-error
    percentages: misjoins, relocations, translocations, inversions) from the alignments'
- name: Fast-SG
  role: Source of the DFS hybrid scaffolds of NA12878 being benchmarked against reference
    assemblies
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:38ee945e7a65ce0b6378e60633103da2789fb3cd090babf8e9acc81f016bd36b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - Gage
  - Nucmer
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# na12878-assembly-benchmarking

## Summary

Benchmark the structural accuracy and continuity of genome assemblies (e.g., the Fast-SG DFS hybrid assembly of NA12878) by whole-genome alignment with Nucmer against the human reference and computation of Gage statistics (N50, identity, structural-error percentages) in comparison to public assemblies (Canu, MaSuRCA, 10X, DOVETAIL).

## When to use

Use when comparing one or more genome assemblies (e.g., DFS hybrid scaffolds of NA12878) against a reference genome and public assemblies to quantify continuity (N50) and structural errors (misjoins, relocations, translocations, inversions).

## When NOT to use

- Input is raw reads or contig-level data that has not been assembled/scaffolded into an assembly to be evaluated against a reference
- No reference genome is available, since the benchmarking relies on 1-to-1 whole-genome alignment of each assembly against the human reference
- The goal is gene or variant annotation of an assembly rather than structural accuracy/continuity comparison across assemblies

## Inputs

- Assembly scaffolds to evaluate (e.g., DFS hybrid scaffolds of NA12878 from Fast-SG scaffolding)
- Human reference genome sequence (FASTA)
- Optional comparison assemblies: public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies (FASTA)

## Outputs

- Nucmer pairwise whole-genome alignments of each assembly against the reference
- Gage statistics per assembly (N50, alignment identity, structural-error percentages)
- Consolidated comparison table gage_statistics_table.csv with one row per assembly

## How to apply

Load the assembly scaffolds (e.g., DFS scaffolds from the Fast-SG hybrid pipeline) and the human reference genome sequence, then run Nucmer whole-genome alignment of the scaffolds against the reference to obtain pairwise nucleotide alignments. Repeat the Nucmer alignment for each optional comparison assembly (public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies). Run Gage on each alignment to compute assembly quality statistics including N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions). Assemble the per-assembly statistics into a single comparison table (gage_statistics_table.csv). In this workflow the accepted accuracy bar is that less than 1% of the total 1-to-1 alignment length is involved in structural errors; all evaluated pipelines, including the DFS hybrid assembly, met it.

## Related tools

- **Nucmer** (Whole-genome alignment of the DFS scaffolds and comparison assemblies against the human reference to obtain pairwise nucleotide alignments)
- **Gage** (Computation of assembly quality statistics (N50, alignment identity, structural-error percentages: misjoins, relocations, translocations, inversions) from the alignments)
- **Fast-SG** (Source of the DFS hybrid scaffolds of NA12878 being benchmarked against reference assemblies) — https://github.com/adigenova/fast-sg

## Evaluation signals

- gage_statistics_table.csv exists and contains, for the DFS scaffolds, the N50, identity, and structural-error percentage values
- The DFS scaffolds' statistics appear in the same comparison table as the optional reference assemblies' rows (Canu, MaSuRCA, 10X, DOVETAIL)
- Structural errors (misjoins, relocations, translocations, inversions) account for less than 1% of the total 1-to-1 alignment length for a structurally accurate assembly
- Nucmer alignments were produced for every assembly included in the comparison table (no missing rows in gage_statistics_table.csv)

## Limitations

- The benchmark assesses structural accuracy and continuity (N50, identity, structural errors) but not base-level completeness or other assembly-quality dimensions
- Structural-error estimation depends on the chosen reference; errors relative to the human reference may not reflect true structural accuracy in divergent regions
- Comparison against Canu, MaSuRCA, 10X, and DOVETAIL assemblies is optional; without them only intrinsic statistics are available

## Evidence

- [results] Whole-genome alignment comparison shows all evaluated pipelines, including the DFS hybrid assembly, are structurally highly accurate, with less than 1% of the total 1-to-1 alignment length involved in structural errors.: [evidence span withheld — anchor status: not_found]
- [methods] Run Gage on each alignment to compute assembly quality statistics, including N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions).: [evidence span withheld — anchor status: not_found]
- [abstract] Fast-SG can construct the graph from either short or long reads, and a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources produced long-range and accurate reconstructions of the NA12878 genome.: "a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions of the genomes of Arabidopsis"
- [methods] The wiki page provides a full example of the use of Fast-SG for the hybrid assembly of NA12878.: "The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878."

---
name: gage-assembly-statistics-computation
description: Use when assembled scaffolds (e.g. DFS hybrid assembly output for NA12878)
  must be benchmarked for structural accuracy and continuity against a reference genome
  and/or competing assemblies (Canu, MaSuRCA, 10X, DOVETAIL).
when_to_use_negative:
- Input is raw short- or long-read data rather than assembled scaffolds/contigs that
  can be aligned to a reference
- No reference genome is available, since Nucmer/Gage statistics here are computed
  by whole-genome alignment against the human reference
- The goal is contig graph construction or scaffolding (use Fast-SG) rather than post-hoc
  evaluation of already-assembled sequences
edam_operation: http://edamontology.org/operation_0544
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Nucmer
  role: Whole-genome aligner producing the pairwise nucleotide alignments of each
    assembly against the human reference that Gage scores
- name: Gage
  role: Computes assembly quality statistics (N50, identity, structural-error percentages)
    from each Nucmer alignment
- name: Fast-SG
  role: Produces the DFS hybrid scaffolds of NA12878 that are evaluated by the Nucmer/Gage
    comparison
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:9e4e28e578d6293e556759f0ebcfe7ab0724894e4c3864cce4a8e408e747b2cb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - Gage
  - Nucmer
  edam_operation: http://edamontology.org/operation_0544
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gage-assembly-statistics-computation

## Summary

Computes reference-based assembly quality statistics (N50, alignment identity, structural-error percentages) by running Nucmer whole-genome alignment of assembled scaffolds against the human reference and evaluating each alignment with Gage, as used to benchmark the Fast-SG DFS hybrid assembly of NA12878.

## When to use

Use when assembled scaffolds (e.g. DFS hybrid assembly output for NA12878) must be benchmarked for structural accuracy and continuity against a reference genome and/or competing assemblies (Canu, MaSuRCA, 10X, DOVETAIL).

## When NOT to use

- Input is raw short- or long-read data rather than assembled scaffolds/contigs that can be aligned to a reference
- No reference genome is available, since Nucmer/Gage statistics here are computed by whole-genome alignment against the human reference
- The goal is contig graph construction or scaffolding (use Fast-SG) rather than post-hoc evaluation of already-assembled sequences

## Inputs

- DFS hybrid assembly scaffolds (FASTA, output of the Fast-SG scaffolding step)
- Human reference genome sequence (FASTA)
- Optional public NA12878 comparison assemblies (Canu, MaSuRCA, 10X, DOVETAIL) as FASTA

## Outputs

- gage_statistics_table.csv containing per-assembly N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions)
- Per-assembly Gage statistics from Nucmer whole-genome alignments

## How to apply

Load the DFS scaffolds produced by the Fast-SG hybrid assembly (task_005 output) together with the human reference genome sequence. Run Nucmer whole-genome alignment of the DFS scaffolds against the reference to obtain pairwise nucleotide alignments, and repeat the identical Nucmer alignment for each optional comparison assembly (public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies) so all assemblies are scored on the same footing. Run Gage on each alignment to compute assembly quality statistics: N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions). Aggregate the per-assembly statistics into a single comparison table saved as gage_statistics_table.csv. In this benchmark, structural accuracy was judged by the fraction of the total 1-to-1 alignment length involved in structural errors, with less than 1% indicating a structurally highly accurate assembly.

## Related tools

- **Nucmer** (Whole-genome aligner producing the pairwise nucleotide alignments of each assembly against the human reference that Gage scores)
- **Gage** (Computes assembly quality statistics (N50, identity, structural-error percentages) from each Nucmer alignment)
- **Fast-SG** (Produces the DFS hybrid scaffolds of NA12878 that are evaluated by the Nucmer/Gage comparison) — https://github.com/adigenova/fast-sg

## Evaluation signals

- gage_statistics_table.csv exists and is valid CSV
- The table contains a row for the DFS scaffolds with N50, alignment identity, and structural-error percentage values
- Rows for the optional reference assemblies (Canu, MaSuRCA, 10X, DOVETAIL) are present in the same table for comparison
- Structural errors (misjoins, relocations, translocations, inversions) account for less than 1% of the total 1-to-1 alignment length for a structurally accurate assembly
- Nucmer alignments completed for every assembly being compared (no missing alignment output before Gage runs)

## Limitations

- Statistics are reference-relative: accuracy is measured by alignment to the chosen human reference, so a different reference changes the error fractions
- Benchmark figures apply to the NA12878 genome assembled with shallow long-read coverage (5X) and moderate computational resources; results may differ for other genomes or coverage
- The 1-to-1 alignment framework captures only structural errors observable against the reference (misjoins, relocations, translocations, inversions), not unaligned or novel sequence quality

## Evidence

- [results] all evaluated pipelines, including the DFS hybrid assembly, are structurally highly accurate, with less than 1% of the total 1-to-1 alignment length involved in structural errors.: [evidence span withheld — anchor status: not_found]
- [methods] The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878.: "The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878."
- [abstract] a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions of the genomes of Arabidopsis: "a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions of the genomes of Arabidopsis"

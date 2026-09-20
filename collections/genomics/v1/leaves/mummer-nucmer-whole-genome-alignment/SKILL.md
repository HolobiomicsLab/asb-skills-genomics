---
name: mummer-nucmer-whole-genome-alignment
description: Use when assembled scaffolds or contigs (e.g., DFS hybrid NA12878 scaffolds)
  must be structurally validated against a reference genome and compared to competing
  assemblies (Canu, MaSuRCA, 10X, DOVETAIL) via whole-genome alignment statistics.
when_to_use_negative:
- Input is unassembled reads rather than finished scaffolds/contigs — this skill evaluates
  assembly accuracy and cannot itself scaffold; use Fast-SG to construct the scaffolding
  graph and produce the DFS scaffolds first.
- No reference genome or comparison assemblies are available — the Gage structural-error
  classification (misjoins, relocations, translocations, inversions) is defined relative
  to the reference whole-genome alignment.
- The goal is k-mer counting or scaffolding-graph construction rather than assembly
  evaluation — use KMC/Fast-SG, which Fast-SG relies on via its alignment-free algorithm.
edam_operation: http://edamontology.org/operation_0192
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Nucmer
  role: Whole-genome aligner producing the pairwise nucleotide alignments of DFS scaffolds
    (and comparison assemblies) against the human reference that Gage consumes
- name: Gage
  role: Computes assembly quality statistics (N50, alignment identity, structural-error
    percentages) from each Nucmer alignment
- name: Fast-SG
  role: Upstream alignment-free scaffolding-graph tool whose DFS hybrid NA12878 scaffolds
    are the subject of the Nucmer/Gage structural-accuracy evaluation
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:b21f3d91fdbc6ca6a9fc1acf70a941a2dd0f2f9e944edb6db67bc63716d10d64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - Gage
  - Nucmer
  edam_operation: http://edamontology.org/operation_0192
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# mummer-nucmer-whole-genome-alignment

## Summary

Structural-accuracy evaluation of genome assemblies (e.g., the Fast-SG/DFS hybrid assembly of NA12878) by whole-genome Nucmer alignment against the human reference, followed by Gage computation of N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions).

## When to use

Use when assembled scaffolds or contigs (e.g., DFS hybrid NA12878 scaffolds) must be structurally validated against a reference genome and compared to competing assemblies (Canu, MaSuRCA, 10X, DOVETAIL) via whole-genome alignment statistics.

## When NOT to use

- Input is unassembled reads rather than finished scaffolds/contigs — this skill evaluates assembly accuracy and cannot itself scaffold; use Fast-SG to construct the scaffolding graph and produce the DFS scaffolds first.
- No reference genome or comparison assemblies are available — the Gage structural-error classification (misjoins, relocations, translocations, inversions) is defined relative to the reference whole-genome alignment.
- The goal is k-mer counting or scaffolding-graph construction rather than assembly evaluation — use KMC/Fast-SG, which Fast-SG relies on via its alignment-free algorithm.

## Inputs

- DFS hybrid assembly scaffolds in FASTA format (output of the Fast-SG/Links scaffolding workflow, task_005)
- Human reference genome sequence in FASTA format
- Optional comparison assemblies in FASTA format: public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies

## Outputs

- gage_statistics_table.csv — a single comparison table with per-assembly rows containing N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions)
- Per-assembly Nucmer pairwise alignment files of scaffolds versus the human reference

## How to apply

Load the DFS scaffolds (produced by the Fast-SG hybrid scaffolding pipeline) and the human reference genome sequence. Run Nucmer whole-genome alignment of the DFS scaffolds against the human reference to obtain pairwise nucleotide alignments, and repeat the alignment for each optional comparison assembly (public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies). Run Gage on each alignment to compute assembly quality statistics including N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions). Aggregate the per-assembly statistics into a single comparison table saved as gage_statistics_table.csv. The decision criterion from this study is that a structurally accurate assembly has less than 1% of the total 1-to-1 alignment length involved in structural errors.

## Related tools

- **Nucmer** (Whole-genome aligner producing the pairwise nucleotide alignments of DFS scaffolds (and comparison assemblies) against the human reference that Gage consumes)
- **Gage** (Computes assembly quality statistics (N50, alignment identity, structural-error percentages) from each Nucmer alignment)
- **Fast-SG** (Upstream alignment-free scaffolding-graph tool whose DFS hybrid NA12878 scaffolds are the subject of the Nucmer/Gage structural-accuracy evaluation) — https://github.com/adigenova/fast-sg

## Evaluation signals

- gage_statistics_table.csv exists and contains a row for the DFS scaffolds with N50, alignment identity, and structural-error percentage populated.
- The DFS scaffolds' row sits alongside rows for the optional Canu, MaSuRCA, 10X, and DOVETAIL assemblies in the same table, enabling direct comparison.
- Each Nucmer alignment completed successfully for all input assemblies before Gage statistics were computed (no missing per-assembly rows).
- Structural-error percentages (misjoins, relocations, translocations, inversions) sum to less than 1% of the total 1-to-1 alignment length for a structurally accurate assembly, as observed for all evaluated pipelines including DFS.
- Reported identity values are on a nucleotide-alignment scale consistent across all assemblies compared in the table.

## Limitations

- Structural-error statistics are relative to the chosen human reference; assembly features absent from or variant in the reference can appear as misjoins, relocations, translocations, or inversions.
- Gage statistics from 1-to-1 whole-genome alignments do not by themselves validate unaligned regions or collapsed/expanded repeats in the assembly.
- The upstream DFS scaffolds depend on Fast-SG, whose synthetic-library insert size is restricted to the actual long-read size and whose gap-size estimation with 10X/DOVETAIL-style long-range mate-pair linking is more challenging because insert sizes span a range and read-pair orientation may be unknown.

## Evidence

- [other] Whole-genome alignment comparison shows all evaluated pipelines, including the DFS hybrid assembly, are structurally highly accurate, with less than 1% of the total 1-to-1 alignment length involved in structural errors.: [evidence span withheld — anchor status: not_found]
- [methods] Run Nucmer whole-genome alignment of the DFS scaffolds against the human reference to obtain pairwise nucleotide alignments.: [evidence span withheld — anchor status: not_found]
- [readme] Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads.: "Fast-SG is an alignment-free algorithm for ultrafast scaffolding graph construction from short or long reads."
- [methods] The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878.: "The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878."

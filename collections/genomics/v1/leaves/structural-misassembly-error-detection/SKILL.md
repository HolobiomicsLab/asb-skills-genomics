---
name: structural-misassembly-error-detection
description: Use when a candidate assembly (FASTA scaffolds) must be checked for structural
  errors against a reference genome or compared head-to-head with other assemblies
  using alignment-based Gage statistics.
when_to_use_negative:
- Input is raw long or short reads rather than an assembled scaffold FASTA — use the
  Fast-SG scaffolding-graph construction step instead of Gage evaluation
- The goal is insert/gap-size estimation with 10X or DOVETAIL long-range mate-pair
  technologies, which the article notes is challenging because linking information
  spans a range of insert sizes with possibly unknown read-pair orientation
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Nucmer
  role: Runs the whole-genome alignment of the DFS scaffolds (and each optional comparison
    assembly) against the human reference to obtain pairwise nucleotide 1-to-1 alignments
- name: Gage
  role: 'Consumes each Nucmer alignment to compute assembly quality statistics: N50,
    alignment identity, and structural-error percentages (misjoins, relocations, translocations,
    inversions)'
- name: Fast-SG
  role: Produces the DFS hybrid scaffolds of NA12878 that are the primary assembly
    being evaluated for structural accuracy
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:1c45c9ef3353664c1e38d6305ce7abe4bf99da6cb08f06656290c3686b87ea7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - Gage
  - Nucmer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# structural-misassembly-error-detection

## Summary

Quantifies structural accuracy of genome assemblies (e.g., the Fast-SG DFS hybrid assembly of NA12878) by whole-genome Nucmer alignment against a reference and Gage statistics (N50, identity, misjoins, relocations, translocations, inversions), benchmarking against public Canu, MaSuRCA, 10X, and DOVETAIL assemblies.

## When to use

Use when a candidate assembly (FASTA scaffolds) must be checked for structural errors against a reference genome or compared head-to-head with other assemblies using alignment-based Gage statistics.

## When NOT to use

- Input is raw long or short reads rather than an assembled scaffold FASTA — use the Fast-SG scaffolding-graph construction step instead of Gage evaluation
- The goal is insert/gap-size estimation with 10X or DOVETAIL long-range mate-pair technologies, which the article notes is challenging because linking information spans a range of insert sizes with possibly unknown read-pair orientation

## Inputs

- Assembly scaffolds in FASTA format (e.g., DFS hybrid assembly scaffolds from the Fast-SG pipeline)
- Human reference genome sequence (FASTA)
- Optional comparison assemblies: public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies (FASTA)

## Outputs

- Nucmer whole-genome pairwise (1-to-1) alignment files of each assembly against the reference
- Per-assembly Gage statistics (N50, alignment identity, structural-error percentages)
- gage_statistics_table.csv — a single comparison table with a row per assembly including N50, identity, and structural-error percentage

## How to apply

Load the scaffolds produced by the assembly step (e.g., DFS scaffolds from task_005) together with the human reference genome sequence. Run Nucmer whole-genome alignment of the scaffolds against the reference to obtain pairwise nucleotide (1-to-1) alignments; repeat the same Nucmer run for each optional comparison assembly (public Canu, MaSuRCA, 10X, DOVETAIL NA12878 assemblies). Run Gage on each alignment to compute assembly quality statistics including N50, alignment identity, and structural-error percentages for misjoins, relocations, translocations, and inversions. Collate the per-assembly statistics into a single comparison table (gage_statistics_table.csv) so the DFS scaffolds' row can be read against the reference assemblies' rows. Judge success by whether structural errors remain below the paper's threshold of <1% of total 1-to-1 alignment length, indicating a structurally highly accurate assembly.

## Related tools

- **Nucmer** (Runs the whole-genome alignment of the DFS scaffolds (and each optional comparison assembly) against the human reference to obtain pairwise nucleotide 1-to-1 alignments)
- **Gage** (Consumes each Nucmer alignment to compute assembly quality statistics: N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions))
- **Fast-SG** (Produces the DFS hybrid scaffolds of NA12878 that are the primary assembly being evaluated for structural accuracy) — https://github.com/adigenova/fast-sg

## Evaluation signals

- gage_statistics_table.csv exists and contains, for the DFS scaffolds, the N50, identity, and structural-error percentage values
- Structural errors account for less than 1% of the total 1-to-1 alignment length (the paper's threshold for 'structurally highly accurate')
- The table includes rows for the optional reference assemblies (Canu, MaSuRCA, 10X, DOVETAIL) so the DFS scaffolds' statistics can be compared in the same table
- Every evaluated assembly has a completed Nucmer alignment against the human reference before Gage statistics are computed

## Limitations

- Structural accuracy is assessed only relative to the chosen reference; <1% structural error does not measure base-level or contiguity differences beyond N50
- The comparison assemblies (Canu, MaSuRCA, 10X, DOVETAIL) are optional — conclusions about relative accuracy are only as strong as the assemblies actually included in gage_statistics_table.csv

## Evidence

- [results] Whole-genome alignment comparison shows all evaluated pipelines, including the DFS hybrid assembly, are structurally highly accurate, with less than 1% of the total 1-to-1 alignment length involved in structural errors.: [evidence span withheld — anchor status: not_found]
- [methods] Gage statistics workflow: [evidence span withheld — anchor status: not_found]
- [intro] NA12878 hybrid assembly dataset: "can produce long-range and accurate reconstructions of the genomes of Arabidopsis thaliana (Ler-0) and human (NA12878)"

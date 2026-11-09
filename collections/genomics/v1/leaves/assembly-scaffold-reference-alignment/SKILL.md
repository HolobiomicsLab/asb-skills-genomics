---
name: assembly-scaffold-reference-alignment
description: Use when assembled scaffolds or contigs (e.g., DFS hybrid NA12878 scaffolds)
  need reference-based validation of N50, alignment identity, and structural errors
  (misjoins, relocations, translocations, inversions) via whole-genome alignment and
  Gage statistics.
when_to_use_negative:
- No finished reference genome is available for the organism — Gage structural-error
  calling (misjoins, relocations, translocations, inversions) requires a reference
  to align against
- The goal is scaffolding-graph construction or synthetic library extraction from
  long reads (the Fast-SG stage upstream), not post-assembly accuracy evaluation
- Input is raw short/long reads rather than assembled scaffolds or contigs
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Nucmer
  role: Performs the whole-genome pairwise nucleotide alignment of the DFS scaffolds
    and comparison assemblies against the human reference
- name: Gage
  role: Computes assembly quality statistics (N50, identity, structural-error percentages)
    from the Nucmer alignments
- name: Fast-SG
  role: Produced the DFS hybrid NA12878 scaffolds whose structural accuracy is evaluated
    by this alignment-based validation step
  repo: https://github.com/adigenova/fast-sg
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giy048
    title: 'Fast-SG: an alignment-free algorithm for hybrid assembly'
  source_content_hash: sha256:5b5294212a1e32bba23a47b1cb33ab138acecf2132f7082ca154145d682701be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/adigenova/fast-sg
  tools:
  - Fast-SG
  - Gage
  - Nucmer
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# assembly-scaffold-reference-alignment

## Summary

Assesses the structural accuracy and continuity of assembled scaffolds — here the Fast-SG DFS hybrid assembly of human NA12878 — by whole-genome Nucmer alignment against the human reference followed by Gage statistics, benchmarked side-by-side with public Canu, MaSuRCA, 10X, and DOVETAIL assemblies.

## When to use

Use when assembled scaffolds or contigs (e.g., DFS hybrid NA12878 scaffolds) need reference-based validation of N50, alignment identity, and structural errors (misjoins, relocations, translocations, inversions) via whole-genome alignment and Gage statistics.

## When NOT to use

- No finished reference genome is available for the organism — Gage structural-error calling (misjoins, relocations, translocations, inversions) requires a reference to align against
- The goal is scaffolding-graph construction or synthetic library extraction from long reads (the Fast-SG stage upstream), not post-assembly accuracy evaluation
- Input is raw short/long reads rather than assembled scaffolds or contigs

## Inputs

- DFS hybrid assembly scaffolds of NA12878 (FASTA, produced by the Fast-SG pipeline)
- Human reference genome sequence (FASTA)
- Optional public NA12878 assemblies for comparison: Canu, MaSuRCA, 10X, and DOVETAIL (FASTA)

## Outputs

- Nucmer pairwise whole-genome alignments (delta/filtered alignment files) per assembly vs. reference
- gage_statistics_table.csv with per-assembly N50, alignment identity, and structural-error percentages (misjoins, relocations, translocations, inversions)

## How to apply

Load the DFS scaffolds produced by the Fast-SG hybrid pipeline and the human reference genome sequence, then run Nucmer whole-genome alignment of the scaffolds against the reference to obtain pairwise nucleotide alignments. Repeat the same Nucmer alignment for each optional comparison assembly (public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies) so that all pipelines are scored under identical settings. Run Gage on each alignment to compute assembly quality statistics, including N50, alignment identity, and structural-error percentages broken down as misjoins, relocations, translocations, and inversions over 1-to-1 alignments. Assemble the per-assembly statistics into a single comparison table saved as gage_statistics_table.csv. Judge success by checking that the table exists and contains the DFS row with N50, identity, and structural-error percentages; per the article's finding, all evaluated pipelines, including the DFS hybrid assembly, should show less than 1% of the total 1-to-1 alignment length involved in structural errors.

## Related tools

- **Nucmer** (Performs the whole-genome pairwise nucleotide alignment of the DFS scaffolds and comparison assemblies against the human reference)
- **Gage** (Computes assembly quality statistics (N50, identity, structural-error percentages) from the Nucmer alignments)
- **Fast-SG** (Produced the DFS hybrid NA12878 scaffolds whose structural accuracy is evaluated by this alignment-based validation step) — https://github.com/adigenova/fast-sg

## Examples

```
nucmer --prefix dfs_vs_ref human_ref.fa dfs_scaffolds.fa && delta-filter -1 dfs_vs_ref.delta > dfs_vs_ref.1delta && gage dfs_vs_ref.1delta -o gage_statistics_table.csv
```

## Evaluation signals

- gage_statistics_table.csv exists and contains, for the DFS scaffolds, the N50, identity, and structural-error percentage values required by the scope
- Summed structural errors (misjoins + relocations + translocations + inversions) account for less than 1% of the total 1-to-1 alignment length, consistent with the article's reported finding
- Comparison rows for the public Canu, MaSuRCA, 10X, and DOVETAIL NA12878 assemblies are present in the same table as the DFS scaffolds
- All assemblies were aligned against the identical human reference with the same Nucmer/Gage settings, making the per-assembly statistics directly comparable

## Limitations

- Structural-error calls are reference-dependent: they are only valid relative to the chosen human reference assembly used for the Nucmer alignment
- Gap size estimation with existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with Fast-SG synthetic libraries because linking information comes from a range of insert sizes and read-pair orientation may be unknown, which can affect the assemblies being compared
- The accuracy claim is conditional on shallow long-read coverage (5X) and the hybrid assembly setup; results will differ for other coverage levels or organisms than Arabidopsis thaliana (Ler-0) and human (NA12878)

## Evidence

- [abstract] a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions of the genomes of Arabidopsis: "a hybrid assembly approach using Fast-SG with shallow long-read coverage (5X) and moderate computational resources can produce long-range and accurate reconstructions of the genomes of Arabidopsis"
- [discussion] An estimation of the gap size with the existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with the synthetic libraries: "An estimation of the gap size with the existing long-range mate-pair technologies (10X genomics and DOVETAIL genomics) is more challenging than with the synthetic libraries"
- [methods] The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878.: "The [wiki page](https://github.com/adigenova/fast-sg/wiki/Hybrid-scaffolding-of-NA12878) provide a full example of the use of Fast-SG for the hybrid assembly of NA12878."

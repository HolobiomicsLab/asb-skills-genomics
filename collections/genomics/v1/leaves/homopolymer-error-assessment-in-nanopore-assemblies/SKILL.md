---
name: homopolymer-error-assessment-in-nanopore-assemblies
description: Use when you have multiple assembly versions of the same genome (e.g.
  uncorrected, Racon, Pilon nanopore scaffolds plus a short-read draft like AZBK01)
  and need to judge whether correction steps moved composition toward a reference
  draft and which platform biases remain.
when_to_use_negative:
- Input is raw nanopore or Illumina reads rather than assembled scaffold FASTA files
  — the comparison operates on finished assembly stages, not read sets
- The goal is locus-level error correction rather than global composition assessment
  — use an actual correction/polishing step (Racon, Pilon) instead, since correlation
  analysis does not identify which sites differ
- The assemblies come from unrelated genomes or very different k-mer conventions —
  Pearson correlation of 6-mer frequencies is only meaningful between assembly stages
  of the same genome
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
tools:
- name: Jellyfish
  role: Counts canonical 6-mer occurrences in each of the four assembly FASTA sets
    to produce k-mer count histograms
- name: pandas/scipy
  role: Normalizes 6-mer counts to relative frequencies and computes the 4x4 pairwise
    Pearson correlation matrix
- name: matplotlib/seaborn
  role: Renders the correlation matrix as a 4x4 heatmap image
- name: Racon
  role: Produced the nanopore-consensus-corrected intermediate assembly that is compared
    in the profile analysis
  repo: https://github.com/isovic/racon
- name: Pilon
  role: Produced the Illumina-polished final assembly whose 6-mer profile should approach
    the AZBK01 draft
  repo: https://github.com/broadinstitute/pilon/wiki
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-07650-6
    title: Rapid de novo assembly of the European eel genome from nanopore sequencing
      reads
  source_content_hash: sha256:d3fc2c803b0b6a19f27607a1f5dffc20a3e08c1a153d2a121df664d165c94945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/isovic/racon
  tools:
  - Jellyfish
  - Pilon
  - Racon
  - matplotlib/seaborn
  - pandas/scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# homopolymer-error-assessment-in-nanopore-assemblies

## Summary

Quantifies platform-specific sequence bias (e.g. nanopore homopolymer underrepresentation, high-CG underrepresentation in short-read drafts) by comparing canonical 6-mer frequency profiles across successive assembly stages — uncorrected TULIP, Racon-corrected, and Pilon-polished nanopore scaffolds versus the Illumina-based AZBK01 draft — with Jellyfish counts and Pearson correlations.

## When to use

Use when you have multiple assembly versions of the same genome (e.g. uncorrected, Racon, Pilon nanopore scaffolds plus a short-read draft like AZBK01) and need to judge whether correction steps moved composition toward a reference draft and which platform biases remain.

## When NOT to use

- Input is raw nanopore or Illumina reads rather than assembled scaffold FASTA files — the comparison operates on finished assembly stages, not read sets
- The goal is locus-level error correction rather than global composition assessment — use an actual correction/polishing step (Racon, Pilon) instead, since correlation analysis does not identify which sites differ
- The assemblies come from unrelated genomes or very different k-mer conventions — Pearson correlation of 6-mer frequencies is only meaningful between assembly stages of the same genome

## Inputs

- NCBI WGS AZBK01 Illumina-based draft assembly (FASTA scaffolds)
- Uncorrected TULIP scaffold FASTA files (The Uncorrected Long-read Integration Process output)
- Racon-corrected scaffold FASTA set
- Pilon-polished scaffold FASTA set

## Outputs

- fig6_kmer_correlation_matrix.csv — 4x4 pairwise Pearson correlation matrix over AZBK01, TULIP, Racon, Pilon
- fig6_kmer_correlation_heatmap.png — 4x4 heatmap including diagonal self-correlations
- Normalized canonical 6-mer relative-frequency profiles per assembly (Jellyfish histograms)

## How to apply

Download the four assembly FASTA sets: the NCBI WGS accession AZBK01 (Illumina draft), the uncorrected TULIP scaffold files, the Racon-corrected scaffolds, and the Pilon-polished scaffolds. Run Jellyfish on each assembly with k-mer length k=6 to obtain canonical 6-mer count histograms, then normalize each profile to relative frequencies so assemblies of different total sizes (e.g. the ~860 Mbp eel genome) are comparable. Compute the full 4x4 matrix of pairwise Pearson correlation coefficients (including the diagonal of self-correlations) over AZBK01, TULIP, Racon, and Pilon with pandas/scipy, and render it as a seaborn heatmap. Interpret deviations compositionally: reduced correlation for uncorrected TULIP and Racon stems from homopolymer underrepresentation in nanopore-derived sequence, while slight high-CG 6-mer underrepresentation is characteristic of the Illumina draft and Pilon sequence. The rationale is that consensus of nanopore data alone (e.g. Racon) cannot resolve all base-level errors, so Pilon polishing with Illumina data from a different specimen should pull the profile toward the short-read draft.

## Related tools

- **Jellyfish** (Counts canonical 6-mer occurrences in each of the four assembly FASTA sets to produce k-mer count histograms)
- **pandas/scipy** (Normalizes 6-mer counts to relative frequencies and computes the 4x4 pairwise Pearson correlation matrix)
- **matplotlib/seaborn** (Renders the correlation matrix as a 4x4 heatmap image)
- **Racon** (Produced the nanopore-consensus-corrected intermediate assembly that is compared in the profile analysis) — https://github.com/isovic/racon
- **Pilon** (Produced the Illumina-polished final assembly whose 6-mer profile should approach the AZBK01 draft) — https://github.com/broadinstitute/pilon/wiki

## Examples

```
jellyfish count -m 6 -C -s 500M -t 4 pilon_scaffolds.fasta -o pilon_6mers.jf
```

## Evaluation signals

- The 4x4 correlation matrix reproduces the qualitative pattern of Fig. 6: near-unity correlation between the nanopore-based assembly and the Illumina draft after Pilon correction
- The highest off-diagonal correlations are Racon-vs-Pilon and Pilon-vs-draft; the uncorrected TULIP and Racon assemblies show visibly reduced correlation to the draft
- Each normalized 6-mer frequency profile covers the canonical 6-mer space and sums to 1, so profiles are comparable across assemblies of different total sizes
- Inspecting homopolymer 6-mer bins shows underrepresentation in TULIP and Racon relative to the draft; high-CG 6-mer bins show slight underrepresentation in AZBK01 and Pilon

## Limitations

- In the absence of a high-quality reference it is not straightforward to establish which assembly is correct at apparent error sites, so 6-mer correlation is a compositional, not a truth, metric
- The eel assembly is a hybrid of two sequencing technologies, three generations of nanopore chemistry, and two different animals, so composition differences can reflect specimen/chemistry mixtures, not only correction status
- Large repetitive regions and (sub)telomeric repeats may be absent from the TULIP assembly, biasing its 6-mer profile independent of homopolymer errors

## Evidence

- [other] The correction steps bring the sequence similarity of the nanopore-based assembly closer to the Illumina-based draft, with the final corrected assembly having a high correlation to the draft: "The correction steps bring the sequence similarity of the nanopore-based assembly closer to the Illumina-based draft, with the final corrected assembly having a high correlation to the draft"
- [other] 6-mer frequencies also show homopolymer underrepresentation in TULIP/Racon and slight high-CG underrepresentation in the Illumina-based draft and Pilon sequence.: "6-mer frequencies also show homopolymer underrepresentation in TULIP/Racon and slight high-CG underrepresentation in the Illumina-based draft and Pilon sequence."
- [discussion] Racon consensus cannot resolve all nanopore base errors: "cannot be resolved by taking a consensus of nanopore data only (e.g. using Racon)"
- [discussion] Nanopore data corrected with Illumina data from a different specimen: "the nanopore data are therefore corrected using Illumina data obtained from a different eel specimen than used for nanopore sequencing"
- [discussion] No high-quality reference to determine assembly correctness: "In the absence of a high-quality reference, it is not straightforward to establish which assembly is correct"
- [readme] Racon polishing is an optional step in the ONT assembly-polish pipeline: "Polish canu contigs using [racon](https://github.com/isovic/racon) (*optional*)."

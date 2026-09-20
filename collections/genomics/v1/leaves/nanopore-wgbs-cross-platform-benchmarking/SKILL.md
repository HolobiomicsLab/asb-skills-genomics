---
name: nanopore-wgbs-cross-platform-benchmarking
description: Use when you have per-site methylation frequency files from nanopore
  tools and matched WGBS frequencies and need to test whether sequence context (k-mer
  around CpG) explains nanopore-WGBS disagreement.
when_to_use_negative:
- When no matched WGBS data exist for the nanopore dataset — the discrepancy computation
  requires per-CpG WGBS methylation frequencies
- When the goal is general accuracy benchmarking against control mixtures rather than
  sequence-context bias analysis (use mixture-based benchmarking instead)
edam_operation: http://edamontology.org/operation_3207
edam_topics:
- http://edamontology.org/topic_3295
tools:
- name: Nanopolish
  role: Produces per-site CpG methylation frequencies (augmented BED) from nanopore
    fast5s that are compared against WGBS
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Produces per-site CpG methylation predictions for the nCATS datasets used
    in the WGBS discrepancy analysis
  repo: https://github.com/nanoporetech/megalodon
- name: METEORE
  role: Consensus (combined-model) methylation predictions whose per-site output is
    included in the 8-mer discrepancy ranking
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:73b53c3fcf319925837fdc7bea10eccb69a50601148c5282536d093913c2a279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_3207
  edam_topics:
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nanopore-wgbs-cross-platform-benchmarking

## Summary

Quantifies per-CpG discrepancies between nanopore methylation predictions (Nanopolish, Megalodon, METEORE) and WGBS frequencies, then tests whether the NNNCGNNN 8-mer sequence context biases tool accuracy, focusing on AT-rich versus GC-rich contexts.

## When to use

Use when you have per-site methylation frequency files from nanopore tools and matched WGBS frequencies and need to test whether sequence context (k-mer around CpG) explains nanopore-WGBS disagreement.

## When NOT to use

- When no matched WGBS data exist for the nanopore dataset — the discrepancy computation requires per-CpG WGBS methylation frequencies
- When the goal is general accuracy benchmarking against control mixtures rather than sequence-context bias analysis (use mixture-based benchmarking instead)

## Inputs

- Per-site methylation frequency files (augmented BED: chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish
- Per-site methylation frequency files from Megalodon
- Per-site methylation frequencies from METEORE combined-model output
- Matched WGBS per-CpG methylation frequencies
- Reference genome FASTA for extracting NNNCGNNN 8-mer contexts

## Outputs

- Ranked table of 8-mer contexts ordered by mean nanopore-WGBS discrepancy per tool
- Per-CpG absolute discrepancy values per tool
- pLogo motif comparisons of flanking sequences from low- versus high-discrepancy CpG sets per GC-content group

## How to apply

Load per-site methylation frequencies for the nCATS datasets in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish, Megalodon, and the METEORE combined-model output. Match each nanopore-predicted CpG to its WGBS methylation frequency and compute the absolute difference per tool per CpG. Extract the 8-mer context (NNNCGNNN) centered on each CpG from the reference genome and assign GC-content groups; group discrepancies by k-mer, compute mean discrepancy per k-mer, and rank k-mers highest to lowest per tool. Within each GC-content group, partition CpGs into low- and high-discrepancy sets and build pLogo motif comparisons of flanking sequences. Inspect the ranked k-mer table and pLogos: the acceptance criterion is qualitative — pLogos must show enriched AT-rich flanking contexts among high-discrepancy k-mers, with no significant motif enrichment for Nanopolish, Megalodon, or METEORE.

## Related tools

- **Nanopolish** (Produces per-site CpG methylation frequencies (augmented BED) from nanopore fast5s that are compared against WGBS) — https://github.com/comprna/METEORE
- **Megalodon** (Produces per-site CpG methylation predictions for the nCATS datasets used in the WGBS discrepancy analysis) — https://github.com/nanoporetech/megalodon
- **METEORE** (Consensus (combined-model) methylation predictions whose per-site output is included in the 8-mer discrepancy ranking) — https://github.com/comprna/METEORE

## Evaluation signals

- Ranked 8-mer table shows the largest mean nanopore-WGBS discrepancies in AT-rich contexts
- pLogo comparisons show AT-rich flanking sequences enriched in the high-discrepancy CpG sets
- pLogos show no significant motif enrichment for Nanopolish, Megalodon, and METEORE
- Per-site absolute discrepancies are computable for every CpG matched between nanopore output and WGBS
- Augmented BED inputs conform to the 6-column schema (chromosome, start, end, coverage, methylation frequency, strand)

## Limitations

- Discrepancies with WGBS could stem from detection differences between platforms or from disparate model types and training regimes of the tools, so sequence-context bias cannot be attributed to a single cause
- The acceptance criterion is qualitative (pLogo enrichment patterns), not a quantitative threshold

## Evidence

- [supplementary] ranking 8-mers by average discrepancy: [evidence span withheld — anchor status: not_found]
- [readme] augmented BED format: "METEORE can now produce two per-site result files in an **augmented BED format** for each tool except for DeepMod"
- [discussion] WGBS discrepancies discussion: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested."
- [discussion] nCATS WGBS comparison: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"

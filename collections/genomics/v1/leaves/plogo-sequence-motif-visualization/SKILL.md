---
name: plogo-sequence-motif-visualization
description: Use when comparing per-site methylation discrepancies between nanopore
  tools and WGBS and you need to test whether sequence context (e.g. AT-rich NNNCGNNN
  8-mers) drives tool-specific error.
when_to_use_negative:
- Input is single-read (per-read) methylation scores rather than aggregated per-site
  methylation frequencies, since the analysis groups discrepancies per CpG site
- The goal is methylation calling itself — this skill only visualizes and tests sequence-context
  bias in already-produced per-site calls
- The methylation system is not CpG (e.g. dam/dcm GATC contexts), since the 8-mer
  extraction is centered on NNNCGNNN
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: Nanopolish
  role: Source of per-site methylation frequencies (augmented BED) compared against
    WGBS for context-bias analysis
- name: Megalodon
  role: Source of per-site methylation frequencies (augmented BED) compared against
    WGBS for context-bias analysis
  repo: https://github.com/nanoporetech/megalodon
- name: METEORE
  role: Provides the consensus combined-model per-site output (augmented BED) included
    in the discrepancy and pLogo comparison
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:c49f55a8bc813ca27f07c7d85cda6b5e50e7bb9e2b04ac636db4550dec921ab6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nanoporetech/megalodon
  tools:
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# pLogo sequence motif visualization

## Summary

A skill for visualizing sequence-context bias in nanopore CpG methylation calls by comparing flanking-sequence motifs (pLogos) between low- and high-discrepancy CpG sites and ranking NNNCGNNN 8-mers by mean nanopore-vs-WGBS discrepancy per tool.

## When to use

Use when comparing per-site methylation discrepancies between nanopore tools and WGBS and you need to test whether sequence context (e.g. AT-rich NNNCGNNN 8-mers) drives tool-specific error.

## When NOT to use

- Input is single-read (per-read) methylation scores rather than aggregated per-site methylation frequencies, since the analysis groups discrepancies per CpG site
- The goal is methylation calling itself — this skill only visualizes and tests sequence-context bias in already-produced per-site calls
- The methylation system is not CpG (e.g. dam/dcm GATC contexts), since the 8-mer extraction is centered on NNNCGNNN

## Inputs

- Per-site methylation frequency files (augmented BED: chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish
- Per-site methylation frequency files from Megalodon
- Per-site methylation frequency files from the METEORE combined-model output
- WGBS per-site methylation frequencies for the matching nCATS samples
- Reference genome FASTA for extracting NNNCGNNN 8-mer contexts

## Outputs

- Table of NNNCGNNN 8-mers ranked by mean nanopore-WGBS absolute discrepancy per tool
- Per-CpG absolute discrepancy values per tool annotated with 8-mer context and GC-content group
- pLogo motif comparisons of flanking sequences of low- versus high-discrepancy CpG sets per GC-content group and tool

## How to apply

Load per-site methylation frequency files in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish, Megalodon, and the METEORE combined-model output for the nCATS datasets, and match each predicted CpG to its WGBS methylation frequency. Compute the absolute discrepancy per tool for every CpG, then extract the 8-mer context (NNNCGNNN) centered on each CpG from the reference genome and assign each site to a GC-content group. Group discrepancies by 8-mer, compute the mean discrepancy per k-mer per tool, and rank k-mers from highest to lowest mean discrepancy. Within each GC-content group, partition CpGs into low- and high-discrepancy sets and generate pLogo motif comparisons of their flanking sequences. The rationale is that enriched AT-rich flanking contexts among high-discrepancy k-mers indicate sequence-context bias, whereas no significant motif enrichment for Nanopolish, Megalodon, and METEORE indicates those tools are robust to context.

## Related tools

- **Nanopolish** (Source of per-site methylation frequencies (augmented BED) compared against WGBS for context-bias analysis)
- **Megalodon** (Source of per-site methylation frequencies (augmented BED) compared against WGBS for context-bias analysis) — https://github.com/nanoporetech/megalodon
- **METEORE** (Provides the consensus combined-model per-site output (augmented BED) included in the discrepancy and pLogo comparison) — https://github.com/comprna/METEORE

## Evaluation signals

- The pLogo comparisons must show enriched AT-rich flanking contexts among high-discrepancy k-mers, consistent with the paper's WGBS comparison finding
- Nanopolish, Megalodon, and METEORE must show no significant sequence-context motif enrichment in the pLogo comparisons
- Each CpG site must have a valid NNNCGNNN 8-mer extracted from the reference genome and an assigned GC-content group before ranking
- The ranked 8-mer table must contain one mean-discrepancy ranking per tool (Nanopolish, Megalodon, METEORE) computed only over matched nanopore-WGBS CpG sites

## Limitations

- The finding is qualitative: validation relies on visual inspection of pLogos rather than a numeric significance threshold
- Discrepancies with WGBS could stem from detection differences or disparate model types and training regimes of the tools, not necessarily sequence-context bias
- The augmented BED per-site output is not yet produced for DeepMod in METEORE, so DeepMod cannot be included in this per-site context analysis

## Evidence

- [supplementary] Ranking 8-mers by average discrepancy across methods shows that nanopore methylation calls and WGBS disagree the most in AT-rich sequence contexts.: [evidence span withheld — anchor status: not_found]
- [discussion] Discrepancies with WGBS could stem from detection differences or disparate model types and training regimes: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested."
- [readme] METEORE augmented BED per-site output format: "In the second output file, we combine the methylation predictions from both strands on CpG sites by averaging the methylation frequencies and adding up the coverage."

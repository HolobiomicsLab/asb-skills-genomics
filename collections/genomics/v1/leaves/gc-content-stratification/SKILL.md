---
name: gc-content-stratification
description: Use when per-site methylation frequencies from nanopore tools and a WGBS
  reference are available and you need to test whether sequence context (k-mer composition,
  GC content) explains tool-WGBS disagreement.
when_to_use_negative:
- You only have per-read methylation scores without reference-anchored per-site frequencies
  — this analysis needs per-site frequency tables matched to WGBS
- No WGBS ground-truth methylation frequencies are available for the same CpG sites
- Your question is about overall tool accuracy (false positives/false negatives tradeoff)
  rather than sequence-context bias
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0160
- http://edamontology.org/topic_0622
tools:
- name: Nanopolish
  role: Provides per-site methylation frequency calls (augmented BED) on nCATS data
    whose discrepancies against WGBS are stratified by 8-mer context
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Provides per-site methylation frequency calls whose discrepancies against
    WGBS are ranked by k-mer context
  repo: https://github.com/nanoporetech/megalodon
- name: METEORE
  role: Supplies combined-model consensus per-site methylation predictions included
    in the k-mer discrepancy ranking
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:a14e328f2ecd2f649be905c8dee4145e70348ee0614dc87d2ceeca4de5a01fdc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0160
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# GC-content stratification of methylation-calling discrepancies

## Summary

Quantifies and stratifies per-site discrepancies between nanopore methylation calls (Nanopolish, Megalodon, METEORE) and WGBS methylation frequencies by CpG 8-mer sequence context (NNNCGNNN) and GC-content group, to reveal whether AT-rich contexts bias nanopore methylation detection.

## When to use

Use when per-site methylation frequencies from nanopore tools and a WGBS reference are available and you need to test whether sequence context (k-mer composition, GC content) explains tool-WGBS disagreement.

## When NOT to use

- You only have per-read methylation scores without reference-anchored per-site frequencies — this analysis needs per-site frequency tables matched to WGBS
- No WGBS ground-truth methylation frequencies are available for the same CpG sites
- Your question is about overall tool accuracy (false positives/false negatives tradeoff) rather than sequence-context bias

## Inputs

- Per-site methylation frequency files (augmented BED: chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish on nCATS datasets
- Per-site methylation frequency files from Megalodon on nCATS datasets
- METEORE combined-model per-site methylation output
- Matched WGBS methylation frequencies for the same CpG sites
- Reference genome FASTA for extracting NNNCGNNN 8-mer contexts

## Outputs

- Ranked table of mean per-k-mer discrepancy (highest to lowest) for each tool
- Per-CpG absolute discrepancy values stratified by GC-content group
- pLogo motif comparisons of flanking sequences between low- and high-discrepancy CpG sets

## How to apply

Load per-site methylation frequency files in the augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) for Nanopolish, Megalodon, and the METEORE combined-model output on the nCATS datasets. Match each nanopore-predicted CpG to its WGBS methylation frequency and compute the absolute difference per tool for every CpG. Extract the 8-mer context (NNNCGNNN) centered on each CpG from the reference genome and assign each site to a GC-content group. Group per-site discrepancies by 8-mer, compute mean discrepancy per k-mer, and rank k-mers highest to lowest per tool; additionally partition sites into low- and high-discrepancy sets within each GC-content group and build pLogo motif comparisons of their flanking sequences. The rationale is that systematic tool-WGBS bias tied to sequence context would appear as enriched AT-rich flanks among high-discrepancy k-mers, whereas unbiased tools show no significant motif enrichment.

## Related tools

- **Nanopolish** (Provides per-site methylation frequency calls (augmented BED) on nCATS data whose discrepancies against WGBS are stratified by 8-mer context) — https://github.com/comprna/METEORE
- **Megalodon** (Provides per-site methylation frequency calls whose discrepancies against WGBS are ranked by k-mer context) — https://github.com/nanoporetech/megalodon
- **METEORE** (Supplies combined-model consensus per-site methylation predictions included in the k-mer discrepancy ranking) — https://github.com/comprna/METEORE

## Evaluation signals

- The pLogo comparisons show enriched AT-rich flanking contexts among high-discrepancy CpG sets
- Nanopolish, Megalodon, and METEORE show no significant sequence-context motif enrichment in the pLogos
- Ranking 8-mers by mean discrepancy shows the largest nanopore-WGBS disagreement concentrated in AT-rich contexts
- Every CpG analyzed has a matched WGBS frequency and a computable NNNCGNNN 8-mer from the reference genome

## Limitations

- The acceptance criterion is qualitative — enriched AT-rich contexts among high-discrepancy k-mers — rather than a numeric threshold
- Discrepancies with WGBS could stem from detection differences between platforms or from disparate model types and training regimes of the tools, not necessarily sequence bias
- WGBS comparison datasets lacked accession identifiers in the discussion, complicating exact replication

## Evidence

- [readme] Augmented BED format fields: "The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome 3. End position in chromosome 4. Read coverage 5. Methylation (i.e. methylation frequency)"
- [discussion] WGBS comparison discussion: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested."
- [methods] METEORE clone command: "git clone https://github.com/comprna/METEORE.git"

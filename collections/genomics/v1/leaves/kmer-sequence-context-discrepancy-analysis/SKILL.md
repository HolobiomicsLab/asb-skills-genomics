---
name: kmer-sequence-context-discrepancy-analysis
description: Use when comparing per-site methylation frequencies from nanopore tools
  against a orthogonal reference (e.g. WGBS) and you need to test whether flanking
  sequence context explains discrepancies at CpG sites.
when_to_use_negative:
- Input data are single-read per-read scores (fast5-level) rather than per-site aggregated
  methylation frequency files — the discrepancy analysis requires site-level frequencies
  in augmented BED format
- The orthogonal reference is another nanopore tool rather than WGBS; this skill specifically
  benchmarks nanopore against bisulfite-derived methylation frequencies
- The organism's methylation context is non-CpG (e.g. bacterial dam/dcm); the NNNCGNNN
  8-mer extraction is defined on CpG dinucleotides
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Nanopolish
  role: Source of per-site methylation frequency calls (augmented BED) for the nanopore
    arm of the comparison
  repo: https://github.com/comprna/METEORE
- name: Megalodon
  role: Source of per-site methylation frequency calls (augmented BED) for the nanopore
    arm of the comparison
  repo: https://github.com/nanoporetech/megalodon
- name: METEORE
  role: Consensus combined-model per-site methylation predictions included as a third
    tool in the discrepancy ranking
  repo: https://github.com/comprna/METEORE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23778-6
    title: Systematic benchmarking of tools for CpG methylation detection from nanopore
      sequencing
  source_content_hash: sha256:9e6c0a2cf86bd8125f5c494b4b3bf8743c7eafa38487ef982040139627d1bb90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/METEORE
  tools:
  - METEORE
  - Megalodon
  - Nanopolish
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# k-mer sequence-context discrepancy analysis

## Summary

A motif-analysis skill that quantifies whether the 8-mer sequence context (NNNCGNNN) centered on CpG sites biases agreement between nanopore methylation calls (Nanopolish, Megalodon, METEORE combined model) and WGBS methylation frequencies in nCATS datasets. It produces ranked k-mer discrepancy tables and low-vs-high discrepancy pLogo motif comparisons to expose AT-rich contexts where nanopore and WGBS disagree most.

## When to use

Use when comparing per-site methylation frequencies from nanopore tools against a orthogonal reference (e.g. WGBS) and you need to test whether flanking sequence context explains discrepancies at CpG sites.

## When NOT to use

- Input data are single-read per-read scores (fast5-level) rather than per-site aggregated methylation frequency files — the discrepancy analysis requires site-level frequencies in augmented BED format
- The orthogonal reference is another nanopore tool rather than WGBS; this skill specifically benchmarks nanopore against bisulfite-derived methylation frequencies
- The organism's methylation context is non-CpG (e.g. bacterial dam/dcm); the NNNCGNNN 8-mer extraction is defined on CpG dinucleotides

## Inputs

- Augmented BED per-site methylation frequency files from Nanopolish (chromosome, start, end, coverage, methylation frequency, strand)
- Augmented BED per-site methylation frequency files from Megalodon
- METEORE combined-model (random forest / linear regression) per-site output file
- WGBS methylation frequency calls matched to the same CpG sites
- Reference genome FASTA for extracting NNNCGNNN 8-mer contexts

## Outputs

- Ranked table of mean per-8-mer discrepancy (highest to lowest) per tool
- Low- and high-discrepancy CpG site sets partitioned within each GC-content group
- pLogo motif comparisons of flanking sequences for low- vs high-discrepancy sets
- Qualitative conclusion on which sequence contexts and tools show largest nanopore-WGBS disagreement

## How to apply

Load per-site methylation frequency files in augmented BED format (chromosome, start, end, coverage, methylation frequency, strand) from Nanopolish, Megalodon, and the METEORE combined-model output for the nCATS datasets. Match each nanopore-predicted CpG to its WGBS methylation frequency and compute the absolute difference per tool for every CpG. Extract the 8-mer context (NNNCGNNN) centered on each CpG from the reference genome, assign each site a GC-content group, then group per-site discrepancies by 8-mer, compute mean discrepancy per k-mer, and rank k-mers highest to lowest per tool. Partition CpG sites into low- and high-discrepancy sets within each GC-content group and generate pLogo motif comparisons of the flanking sequences to identify enriched contexts. The rationale is that consensus/individual tool predictions show high dispersion against expected methylation frequencies, and the WGBS comparison in this study found the largest disagreement in AT-rich contexts.

## Related tools

- **Nanopolish** (Source of per-site methylation frequency calls (augmented BED) for the nanopore arm of the comparison) — https://github.com/comprna/METEORE
- **Megalodon** (Source of per-site methylation frequency calls (augmented BED) for the nanopore arm of the comparison) — https://github.com/nanoporetech/megalodon
- **METEORE** (Consensus combined-model per-site methylation predictions included as a third tool in the discrepancy ranking) — https://github.com/comprna/METEORE

## Examples

```
python -c "import pandas as pd; d=pd.read_csv('megalodon_freq.bed',sep='\t',names=['chr','start','end','cov','freq','strand']); d['discrepancy']=(d['freq']-wgbs['freq']).abs(); print(d.groupby('kmer')['discrepancy'].mean().sort_values(ascending=False))"
```

## Evaluation signals

- Each matched CpG site has an absolute discrepancy value computed for every tool (Nanopolish, Megalodon, METEORE) — no tool left unpaired against WGBS
- Every ranked 8-mer in the table has exactly 8 bases matching the NNNCGNNN pattern (C at positions 4-5)
- pLogo comparisons show enriched AT-rich flanking contexts among high-discrepancy k-mers, consistent with the paper's reported WGBS comparison finding
- Nanopolish, Megalodon, and METEORE show no significant sequence-context motif enrichment in the pLogo comparison
- Ranking is computed within each GC-content group and per tool, not pooled across tools

## Limitations

- Discrepancies with WGBS could stem from detection differences between the platforms or from disparate model types and training regimes of the tools, not from sequence-context bias alone
- The acceptance criterion is qualitative (pLogo enrichment of AT-rich contexts), not a quantitative threshold, so replication depends on motif-visualization judgment
- Part of the low agreement with expected methylation frequency may reflect that the M.SssI methylated control only has 95% efficiency

## Evidence

- [discussion] the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency: "the predictions per site generally showed a high dispersion and did not generally agree with the expected methylation frequency"
- [discussion] WGBS discrepancies attribution: "Although these discrepancies could be due to the detection differences between nanopore and WGBS, they could also point to the disparate types of models and training regimes used by the tools tested."
- [discussion] nCATS WGBS comparison consistency: "The comparison with WGBS datasets using two independent nCATS experiments recovered results consistent with the analyses"
- [readme] augmented BED output format: "The first output file contains the following fields: 1. Reference chromosome 2. Start position in chromosome 3. End position in chromosome 4. Read coverage 5. Methylation (i.e. methylation frequency)"
- [other] AT-rich discrepancy finding: [evidence span withheld — anchor status: not_found]

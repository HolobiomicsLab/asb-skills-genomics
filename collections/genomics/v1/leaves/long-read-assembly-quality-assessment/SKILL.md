---
name: long-read-assembly-quality-assessment
description: Use when comparing mis-scaffolding between two long-read assemblies (e.g.,
  HiFiCCL-Hifiasm vs Hifiasm from ultra-low-coverage HG002/NA19240 HiFi reads) scaffolded
  to chromosomes with RagTag, where inter-chromosomal synteny links >100 kb quantify
  mis-scaffolding.
when_to_use_negative:
- 'When the goal is reference-corrected scaffolding: RagTag''s correct function is
  deliberately NOT used here because it would overwrite true SVs'
- When evaluating contig-level misassembly length at ~5x coverage (the 21.19% average
  reduction metric) rather than post-scaffolding inter-chromosomal mis-scaffolding
- When assemblies were already scaffolded with orthogonal Hi-C/Bionano data — the
  article notes such integration is not yet implemented in this workflow
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: RagTag
  role: Scaffolds contig assemblies to chromosome level against CHM13 (v2.0); the
    correct function is deliberately not used so true SVs are not overwritten
- name: NGenomeSyn
  role: Synteny analysis of scaffolded assemblies vs reference; inter-chromosomal
    links >100 kb are counted as mis-scaffolding events
- name: minimap2
  role: Aligns scaffolded assemblies to the reference genome to derive chromosome-level
    placement of scaffolds
  repo: https://github.com/lh3/minimap2
- name: hifiasm
  role: Produces the contig assemblies (with and without HiFiCCL pre-clustering) that
    are scaffolded and compared
  repo: https://github.com/chhylp123/hifiasm
- name: HiFiCCL
  role: Reference-guided chromosome-by-chromosome read binning framework whose output
    assembly is the test case for reduced mis-scaffolding
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: Quast
  role: Continuity metrics compiled alongside mis-link counts in the comparison table
- name: BUSCO
  role: Completeness metrics compiled alongside mis-link counts in the comparison
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:0f420681e606f7616db9ded67a1b23255bd7d8760633e701e4b7cd427ceb1e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - BUSCO
  - HiFiCCL
  - NGenomeSyn
  - Quast
  - RagTag
  - hifiasm
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# long-read-assembly-quality-assessment

## Summary

Assesses chromosome-level scaffolding quality of low-coverage HiFi assemblies by scaffolding contigs with RagTag against CHM13 (v2.0) and quantifying inter-chromosomal links >100,000 bp in NGenomeSyn synteny analysis. Demonstrates mis-scaffolding reduction (e.g., HG002: 14 vs 189 links = 92.59% reduction for HiFiCCL-Hifiasm vs Hifiasm).

## When to use

Use when comparing mis-scaffolding between two long-read assemblies (e.g., HiFiCCL-Hifiasm vs Hifiasm from ultra-low-coverage HG002/NA19240 HiFi reads) scaffolded to chromosomes with RagTag, where inter-chromosomal synteny links >100 kb quantify mis-scaffolding.

## When NOT to use

- When the goal is reference-corrected scaffolding: RagTag's correct function is deliberately NOT used here because it would overwrite true SVs
- When evaluating contig-level misassembly length at ~5x coverage (the 21.19% average reduction metric) rather than post-scaffolding inter-chromosomal mis-scaffolding
- When assemblies were already scaffolded with orthogonal Hi-C/Bionano data — the article notes such integration is not yet implemented in this workflow

## Inputs

- HiFiCCL-Hifiasm and Hifiasm contig-level assemblies (FASTA) from ultra-low-coverage (~5x) HiFi datasets such as HG002 (SRR10382244) and NA19240 (SRR14611231)
- CHM13 (v2.0) reference genome sequence for RagTag scaffolding
- Sample-specific reference genome (e.g., HG002 maternal reference) for NGenomeSyn synteny analysis

## Outputs

- Chromosome-level scaffolded assemblies for each assembly method
- NGenomeSyn synteny plots with chromosome-to-chromosome links
- Counts of inter-chromosomal links >100,000 bp per sample and per assembly method
- scaffolding_mislinks_comparison.tsv with link counts, percentage reductions, and continuity metrics (QUAST/BUSCO)

## How to apply

Obtain the HiFiCCL-Hifiasm and Hifiasm contig assemblies (e.g., from Zenodo DOI: 10.5281/zenodo.17204429 for HG002 and NA19240 ultra-low-coverage datasets) and scaffold each with RagTag against CHM13 (v2.0), deliberately omitting RagTag's correct step so true structural variants are not overwritten by the reference. Align the scaffolded assemblies to the corresponding reference genome to derive chromosome-level placement of scaffolds. Run synteny analysis with NGenomeSyn and count inter-chromosomal links larger than 100 kb, which represent inter-chromosomal mis-scaffolding events. Compare link counts per sample (HG002: 14 vs 189 links = 92.59% reduction; HG00438: 95.51%/96.77% reduction) and compile counts, reductions, and continuity metrics (QUAST, BUSCO) into scaffolding_mislinks_comparison.tsv.

## Related tools

- **RagTag** (Scaffolds contig assemblies to chromosome level against CHM13 (v2.0); the correct function is deliberately not used so true SVs are not overwritten)
- **NGenomeSyn** (Synteny analysis of scaffolded assemblies vs reference; inter-chromosomal links >100 kb are counted as mis-scaffolding events)
- **minimap2** (Aligns scaffolded assemblies to the reference genome to derive chromosome-level placement of scaffolds) — https://github.com/lh3/minimap2
- **hifiasm** (Produces the contig assemblies (with and without HiFiCCL pre-clustering) that are scaffolded and compared) — https://github.com/chhylp123/hifiasm
- **HiFiCCL** (Reference-guided chromosome-by-chromosome read binning framework whose output assembly is the test case for reduced mis-scaffolding) — https://github.com/zjjbuqi/HiFiCCL
- **Quast** (Continuity metrics compiled alongside mis-link counts in the comparison table)
- **BUSCO** (Completeness metrics compiled alongside mis-link counts in the comparison table)

## Examples

```
minimap2 -cx asm5 CHM13v2.fa HiFiCCL_hifiasm_scaffolds.fa > HiFiCCL_hifiasm_vs_CHM13.paf
```

## Evaluation signals

- For HG002, HiFiCCL-Hifiasm must show 14 inter-chromosomal links >100 kb vs 189 for Hifiasm, matching the reported 92.59% reduction within rounding tolerance
- HiFiCCL-Hifiasm must show fewer inter-chromosomal links than Hifiasm for every sample tested (e.g., HG00438 with 95.51%/96.77% reduction)
- Verify the RagTag pipeline ran without the correct step, since running it would overwrite true SVs and bias the synteny comparison
- scaffolding_mislinks_comparison.tsv exists and contains per-sample link counts, percentage reductions, and continuity metrics
- All synteny-derived links counted as mis-scaffolding events exceed the 100,000 bp threshold

## Limitations

- RagTag scaffolding relies on reference genome alignments, which may overlook true SVs (only 62.5% of inversion SVs were detected in the article's analysis)
- No version numbers were stated for HiFiCCL, Hifiasm, or RagTag, so exact reproduction may vary with tool versions
- At ~5x coverage many genomic regions remain missing, so completeness metrics are limited and mis-link counts do not capture all assembly errors

## Evidence

- [discussion] To mitigate this effect, we chose not to use RagTag's correct function: "To mitigate this effect, we chose not to use RagTag's correct function"
- [abstract] These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [methods] available on Zenodo and can be accessed using the [DOI: 10.5281/zenodo.17204429]: "available on Zenodo and can be accessed using the [DOI: 10.5281/zenodo.17204429]"
- [discussion] the scaffolding process relies on reference genome alignments, which may overlook true SVs: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"
- [readme] README minimap2 asm-to-asm alignment preset: [evidence span withheld — anchor status: external_doc]

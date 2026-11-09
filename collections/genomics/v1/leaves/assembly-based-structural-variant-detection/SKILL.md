---
name: assembly-based-structural-variant-detection
description: Use when the goal is to detect and benchmark large germline structural
  variants (e.g. >7000 bp) from de novo assembly FASTAs of low-coverage HiFi reads,
  against truth sets such as GIAB Tier 1 or T2T-HG002-Q100.
when_to_use_negative:
- When reads are available at high coverage (e.g. HG00438 requiring 41x HiFi) — standard
  read-level SV calling on full-depth data is the appropriate route instead of low-coverage
  assembly-based detection
- When the target variants are small (<7000 bp) — this workflow filters explicitly
  to SVs larger than 7000 bp and the benchmark thresholds are tuned for large SVs
- When a reference genome is unavailable — HiFiCCL relies on reference-guided chromosome-by-chromosome
  binning of reads before assembly
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: SVIM-asm
  role: Calls structural variants from assembly-to-reference alignments of each assembly
    FASTA (v1.0.3)
- name: Truvari
  role: Benchmarks filtered SV calls (precision, recall, F1) against GIAB Tier 1 and
    T2T-HG002-Q100 truth sets (v3.2.0)
- name: HiFiCCL
  role: Produces reference-guided chromosome-by-chromosome assemblies at ~5x coverage
    whose primary+alternate contigs are the SV-calling inputs
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Base assembler combined with HiFiCCL to generate primary and alternate assemblies
    (~5x datasets)
  repo: https://github.com/chhylp123/hifiasm
- name: minimap2
  role: Assembler-to-reference alignment with the asm5 preset underlies assembly-space
    SV calling
  repo: https://github.com/lh3/minimap2
- name: LJA
  role: HiFi de Bruijn-graph assembler used for the ~8x dataset assembly variants
  repo: https://github.com/AntonBankevich/LJA
- name: Flye / HiFlye
  role: Base assembler whose HiFi mode provides the ~8x dataset assembly variants
  repo: https://github.com/mikolmogorov/Flye
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:aed71c6b5c53f2856d3920e07fdc0e562d06f8704c9d4d7d8985fbeabd277773
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - Flye / HiFlye
  - HiFiCCL
  - LJA
  - SVIM-asm
  - Truvari
  - hifiasm
  - minimap2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# assembly-based-structural-variant-detection

## Summary

Detects large structural variants from de novo genome assemblies by aligning contigs to a reference and calling SVs in assembly-to-reference alignment space, then benchmarking calls against truth sets. Used here to show HiFiCCL-combined assemblies improve detection of SVs >7000 bp on low-coverage (~5x–8x) HiFi data.

## When to use

Use when the goal is to detect and benchmark large germline structural variants (e.g. >7000 bp) from de novo assembly FASTAs of low-coverage HiFi reads, against truth sets such as GIAB Tier 1 or T2T-HG002-Q100.

## When NOT to use

- When reads are available at high coverage (e.g. HG00438 requiring 41x HiFi) — standard read-level SV calling on full-depth data is the appropriate route instead of low-coverage assembly-based detection
- When the target variants are small (<7000 bp) — this workflow filters explicitly to SVs larger than 7000 bp and the benchmark thresholds are tuned for large SVs
- When a reference genome is unavailable — HiFiCCL relies on reference-guided chromosome-by-chromosome binning of reads before assembly

## Inputs

- Primary and alternate assembly FASTA files from HiFiCCL-Hifiasm or Hifiasm (~5x HG002) and HiFlye/LJA variant assemblies (~8x), from Zenodo DOI 10.5281/zenodo.17204429
- GIAB Tier 1 structural variant truth set
- T2T-HG002-Q100 SV benchmark
- Reference genome GRCh37 (GRCh38 also used elsewhere in the study)

## Outputs

- Structural variant calls per assembly in VCF (SVIM-asm output)
- Filtered SV call set restricted to variants larger than 7000 bp
- Precision, recall, and F1 metrics per benchmark (GIAB Tier 1, T2T-HG002-Q100), written to svim_asm_sv_benchmark_metrics.tsv
- Insertion F1 stratified by size bins (10–15 kb and 15–20 kb)

## How to apply

Obtain primary and alternate assemblies from the manuscript's Zenodo deposit (DOI: 10.5281/zenodo.17204429) — HiFiCCL-Hifiasm and Hifiasm assemblies for the ~5x HG002 data, HiFlye and LJA variant assemblies for the ~8x data. Run SVIM-asm (v1.0.3) on each assembly FASTA to call structural variants in assembly-to-reference alignment space, then filter calls to SVs larger than 7000 bp. Compare filtered calls against the GIAB Tier 1 truth set and the T2T-HG002-Q100 benchmark using Truvari (v3.2.0) with GRCh37 as the reference, computing precision, recall, and F1 for the primary plus alternate assemblies. Additionally stratify insertion calls by size (10–15 kb and 15–20 kb bins) and compute per-bin F1. The rationale is that reference-guided chromosome-by-chromosome assembly (HiFiCCL) reconstructs new SVs that scaffolding approaches like RagTag may overlook, so assembly-level calling recovers large germline SVs better than read-level callers alone.

## Related tools

- **SVIM-asm** (Calls structural variants from assembly-to-reference alignments of each assembly FASTA (v1.0.3))
- **Truvari** (Benchmarks filtered SV calls (precision, recall, F1) against GIAB Tier 1 and T2T-HG002-Q100 truth sets (v3.2.0))
- **HiFiCCL** (Produces reference-guided chromosome-by-chromosome assemblies at ~5x coverage whose primary+alternate contigs are the SV-calling inputs) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Base assembler combined with HiFiCCL to generate primary and alternate assemblies (~5x datasets)) — https://github.com/chhylp123/hifiasm
- **minimap2** (Assembler-to-reference alignment with the asm5 preset underlies assembly-space SV calling) — https://github.com/lh3/minimap2
- **LJA** (HiFi de Bruijn-graph assembler used for the ~8x dataset assembly variants) — https://github.com/AntonBankevich/LJA
- **Flye / HiFlye** (Base assembler whose HiFi mode provides the ~8x dataset assembly variants) — https://github.com/mikolmogorov/Flye

## Examples

```
svim-asm svim_asm_out hificcl_hifiasm.asm.paf hificcl_hifiasm.primary.fa
```

## Evaluation signals

- On GIAB Tier 1, SVIM-asm on HiFiCCL-Hifiasm assemblies should reach precision 0.9193, recall 0.5643, and F1 0.6993 for SVs >7000 bp — F1 should be the highest among compared assemblies
- Against T2T-HG002-Q100, recall should reach 0.4369 (vs Sniffles2's 0.3526) with precision 0.9255 and F1 0.6165 when combining primary+alternate assemblies
- Insertion F1 in the 10–15 kb bin should be ~0.5851 and in the 15–20 kb bin ~0.6038
- Truvari bench must be run against GRCh37 coordinates with truth sets in matching reference space — mismatched reference builds will collapse recall
- All precision/recall/F1 values land in svim_asm_sv_benchmark_metrics.tsv with one row per assembly-vs-truth-set comparison

## Limitations

- RagTag-based scaffolding relies on reference genome alignments and may overlook true SVs — only 62.5% of inversion SVs were detected via scaffolding
- At ~5x coverage, many genomic regions remain missing, so assembly-based SV recall is bounded by assembly completeness gaps
- The paper does not state version numbers for HiFiCCL, Hifiasm, or RagTag, complicating exact reproduction
- Large germline SV detection was validated for SVs larger than 10,000 bp in the framework's applications; the >7000 bp filter is the reproduced benchmark threshold

## Evidence

- [abstract] abstract: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [intro] intro: "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp)"
- [readme] readme: [evidence span withheld — anchor status: external_doc]
- [discussion] discussion: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"

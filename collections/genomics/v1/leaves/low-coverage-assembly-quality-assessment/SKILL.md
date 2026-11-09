---
name: low-coverage-assembly-quality-assessment
description: Use when evaluating whether assemblies from ~5x–8x HiFi reads (e.g. HiFiCCL-Hifiasm
  primary+alternate, HiFlye, LJA) improve detection of large structural variants (>7000
  bp) against GIAB Tier 1 or T2T-HG002-Q100 truth sets.
when_to_use_negative:
- Input is high-coverage (>25x) HiFi data — HiFiCCL and this assessment workflow are
  designed for low-coverage (~5x–8x) scenarios where de novo assembly alone is challenged
- Input reads are noisy ONT/CLR reads — the framework targets high-fidelity (HiFi)
  reads, and the compared assemblers (Hifiasm, HiFlye, LJA) are HiFi-based
- The goal is read-based SV calling (e.g. Sniffles2-style on BAM/CRAM) rather than
  assembly-based calling — the article uses Sniffles2 only as a recall comparator
  (0.3526) for the assembly-based SVIM-asm approach
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: HiFiCCL
  role: Reference-guided chromosome-by-chromosome assembly framework producing the
    low-coverage primary/alternate assemblies under evaluation
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Base assembler run on HiFiCCL chromosome bins and on unclustered reads (~5x)
    to generate primary and alternate assemblies
  repo: https://github.com/chhylp123/hifiasm
- name: minimap2
  role: Aligns assembly contigs to the reference genome (asm5 preset) to support assembly-to-reference
    SV calling and misassembly assessment
  repo: https://github.com/lh3/minimap2
- name: SVIM-asm
  role: Calls structural variants from assembly-to-reference alignments; v1.0.3 used
    to call SVs on each assembly FASTA
- name: Truvari
  role: Benchmarks filtered SV calls (">7000 bp") against GIAB Tier 1 and T2T-HG002-Q100
    truth sets; v3.2.0
- name: RagTag
  role: Scaffolding-based comparator workflow whose reference reliance can overlook
    true SVs (only 62.5% of inversion SVs detected)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:336c181d11b316d4214991e3665d9e79babc5e75d0e887224c9f58fce9d17b27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zjjbuqi/HiFiCCL
  tools:
  - HiFiCCL
  - RagTag
  - SVIM-asm
  - Truvari
  - hifiasm
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# low-coverage-assembly-quality-assessment

## Summary

A benchmark-driven quality assessment skill for assemblies produced from low-coverage (~5x–8x) PacBio HiFi reads, judging contig quality by misassembled-contig length and large-variant detection power (SVs >7000 bp) against curated truth sets such as GIAB Tier 1 and T2T-HG002-Q100.

## When to use

Use when evaluating whether assemblies from ~5x–8x HiFi reads (e.g. HiFiCCL-Hifiasm primary+alternate, HiFlye, LJA) improve detection of large structural variants (>7000 bp) against GIAB Tier 1 or T2T-HG002-Q100 truth sets.

## When NOT to use

- Input is high-coverage (>25x) HiFi data — HiFiCCL and this assessment workflow are designed for low-coverage (~5x–8x) scenarios where de novo assembly alone is challenged
- Input reads are noisy ONT/CLR reads — the framework targets high-fidelity (HiFi) reads, and the compared assemblers (Hifiasm, HiFlye, LJA) are HiFi-based
- The goal is read-based SV calling (e.g. Sniffles2-style on BAM/CRAM) rather than assembly-based calling — the article uses Sniffles2 only as a recall comparator (0.3526) for the assembly-based SVIM-asm approach

## Inputs

- Primary and alternate assembly FASTA files from HiFiCCL-Hifiasm, Hifiasm (~5x), or HiFlye/LJA (~8x)
- Reference genome FASTA (GRCh37 for SV benchmarking; GRCh38 used as assembly reference in the paper)
- GIAB Tier 1 structural variant truth set (VCF)
- T2T-HG002-Q100 benchmark SV calls

## Outputs

- Precision/recall/F1 metrics per assembly and benchmark, written to svim_asm_sv_benchmark_metrics.tsv
- Filtered SV call sets (SVs >7000 bp) per assembly in VCF
- Insertion size-stratified F1 per bin (10–15 kb and 15–20 kb)
- Misassembled-contig length comparison vs hifiasm-alone assemblies

## How to apply

Obtain primary and alternate assemblies from the manuscript's Zenodo deposit (DOI: 10.5281/zenodo.17204429) for the ~5x HiFiCCL-Hifiasm and Hifiasm datasets, and HiFlye/LJA assemblies for the ~8x datasets. Align each assembly FASTA to the reference (GRCh37/GRCh38) in assembly-to-reference space using minimap2's asm5 preset. Run SVIM-asm (v1.0.3) on each assembly to call structural variants, filter calls to >7000 bp, and benchmark with Truvari (v3.2.0) against GIAB Tier 1 and T2T-HG002-Q100, computing precision, recall, and F1 on primary plus alternate assemblies combined. Additionally stratify insertion calls into 10–15 kb and 15–20 kb bins to check F1 per bin, and compare misassembled-contig length against hifiasm-alone assemblies (expected average reduction 21.19%, up to 38.58%). The rationale is that chromosome-binned de novo assembly from low-coverage reads reconstructs new SVs that reference-guided scaffolding alone (e.g. RagTag) may miss.

## Related tools

- **HiFiCCL** (Reference-guided chromosome-by-chromosome assembly framework producing the low-coverage primary/alternate assemblies under evaluation) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Base assembler run on HiFiCCL chromosome bins and on unclustered reads (~5x) to generate primary and alternate assemblies) — https://github.com/chhylp123/hifiasm
- **minimap2** (Aligns assembly contigs to the reference genome (asm5 preset) to support assembly-to-reference SV calling and misassembly assessment) — https://github.com/lh3/minimap2
- **SVIM-asm** (Calls structural variants from assembly-to-reference alignments; v1.0.3 used to call SVs on each assembly FASTA)
- **Truvari** (Benchmarks filtered SV calls (">7000 bp") against GIAB Tier 1 and T2T-HG002-Q100 truth sets; v3.2.0)
- **RagTag** (Scaffolding-based comparator workflow whose reference reliance can overlook true SVs (only 62.5% of inversion SVs detected))

## Examples

```
minimap2 -cx asm5 GRCh37.fa hificcl_hifiasm.primary.fa > asm.paf && svim asm --sample HG002 svim_out asm.paf
```

## Evaluation signals

- On GIAB Tier 1, SVIM-asm on HiFiCCL-Hifiasm assemblies should reach precision 0.9193, recall 0.5643, and F1 0.6993 for SVs >7000 bp
- Against T2T-HG002-Q100, recall should improve to 0.4369 (vs Sniffles2's 0.3526) with precision 0.9255 and F1 0.6165 when combining primary+alternate assemblies
- Insertion F1 should fall near 0.5851 in the 10–15 kb bin and 0.6038 in the 15–20 kb bin
- Misassembled-contig length for HiFiCCL-Hifiasm should be reduced by ~21.19% on average (up to 38.58%) relative to hifiasm alone on 45 human datasets at ~5x
- Metrics table svim_asm_sv_benchmark_metrics.tsv must be populated per assembly with all three metrics for both benchmarks

## Limitations

- Many genomic regions remain missing at 5x coverage; assembly completeness gaps are unresolved
- RagTag scaffolding relies on reference alignments and may overlook true SVs — only 62.5% of inversion SVs were detected via scaffolding
- No version numbers are stated for HiFiCCL, Hifiasm, or RagTag, complicating exact reproduction
- Only ~5x (Hifiasm/HiFiCCL-Hifiasm) and ~8x (HiFlye/LJA) coverage points are benchmarked; other low-coverage regimes are untested

## Evidence

- [abstract] misassembly reduction claim: "reduces the length of misassembled contigs relative to hifiasm by an average of 21.19% and up to 38.58%"
- [abstract] large SV detection claim: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [methods] SV size threshold in task workflow: "using the GRCh38 sequence as the reference"
- [discussion] RagTag scaffolding caveat: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"
- [discussion] coverage gaps: "due to low coverage and other limitations, many genomic regions remain missing"
- [readme] minimap2 assembly alignment preset: [evidence span withheld — anchor status: external_doc]

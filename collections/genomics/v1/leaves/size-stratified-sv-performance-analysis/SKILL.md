---
name: size-stratified-sv-performance-analysis
description: Use when evaluating whether assemblies built from low-coverage (~5x-8x)
  PacBio HiFi reads improve detection of large structural variants, and you need size-stratified
  precision/recall/F1 against GIAB or T2T benchmark VCFs.
when_to_use_negative:
- Input is short-read variant calling (e.g. Sniffles2 on reads rather than SVIM-asm
  on assemblies) or the SVs of interest are smaller than the 7000 bp cutoff — the
  size-stratified bins (10-15 kb, 15-20 kb) only characterize large SVs
- The sample has no curated benchmark truth set (GIAB Tier 1 or T2T-HG002-Q100); Truvari-based
  precision/recall cannot be computed without one
- The goal is contig-level assembly quality (misassembly length, NGA50) rather than
  SV detection performance
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: SVIM-asm
  role: Calls structural variants from assembly-to-reference alignments (v1.0.3)
- name: Truvari
  role: Benchmark filtered SV calls against GIAB Tier 1 and T2T-HG002-Q100 truth sets
    (v3.2.0), producing precision/recall/F1
- name: minimap2
  role: Aligns assembly contigs to the GRCh37 reference to create assembly-to-reference
    alignments for SV calling
  repo: https://github.com/lh3/minimap2
- name: HiFiCCL
  role: Produces the low-coverage chromosome-binned assemblies whose SV-detection
    performance is being benchmarked
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Base assembler combined with HiFiCCL to generate the ~5x assemblies under
    evaluation
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:daa00ceaf3ea9f52cbffea5c22c3e6f7c90b4786c2d480e964859832b089d683
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - HiFiCCL
  - SVIM-asm
  - Truvari
  - hifiasm
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# size-stratified-sv-performance-analysis

## Summary

Benchmark assembly-derived structural variant calls from low-coverage HiFi assemblies (HiFiCCL + Hifiasm/HiFlye/LJA) against curated truth sets, filtering to SVs >7000 bp and stratifying insertion calls into 10-15 kb and 15-20 kb size bins to quantify precision, recall, and F1.

## When to use

Use when evaluating whether assemblies built from low-coverage (~5x-8x) PacBio HiFi reads improve detection of large structural variants, and you need size-stratified precision/recall/F1 against GIAB or T2T benchmark VCFs.

## When NOT to use

- Input is short-read variant calling (e.g. Sniffles2 on reads rather than SVIM-asm on assemblies) or the SVs of interest are smaller than the 7000 bp cutoff — the size-stratified bins (10-15 kb, 15-20 kb) only characterize large SVs
- The sample has no curated benchmark truth set (GIAB Tier 1 or T2T-HG002-Q100); Truvari-based precision/recall cannot be computed without one
- The goal is contig-level assembly quality (misassembly length, NGA50) rather than SV detection performance

## Inputs

- Primary and alternate assembly FASTA files (HiFiCCL-Hifiasm, Hifiasm ~5x; HiFlye/LJA ~8x) from Zenodo DOI 10.5281/zenodo.17204429
- GRCh37 reference genome FASTA
- GIAB Tier 1 structural variant truth VCF
- T2T-HG002-Q100 benchmark VCF
- Sniffles2 call set for recall comparison on the T2T benchmark

## Outputs

- Filtered SV call sets restricted to variants >7000 bp per assembly
- Per-benchmark precision, recall, and F1 for primary+alternate assemblies (e.g. F1 0.6993 on GIAB Tier 1; F1 0.6165 on T2T-HG002-Q100)
- Insertion-call F1 per size bin: 0.5851 (10-15 kb) and 0.6038 (15-20 kb)
- Consolidated metrics table svim_asm_sv_benchmark_metrics.tsv

## How to apply

Obtain primary and alternate assemblies from the manuscript's Zenodo deposit (DOI: 10.5281/zenodo.17204429): Hifiasm and HiFiCCL-Hifiasm assemblies for the ~5x HG002 data, and HiFlye/LJA variant assemblies for the ~8x data. Align each assembly FASTA to GRCh37 with minimap2 (asm-to-reference space) and run SVIM-asm (v1.0.3) to call structural variants from the alignments. Filter calls to SVs larger than 7000 bp, then compare against the GIAB Tier 1 truth set and the T2T-HG002-Q100 benchmark using Truvari (v3.2.0), computing precision, recall, and F1 for primary plus alternate assemblies combined. Finally stratify insertion calls by size into 10-15 kb and 15-20 kb bins and compute F1 per bin, verifying against reported values (Tier 1: precision 0.9193, recall 0.5643, F1 0.6993; T2T: recall 0.4369 vs Sniffles2's 0.3526, precision 0.9255, F1 0.6165; insertion F1 0.5851 and 0.6038). The rationale is that HiFiCCL's chromosome-wise binning produces assemblies that excel specifically at large germline SVs, so benchmarking only the >7000 bp regime isolates the effect the method targets.

## Related tools

- **SVIM-asm** (Calls structural variants from assembly-to-reference alignments (v1.0.3))
- **Truvari** (Benchmark filtered SV calls against GIAB Tier 1 and T2T-HG002-Q100 truth sets (v3.2.0), producing precision/recall/F1)
- **minimap2** (Aligns assembly contigs to the GRCh37 reference to create assembly-to-reference alignments for SV calling) — https://github.com/lh3/minimap2
- **HiFiCCL** (Produces the low-coverage chromosome-binned assemblies whose SV-detection performance is being benchmarked) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Base assembler combined with HiFiCCL to generate the ~5x assemblies under evaluation) — https://github.com/chhylp123/hifiasm

## Examples

```
minimap2 -cx asm5 GRCh37.fa hificcl_hifiasm.hap1.fa > hificcl_asm2ref.paf  # then SVIM-asm on the alignment, filter >7000 bp, and truvari bench against GIAB Tier 1
```

## Evaluation signals

- Reproduced GIAB Tier 1 metrics for SVIM-asm on HiFiCCL-Hifiasm >7000 bp calls match precision 0.9193, recall 0.5643, F1 0.6993
- T2T-HG002-Q100 comparison shows SVIM-asm recall 0.4369 exceeding Sniffles2's 0.3526, with precision 0.9255 and F1 0.6165 when combining primary+alternate assemblies
- Insertion F1 per size bin falls at ~0.5851 for the 10-15 kb bin and ~0.6038 for the 15-20 kb bin
- Output TSV (svim_asm_sv_benchmark_metrics.tsv) contains one precision/recall/F1 row per assembly per benchmark, restricted to calls >7000 bp
- Truvari version (v3.2.0) and reference genome (GRCh37) are fixed and identical across all compared call sets

## Limitations

- RagTag-based scaffolding of the assemblies overlooks true SVs — only 62.5% of inversion SVs were detected in the article's discussion
- Many genomic regions remain missing at ~5x coverage due to low coverage and other limitations, capping recall (e.g. 0.5643 on Tier 1)
- SVIM-asm results depend on primary+alternate assembly combination; benchmarking primary assemblies alone understates performance (F1 0.6165 only when combining both)

## Evidence

- [abstract] These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [intro] large germline SV detection larger than 10,000 bp is a validation application: "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp)"
- [readme] minimap2 asm-to-asm/asm-to-reference preset used to align assemblies: [evidence span withheld — anchor status: external_doc]
- [discussion] the scaffolding process may miss true SVs: "the scaffolding process relies on reference genome alignments, which may overlook true SVs"

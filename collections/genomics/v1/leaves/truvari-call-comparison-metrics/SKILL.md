---
name: truvari-call-comparison-metrics
description: Use when evaluating large SV detection (variants >7000 bp) from de novo
  assemblies against GIAB Tier 1 or T2T-HG002-Q100 truth sets and you need precision/recall/F1
  metrics per assembly and per insertion-size bin.
when_to_use_negative:
- Benchmarking SVs smaller than 7000 bp — the comparison deliberately filters calls
  to structural variants larger than 7000 bp, so metrics on shorter variants are not
  comparable to the reported values
- Calling SVs directly from reads (e.g. Sniffles2-style read-based callers) rather
  than from assembly FASTAs — this skill operates in assembly-to-reference alignment
  space via SVIM-asm, with Sniffles2 only serving as the recall comparator (0.3526
  vs 0.4369)
- Plant genome evaluations beyond HG002/HG00438-style human benchmarks — the paper
  notes 'For plant genomes with higher genetic diversity, we have tested only two
  samples', so these human truth-set metrics do not transfer
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Truvari
  role: Compares filtered SVIM-asm calls (>7000 bp) against GIAB Tier 1 and T2T-HG002-Q100
    truth sets to compute precision, recall, and F1 (v3.2.0)
- name: SVIM-asm
  role: Calls structural variants from each assembly FASTA in assembly-to-reference
    alignment space (v1.0.3)
- name: minimap2
  role: Produces assembly-to-reference/assembly alignments (asm preset, PAF) that
    underpin the alignment space in which SVs are called
  repo: https://github.com/lh3/minimap2
- name: HiFiCCL
  role: Generates the low-coverage (~5x) HiFi assemblies whose SV-detection performance
    is benchmarked
  repo: https://github.com/zjjbuqi/HiFiCCL
- name: hifiasm
  role: Base assembler producing the primary and alternate assemblies evaluated at
    ~5x coverage
  repo: https://github.com/chhylp123/hifiasm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/advs.202515308
    title: Reference‐Guided Chromosome‐by‐Chromosome <i>de novo</i> Assembly at Scale
      Using Low‐Coverage High‐Fidelity Long‐Reads with HiFiCCL
  source_content_hash: sha256:462059f6227e788cb8270b604eb7a1d8b556ade8c84d37a5694797a8b7883f69
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# truvari-call-comparison-metrics

## Summary

Benchmark assembly-derived structural variant calls with Truvari (v3.2.0) by comparing SVIM-asm (1.0.3) calls from HiFiCCL-based HG002 assemblies against GIAB Tier 1 and T2T-HG002-Q100 truth sets, reporting precision, recall, and F1 for SVs >7000 bp.

## When to use

Use when evaluating large SV detection (variants >7000 bp) from de novo assemblies against GIAB Tier 1 or T2T-HG002-Q100 truth sets and you need precision/recall/F1 metrics per assembly and per insertion-size bin.

## When NOT to use

- Benchmarking SVs smaller than 7000 bp — the comparison deliberately filters calls to structural variants larger than 7000 bp, so metrics on shorter variants are not comparable to the reported values
- Calling SVs directly from reads (e.g. Sniffles2-style read-based callers) rather than from assembly FASTAs — this skill operates in assembly-to-reference alignment space via SVIM-asm, with Sniffles2 only serving as the recall comparator (0.3526 vs 0.4369)
- Plant genome evaluations beyond HG002/HG00438-style human benchmarks — the paper notes 'For plant genomes with higher genetic diversity, we have tested only two samples', so these human truth-set metrics do not transfer

## Inputs

- Primary and alternate assembly FASTA files from the manuscript's Zenodo deposit (DOI: 10.5281/zenodo.17204429): HiFiCCL-Hifiasm and Hifiasm (~5x), HiFlye and LJA variant assemblies (~8x)
- Reference genome GRCh37 for assembly-to-reference alignment and benchmarking
- GIAB Tier 1 truth set (HG002)
- T2T-HG002-Q100 benchmark truth set
- VCF output of SVIM-asm (1.0.3) calls per assembly

## Outputs

- Filtered VCF/BedPE of SV calls restricted to variants larger than 7000 bp
- Truvari (v3.2.0) comparison outputs giving precision, recall, and F1 for primary plus alternate assemblies against GIAB Tier 1 and T2T-HG002-Q100
- Insertion-stratified F1 per size bin (10-15 kb and 15-20 kb)
- svim_asm_sv_benchmark_metrics.tsv containing all precision/recall/F1 results

## How to apply

Run SVIM-asm (1.0.3) on each assembly FASTA — HiFiCCL-Hifiasm and Hifiasm primary/alternate assemblies at ~5x, HiFlye and LJA variants at ~8x — to call structural variants in assembly-to-reference alignment space (e.g. after minimap2 asm-style alignment). Filter calls to structural variants larger than 7000 bp, then compare the filtered VCFs against the GIAB Tier 1 truth set and the T2T-HG002-Q100 benchmark using Truvari (v3.2.0) with GRCh37 as the reference genome, computing precision, recall, and F1 for the primary plus alternate assemblies combined. Additionally stratify insertion calls by size (10-15 kb and 15-20 kb bins) and compute F1 per bin. Validate the reproduced metrics against the reported values: precision 0.9193, recall 0.5643, F1 0.6993 (GIAB Tier 1); recall 0.4369 vs Sniffles2's 0.3526, precision 0.9255, F1 0.6165 (T2T-HG002-Q100); insertion F1 0.5851 (10-15 kb) and 0.6038 (15-20 kb). Write all precision/recall/F1 results to svim_asm_sv_benchmark_metrics.tsv.

## Related tools

- **Truvari** (Compares filtered SVIM-asm calls (>7000 bp) against GIAB Tier 1 and T2T-HG002-Q100 truth sets to compute precision, recall, and F1 (v3.2.0))
- **SVIM-asm** (Calls structural variants from each assembly FASTA in assembly-to-reference alignment space (v1.0.3))
- **minimap2** (Produces assembly-to-reference/assembly alignments (asm preset, PAF) that underpin the alignment space in which SVs are called) — https://github.com/lh3/minimap2
- **HiFiCCL** (Generates the low-coverage (~5x) HiFi assemblies whose SV-detection performance is benchmarked) — https://github.com/zjjbuqi/HiFiCCL
- **hifiasm** (Base assembler producing the primary and alternate assemblies evaluated at ~5x coverage) — https://github.com/chhylp123/hifiasm

## Examples

```
minimap2 -cx asm5 GRCh37.fa hg002_hificcl_hifiasm.p_ctg.fa > asm2ref.paf && svim asm out_dir . --min_sv_size 7000 && truvari bench -b giab_tier1.vcf.gz -c svim_calls.vcf.gz -o truvari_out --includebed tier1_regions.bed
```

## Evaluation signals

- GIAB Tier 1 benchmark of HiFiCCL-Hifiasm assemblies reproduces precision 0.9193, recall 0.5643, and the highest F1 (0.6993) for SVs >7000 bp
- T2T-HG002-Q100 benchmark reproduces recall 0.4369 (exceeding Sniffles2's 0.3526), precision 0.9255, and highest F1 0.6165 combining primary+alternate assemblies
- Insertion-size stratification reproduces F1 0.5851 for the 10-15 kb bin and 0.6038 for the 15-20 kb bin
- All precision/recall/F1 rows are present in svim_asm_sv_benchmark_metrics.tsv for every evaluated assembly (HiFiCCL-Hifiasm, Hifiasm, HiFlye, LJA variants)
- Filtered call sets contain no SVs ≤7000 bp before Truvari comparison

## Limitations

- Metrics apply only to large SVs (>7000 bp) in assembly-to-reference alignment space; smaller variants are excluded by the benchmark's size filter
- Reference-guided steps can miss true SVs: 'the scaffolding process relies on reference genome alignments, which may overlook true SVs' (only 62.5% of inversion SVs detected in RagTag scaffolding)
- At ~5x coverage, 'many genomic regions remain missing', so recall is bounded by assembly completeness even when precision is high (~0.92)
- No version numbers are stated for HiFiCCL, Hifiasm, or RagTag in the source, which complicates exact reproduction of the assembly inputs feeding the benchmark

## Evidence

- [other] SVIM-asm on HiFiCCL-Hifiasm assemblies achieved precision 0.9193, recall 0.5643, and the highest F1 (0.6993) for SVs >7000 bp: [evidence span withheld — anchor status: not_found]
- [other] T2T-HG002-Q100 recall improvement over Sniffles2: [evidence span withheld — anchor status: not_found]
- [abstract] Assemblies excel at large germline SV detection: "These improved assemblies excel in detecting large germline structural variants, minimize inter-chromosome mis-scaffolding"
- [intro] Large SV detection is a validated application of HiFiCCL: "We validate the effectiveness of HiFiCCL across various applications, including contig‐level assembly, large germline SVs detection (larger than 10,000 bp)"
- [readme] minimap2 asm-preset alignment supports assembly-based SV calling: [evidence span withheld — anchor status: external_doc]

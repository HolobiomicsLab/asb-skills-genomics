---
name: structural-variant-calling-from-assemblies
description: Use when two assemblies of the same organism exist (e.g., Canu >500k
  contigs vs CaSM scaffolds) and you need SV coordinates to benchmark reference-free
  error calls like CRAQ CREs/CSEs against.
when_to_use_negative:
- Only one assembly is available and no second assembly or reference exists to define
  structural differences — CRAQ alone is the reference-free option
- The goal is small variant calling (SNPs/indels) rather than assembly-vs-assembly
  SVs; SyRI parameters here deliberately ignore SNPs, HDRs, INSs, and DELs < 40 bp
- The genome is polyploid — the article states evaluation accuracy 'may not be as
  accurate as the benchmarked cases' for higher ploidy levels
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SyRI
  role: Calls SVs between the Canu query assembly and CaSM reference assembly after
    merging overlapped regions
- name: Minimap2
  role: Maps ONT long reads ('-ax map-ont') and Illumina short reads ('-ax sr') to
    the query assembly
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with '-F 1796 -q 20' to remove MAPQ<20, unmapped, secondary,
    QC-failed, and duplicate reads
- name: CRAQ
  role: Calls CRE/CSE error breakpoints from filtered BAMs using clipping ratios (h
    = 0.4–0.6, f = 0.75) for overlap testing against SyRI SVs
  repo: https://github.com/JiaoLaboratory/CRAQ
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:cb3cdc8c1f779873f90541d89efd7dc0caf7c739c4795adb84587d2515e99e22
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CRAQ
  - Minimap2
  - SAMtools
  - SyRI
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# structural-variant-calling-from-assemblies

## Summary

Call structural variants between two de novo assemblies (e.g., SyRI with a Canu assembly as query against a CaSM assembly as reference) and quantify their overlap with CRAQ-detected assembly errors to distinguish true assembly errors from genuine haplotype/structural divergence.

## When to use

Use when two assemblies of the same organism exist (e.g., Canu >500k contigs vs CaSM scaffolds) and you need SV coordinates to benchmark reference-free error calls like CRAQ CREs/CSEs against.

## When NOT to use

- Only one assembly is available and no second assembly or reference exists to define structural differences — CRAQ alone is the reference-free option
- The goal is small variant calling (SNPs/indels) rather than assembly-vs-assembly SVs; SyRI parameters here deliberately ignore SNPs, HDRs, INSs, and DELs < 40 bp
- The genome is polyploid — the article states evaluation accuracy 'may not be as accurate as the benchmarked cases' for higher ploidy levels

## Inputs

- Query genome assembly FASTA (Canu contig-level assembly of S. pennellii LYC1722, >500k contigs)
- Reference genome assembly FASTA (CaSM assembly of S. pennellii)
- Nanopore long reads (FASTQ) for read-mapping-based error calling
- Illumina short reads (FASTQ)
- CRAQ output coordinates of CREs and CSEs from the filtered BAMs

## Outputs

- SyRI SV coordinates (TRANS/INV/DEL/INS/DUP types) between query and reference assemblies
- Bidirectional overlap fractions: CRAQ errors overlapping SyRI SVs (e.g., 5736/8029 = 71.4%) and SyRI SVs overlapping CRAQ errors (e.g., 6539/13,114 = 49.8%)
- Categorized table of CRAQ-unique and SyRI-unique calls (errors in CaSM reference, heterozygous sites, noisy base-error clusters, low-confidence regions, errors shared between assemblies, others)

## How to apply

Run SyRI with the fragmented Canu assembly (>500k contigs) as the query genome and the CaSM assembly as the reference, ignoring SNPs, HDRs, INSs, and DELs < 40 bp and merging overlapped SV regions to get a nonredundant SV set (13,114 SVs in the S. pennellii case). Align the same Nanopore and Illumina reads to the query assembly with Minimap2 ('-ax map-ont' for ONT long reads, '-ax sr' for short reads) and filter with 'samtools view -F 1796 -q 20' before CRAQ calling, so both error and SV sets are computed on identically filtered evidence. Classify a CRAQ CRE/CSE breakpoint (called with default clipping cutoffs h = 0.4–0.6 and f = 0.75) as overlapping a SyRI SV if the breakpoint falls within the SV boundary or the adjacent 50-bp region, then compute the bidirectional overlap fractions (CRAQ errors within SVs, and SVs within CRAQ errors). Assign non-overlapping calls to categories such as errors in the CaSM reference, heterozygous sites, noisy base-error clusters, CRAQ low-confidence regions, and errors shared between Canu and CaSM to reproduce the Fig. 3a breakdown.

## Related tools

- **SyRI** (Calls SVs between the Canu query assembly and CaSM reference assembly after merging overlapped regions)
- **Minimap2** (Maps ONT long reads ('-ax map-ont') and Illumina short reads ('-ax sr') to the query assembly) — https://github.com/lh3/minimap2
- **SAMtools** (Filters alignments with '-F 1796 -q 20' to remove MAPQ<20, unmapped, secondary, QC-failed, and duplicate reads)
- **CRAQ** (Calls CRE/CSE error breakpoints from filtered BAMs using clipping ratios (h = 0.4–0.6, f = 0.75) for overlap testing against SyRI SVs) — https://github.com/JiaoLaboratory/CRAQ

## Examples

```
minimap2 -ax map-ont Canu_spennellii.fa ont_reads.fq | samtools view -F 1796 -q 20 -b - > ont.filtered.bam
```

## Evaluation signals

- Reproduce the reported overlap: ~71.4% (5736/8029) of CRAQ breakpoints overlapping SyRI SVs, and 49.8% (6539/13,114) of SyRI SVs overlapping CRAQ breakpoints
- CRAQ should report 8029 breakpoints total (7910 CREs, 119 CSEs) on the Canu S. pennellii assembly
- SyRI SV set should reach ~13,114 calls after merging overlapped SV regions and excluding events < 40 bp
- Every overlap call should satisfy the 50-bp rule: breakpoint within the SV boundary or the adjacent 50-bp region
- Filtering check: both alignment BAMs must have passed 'samtools view -F 1796 -q 20' (no MAPQ<20, unmapped, secondary, QC-failed, or duplicate alignments) before CRAQ

## Limitations

- Optical-mapping-based validation only infers approximate misjoin locations, so SV/overlap boundaries carry positional uncertainty
- Error breakpoint clipping ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating classification
- CRAQ reports ambiguous regions (coverage above h threshold and below f) without definitive classification, which can inflate or deflate overlap counts
- Read mapping is the most resource-intensive step; users must manually split query sequences into multiple fragments to parallelize

## Evidence

- [discussion] The Canu assemblies (>500k) of S. pennellii were input as the query genome and the CaSM assembly was input as the reference: "The Canu assemblies (>500k) of S. pennellii were input as the query genome and the CaSM assembly was input as the reference"
- [discussion] filtered out using the '-F 1796 -q 20' parameters in 'samtools view'35: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [discussion] A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75): "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [abstract] Minimap2 '-ax sr' and '-ax map-hifi'('map-hifi' for PacBio HiFi,'map-pb' for PacBio CLR, 'map-ont' for ONT library) options were used: "By default, Minimap2 ‘–ax sr’ and  ‘–ax map-hifi’(‘map-hifi’ for PacBio HiFi,‘map-pb’ for PacBio CLR, ‘map-ont’ for ONT library) options were used"
- [discussion] CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios.: "CRAQ distinguishes error breakpoints and heterozygous loci based on read-mapping coverage data and effective clipping ratios."

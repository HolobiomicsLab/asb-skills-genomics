---
name: long-read-to-contig-alignment
description: Use when corrected long reads must be mapped to haplotype-collapsed contigs
  to obtain pileup-based heterozygous SNP sites for read grouping and inconsistent-overlap
  detection.
when_to_use_negative:
- The reads are raw (uncorrected) and the goal is read-overlap detection — PECAT instead
  uses minimap2 'ava-pb' all-against-all overlaps for raw reads
- The input is PacBio HiFi or Nanopore duplex data, where PECAT is reported to achieve
  only mediocre performance
- A reference-based SNP caller is appropriate — corrected reads alone are insufficient
  for accurate SNP calling (unlike Hifiasm with HiFi reads), and PECAT falls back
  to Clair3/Medaka for Nanopore polishing
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Maps corrected reads to the haplotype-collapsed first-round contigs with '-x
    map-pb -c -p 0.5 -r 1000' (PacBio CLR) to produce base-level alignments for SNP
    allele counting
  repo: https://github.com/lh3/minimap2
- name: PECAT
  role: Pipeline that consumes the read-to-contig alignments to call heterozygous
    SNPs, build SNP-allele vectors, and flag inconsistent overlaps for the second-round
    assembly
  repo: https://github.com/lemene/PECAT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:006e4ea48bad96280194ab10c539527d59f0fe8b18478ec50389414239f160cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# long-read-to-contig-alignment

## Summary

Aligns long noisy reads (corrected PacBio CLR or Nanopore reads) back to a haplotype-collapsed first-round assembly using minimap2, producing the base-level alignments needed for downstream SNP calling and read phasing in PECAT's second diploid assembly round.

## When to use

Use when corrected long reads must be mapped to haplotype-collapsed contigs to obtain pileup-based heterozygous SNP sites for read grouping and inconsistent-overlap detection.

## When NOT to use

- The reads are raw (uncorrected) and the goal is read-overlap detection — PECAT instead uses minimap2 'ava-pb' all-against-all overlaps for raw reads
- The input is PacBio HiFi or Nanopore duplex data, where PECAT is reported to achieve only mediocre performance
- A reference-based SNP caller is appropriate — corrected reads alone are insufficient for accurate SNP calling (unlike Hifiasm with HiFi reads), and PECAT falls back to Clair3/Medaka for Nanopore polishing

## Inputs

- Corrected long reads (FASTA/FASTQ; e.g. PacBio CLR or Nanopore R9/R10 corrected reads)
- Haplotype-collapsed first-round assembly contigs (e.g. $PROJECT/3-assemble/primary.fasta)

## Outputs

- Read-to-contig alignments (PAF with CIGAR via minimap2 -c)
- Heterozygous SNP site calls with allele base frequencies
- Per-read SNP-allele vectors H_r(s) used for haplotype grouping

## How to apply

Map the corrected reads to the haplotype-collapsed first-round contigs ($PROJECT/3-assemble/primary.fasta) with minimap2, using preset-specific parameters such as '-x map-pb -c -p 0.5 -r 1000' for PacBio CLR reads (the -c flag produces CIGAR-containing PAF alignments needed for base-frequency counting). From the aligned pileup, call heterozygous SNP sites whose coverage falls in [10, 1000] and whose second-most-common base counts pass the r_l=0.4, r_h=0.2, c_l=10, c_h=100 thresholds. Each read then receives an SNP-allele vector H_r(s) over the called sites, which feeds the bisecting-k-means read grouping and the subsequent inconsistent-overlap flagging (>=8 differing alleles for corrected reads, >=6 for raw reads). The alignment must retain base-level detail; mapping-only output without CIGAR is insufficient for allele extraction.

## Related tools

- **minimap2** (Maps corrected reads to the haplotype-collapsed first-round contigs with '-x map-pb -c -p 0.5 -r 1000' (PacBio CLR) to produce base-level alignments for SNP allele counting) — https://github.com/lh3/minimap2
- **PECAT** (Pipeline that consumes the read-to-contig alignments to call heterozygous SNPs, build SNP-allele vectors, and flag inconsistent overlaps for the second-round assembly) — https://github.com/lemene/PECAT

## Examples

```
minimap2 -x map-pb -c -p 0.5 -r 1000 $PROJECT/3-assemble/primary.fasta corrected_reads.fasta > corrected_to_ctg.paf
```

## Evaluation signals

- Alignment output is in PAF format and contains CIGAR strings (minimap2 run with -c), enabling per-site base frequency extraction
- Called SNP sites all have coverage within the [10, 1000] range and second-most-common base counts satisfying r_l=0.4, r_h=0.2, c_l=10, c_h=100
- Every read passing grouping has a defined SNP-allele vector H_r(s) over the called heterozygous sites
- Downstream inconsistent_read_pairs.tsv is parseable and every flagged pair satisfies its applicable allele threshold (>=8 corrected-read alleles or >=6 raw-read alleles)
- Read-to-contig distance differences are < 1000 bp against the haplotype-collapsed contigs (raw-read threshold max(2500, 0.05*Dc))

## Limitations

- Corrected reads alone do not yield accurate SNP calls (unlike Hifiasm with HiFi reads), so SNP accuracy is low in low-heterozygosity regions
- PECAT needs higher sequencing coverage to obtain high-quality phased assemblies
- This step is specific to the PECAT correct-then-assemble two-round strategy; the alignment target is the haplotype-collapsed first-round assembly, not a reference genome

## Evidence

- [discussion] Map corrected reads to first-round assembly and call heterozygous SNPs: "We map corrected reads to the first round of assembly using minimap2 with parameters “-x map-pb -c -p 0.5 -r 1000” for PacBio CLR reads"
- [discussion] Generate haplotype-collapsed genome and call SNPs: "PECAT then first generates a haplotype-collapsed genome and calls SNPs by aligning the corrected reads to the haplotype-collapsed genome."
- [discussion] SNP site coverage filter: "(1) Its coverage is in the range [10, 1000]."
- [discussion] Corrected reads insufficient for SNP calling: "even after error corrections, it is not able to accurately call SNPs by just aligning corrected reads as the Hifiasm did for HiFi reads."
- [readme] minimap2 CIGAR support in README: [evidence span withheld — anchor status: external_doc]

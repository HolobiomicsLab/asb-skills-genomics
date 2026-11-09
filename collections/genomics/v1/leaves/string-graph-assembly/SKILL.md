---
name: string-graph-assembly
description: Use when assembling diploid genomes from long noisy reads (Nanopore R9/R10,
  PacBio CLR) where overlap-graph construction and phasing of haplotype-specific contigs
  is the goal.
when_to_use_negative:
- Input reads are PacBio HiFi or Nanopore duplex — PECAT achieves mediocre performance
  on these highly accurate read types
- Input is short-read data or the goal is reference-based variant calling rather than
  de novo diploid assembly
- Dataset has very low heterozygosity rate — SNP calling accuracy is low in such regions,
  undermining inconsistent-overlap removal
edam_operation: http://edamontology.org/operation_0525
edam_topics: []
tools:
- name: PECAT
  role: Implements the string graph construction, best-overlap-graph scoring, and
    two-sets-of-contigs diploid assembly
  repo: github.com/lemene/PECAT
- name: minimap2
  role: Detects candidate read-to-read overlaps (e.g. -x ava-pb) and maps corrected
    reads to the first-round assembly for SNP calling
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Calls SNPs on raw Nanopore reads for the loose-threshold inconsistent-overlap
    check
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Optional polishing to further improve assembly quality after string-graph
    assembly
  repo: https://github.com/nanoporetech/medaka
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:69f8d240e331a61ae269e237c8631bb0e026c3e70b455d68ab107240c3e1cef0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Clair3
  - Medaka
  - PECAT
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# string-graph-assembly

## Summary

Build and traverse a string graph from long-read overlaps to assemble haplotype-collapsed and phased diploid contigs, as implemented in PECAT with minimap2 candidate overlaps and Myers' transitive-edge reduction.

## When to use

Use when assembling diploid genomes from long noisy reads (Nanopore R9/R10, PacBio CLR) where overlap-graph construction and phasing of haplotype-specific contigs is the goal.

## When NOT to use

- Input reads are PacBio HiFi or Nanopore duplex — PECAT achieves mediocre performance on these highly accurate read types
- Input is short-read data or the goal is reference-based variant calling rather than de novo diploid assembly
- Dataset has very low heterozygosity rate — SNP calling accuracy is low in such regions, undermining inconsistent-overlap removal

## Inputs

- Corrected long reads (FASTA/FASTQ) from Nanopore R9, Nanopore R10, or PacBio CLR sequencing (~80X or all reads if below 80X)
- Raw long reads for loose-threshold inconsistent-overlap detection
- PECAT configuration file (cfgfile) with align_rd2rd_options, align_filter_options, asm1_assemble_options, phase_filter_options

## Outputs

- $PROJECT/3-assemble/primary.fasta — haplotype-collapsed contigs from round-one string graph assembly
- Round-two contig files under $PROJECT/5-assemble — primary/alternate and dual haplotype contigs in FASTA format

## How to apply

Detect candidate read-to-read overlaps among corrected reads with minimap2 (e.g. -x ava-pb for PacBio CLR), extend alignments to read ends with the diff algorithm, and filter overlaps still having long overhangs. Build the string graph by marking transitive overlaps with Myers' algorithm, removing edges whose identities fall below an adaptive threshold, and bridging contained reads, so identity computation (local alignment) is only performed on the small set of active edges. Score best-overlap-graph edges as s_i = (n_i+ * w - n_i-, l_i) and emit haplotype-collapsed contigs ('pecat.pl assemble cfgfile'). For the diploid round, reuse round-one overlaps and remove inconsistent overlaps identified by heterozygous SNP checks — strict threshold of 8 different SNP alleles on corrected reads, loose threshold of 6 different SNP alleles on raw reads — then apply the alternate-branch heuristic (>30% of reads in the shorter branch inconsistent with the other branch) to output primary/alternate and dual haplotype contig files.

## Related tools

- **PECAT** (Implements the string graph construction, best-overlap-graph scoring, and two-sets-of-contigs diploid assembly) — github.com/lemene/PECAT
- **minimap2** (Detects candidate read-to-read overlaps (e.g. -x ava-pb) and maps corrected reads to the first-round assembly for SNP calling) — https://github.com/lh3/minimap2
- **Clair3** (Calls SNPs on raw Nanopore reads for the loose-threshold inconsistent-overlap check) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Optional polishing to further improve assembly quality after string-graph assembly) — https://github.com/nanoporetech/medaka

## Examples

```
pecat.pl assemble cfgfile
```

## Evaluation signals

- Demo run produces $PROJECT/3-assemble/primary.fasta and round-two contig files under $PROJECT/5-assemble as valid FASTA
- Phase block NG50 of the diploid assembly (reference: 59.4/58.0 Mb for HG002 with Nanopore R10 reads)
- All overlaps surviving the overhang filter have no long overhangs after extension to read ends with the diff algorithm
- Inconsistent overlaps flagged only when read pairs carry ≥8 different SNP alleles (corrected reads) or ≥6 different SNP alleles (raw reads)
- Alternate branches emitted only when >30% of reads in the shorter branch are inconsistent with the other branch

## Limitations

- PECAT needs higher data coverage to obtain high-quality phased assemblies
- SNP calling accuracy is low in low heterozygosity rate regions
- Error correction may not distinguish small errors from heterozygote alleles in very low heterozygosity regions, even with highly accurate reads
- Mediocre performance on Nanopore duplex and PacBio HiFi reads; detailed parameters are only documented in Supplementary Notes 4 and 5

## Evidence

- [abstract] We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph.: "We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph."
- [discussion] Overhang filtering after diff extension: "we extend the alignment to the ends of the reads with the diff36 algorithm and filter out overlaps still with long overhangs."
- [discussion] Adaptive identity threshold edge removal: "The edges whose identities are less than the threshold are marked as low quality and removed."
- [discussion] Strict inconsistent-overlap threshold: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Reuse round-one overlaps for diploid reassembly: "reuse the overlaps built in the first round of assembly and just simplify the overlaps by removing inconsistent ones before reassembling the diploid genome"
- [discussion] Alternate branch heuristic: "More than 30% of reads in the shorter branch are inconsistent with the reads in the other branch."
- [methods] PECAT repo availability: "git clone --recursive https://github.com/lemene/PECAT.git"
- [readme] PECAT README description: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."
- [readme] minimap2 overlap detection usage: [evidence span withheld — anchor status: external_doc]

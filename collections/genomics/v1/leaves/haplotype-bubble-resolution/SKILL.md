---
name: haplotype-bubble-resolution
description: Use when assembling a diploid genome from long noisy reads (Nanopore
  R9/R10, PacBio CLR) and the string graph contains heterozygous bubbles/branches
  that must be split into primary and alternate haplotype contigs.
when_to_use_negative:
- Input is high-accuracy PacBio HiFi or Nanopore duplex reads — the article reports
  PECAT achieves mediocre performance on these data types
- The organism is haploid or near-clonal with very low heterozygosity — SNP calling
  accuracy is low in low heterozygosity rate regions and error correction may not
  distinguish errors from heterozygote alleles
- Available data coverage is low — PECAT needs higher coverage of data to obtain a
  high-quality phased assembly
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3673
tools:
- name: PECAT
  role: Two-round string-graph assembler that scores best-overlap-graph edges, removes
    SNP-inconsistent overlaps, and emits primary/alternate haplotype contigs
  repo: https://github.com/lemene/PECAT
- name: minimap2
  role: Detects candidate read-to-read overlaps (ava-pb/ava-ont presets) and maps
    corrected reads to the first-round assembly for heterozygous SNP calling
  repo: https://github.com/lh3/minimap2
- name: Clair3
  role: Calls SNPs on raw Nanopore reads for the loose-threshold inconsistent-overlap
    check
  repo: https://github.com/HKU-BAL/Clair3
- name: Medaka
  role: Optional downstream polishing to further improve assembly quality after haplotype
    bubble resolution
  repo: https://github.com/nanoporetech/medaka
- name: Pomoxis
  role: Reference-based evaluation of the resolved contigs (e.g. qstat assembly QC)
  repo: https://github.com/nanoporetech/pomoxis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-47349-7
    title: De novo diploid genome assembly using long noisy reads
  source_content_hash: sha256:0c357b9f937755720956f351850bbb6f2a3d669267cb09f5a8a740d9482d5ef1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lemene/PECAT
  tools:
  - Clair3
  - Medaka
  - PECAT
  - Pomoxis
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# haplotype-bubble-resolution

## Summary

Resolves heterozygous bubbles in a string graph by scoring best-overlap-graph edges (s_i = (n_i+ * w - n_i-, l_i)), removing SNP-inconsistent overlaps, and applying an alternate-branch heuristic to emit primary and alternate haplotype contigs from long noisy reads.

## When to use

Use when assembling a diploid genome from long noisy reads (Nanopore R9/R10, PacBio CLR) and the string graph contains heterozygous bubbles/branches that must be split into primary and alternate haplotype contigs.

## When NOT to use

- Input is high-accuracy PacBio HiFi or Nanopore duplex reads — the article reports PECAT achieves mediocre performance on these data types
- The organism is haploid or near-clonal with very low heterozygosity — SNP calling accuracy is low in low heterozygosity rate regions and error correction may not distinguish errors from heterozygote alleles
- Available data coverage is low — PECAT needs higher coverage of data to obtain a high-quality phased assembly

## Inputs

- Corrected long reads (Nanopore R9/R10 or PacBio CLR) in FASTA/FASTQ
- Round-one string graph overlaps among corrected reads (candidate overlaps from minimap2, e.g. -x ava-pb / -x ava-ont)
- Heterozygous SNP calls from corrected reads mapped to the first-round haplotype-collapsed assembly (strict: 8 SNP alleles)
- Raw-read SNP calls (loose: 6 SNP alleles; optionally Clair3 for Nanopore reads)
- PECAT config file with asm1_assemble_options, phase_filter_options, align_filter_options

## Outputs

- Haplotype-collapsed primary contigs (e.g. $PROJECT/3-assemble/primary.fasta)
- Two round-two diploid contig sets under $PROJECT/5-assemble: primary/alternate contig files and dual haplotype contig files in FASTA format

## How to apply

First build the string graph from corrected-read overlaps: extend minimap2 candidate overlaps to read ends with the diff algorithm, filter long overhangs, mark transitive edges with Myers' algorithm, and remove edges whose identity falls below an adaptive threshold. Score best-overlap-graph edges as s_i = (n_i+ * w - n_i-, l_i), balancing supporting versus conflicting read counts against overlap length, and assemble haplotype-collapsed contigs. Then reassemble in a second round reusing round-one overlaps: remove inconsistent overlaps identified by heterozygous SNP checks using a strict threshold of 8 different SNP alleles on corrected reads and a loose threshold of 6 different SNP alleles on raw reads (SNPs called by mapping corrected reads to the round-one assembly with minimap2, and optionally Clair3 for raw Nanopore reads). Finally, apply the alternate-branch heuristic — a branch is emitted as alternate when more than 30% of reads in the shorter branch are inconsistent with the reads in the other branch — producing both haplotype-collapsed (primary) and phased contig files.

## Related tools

- **PECAT** (Two-round string-graph assembler that scores best-overlap-graph edges, removes SNP-inconsistent overlaps, and emits primary/alternate haplotype contigs) — https://github.com/lemene/PECAT
- **minimap2** (Detects candidate read-to-read overlaps (ava-pb/ava-ont presets) and maps corrected reads to the first-round assembly for heterozygous SNP calling) — https://github.com/lh3/minimap2
- **Clair3** (Calls SNPs on raw Nanopore reads for the loose-threshold inconsistent-overlap check) — https://github.com/HKU-BAL/Clair3
- **Medaka** (Optional downstream polishing to further improve assembly quality after haplotype bubble resolution) — https://github.com/nanoporetech/medaka
- **Pomoxis** (Reference-based evaluation of the resolved contigs (e.g. qstat assembly QC)) — https://github.com/nanoporetech/pomoxis

## Examples

```
pecat.pl assemble cfgfile
```

## Evaluation signals

- The demo run produces a valid FASTA at $PROJECT/3-assemble/primary.fasta (haplotype-collapsed round-one assembly)
- Round-two contig files appear under $PROJECT/5-assemble with both primary/alternate and dual haplotype FASTA sets present
- Every removed inconsistent overlap is justified: the read pair carries at least 8 different SNP alleles on corrected reads (strict) or 6 different SNP alleles on raw reads (loose)
- Alternate branches were emitted only where more than 30% of reads in the shorter branch conflict with the other branch — no branch is phased on weaker evidence
- Haplotype-resolved quality is reflected in phase block NG50 metrics (e.g. 59.4/58.0 Mb for HG002 Nanopore R10), verifiable with hapdiff/SV comparison against a curated truth set

## Limitations

- SNP calling accuracy is low in low heterozygosity rate regions, weakening the inconsistent-overlap signal used for bubble resolution
- Error correction may not effectively distinguish small sequencing errors from heterozygote alleles in very low heterozygosity regions, even with highly accurate reads
- PECAT needs higher data coverage for a high-quality phased assembly
- Assembly parameters are only fully described in Supplementary Notes 4 and 5, not the main text; some issues are deferred to subsequent PECAT releases

## Evidence

- [discussion] Alternate branch requires >30% of reads in shorter branch inconsistent with other branch: "More than 30% of reads in the shorter branch are inconsistent with the reads in the other branch."
- [discussion] Inconsistent overlap detection thresholds on corrected and raw reads: "using corrected reads with the strict threshold that the pair of reads should contain 8 different SNP alleles"
- [discussion] Two-round assembly reuses round-one overlaps: "reuse the overlaps built in the first round of assembly and just simplify the overlaps by removing inconsistent ones before reassembling the diploid genome"
- [abstract] Combining corrected- and raw-read SNP callers: "We combine a corrected read SNP caller and a raw read SNP caller to further improve the identification of inconsistent overlaps in the string graph."
- [readme] PECAT is a phased error correction and assembly tool: "PECAT is a phased error correction and assembly tool for long reads. It includes a haplotype-aware correction method and an efficient diploid assembly method."

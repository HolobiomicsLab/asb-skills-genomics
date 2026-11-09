---
name: homopolymer-context-variant-filtering
description: Use when genotyping targeted InDel sites from long-read (e.g., PacBio
  HiFi) BAM alignments in MacroHapCaller and candidates must be filtered by homopolymer
  context and insert base quality before reporting calls.
when_to_use_negative:
- 'Discovering de novo InDels genome-wide: MacroHapCaller restricts InDel calls to
  the sites listed in the BED configure file, so unlisted sites are never called'
- 'Calling STR/SNP variants: those variant classes use separate configure files (-a
  strAnchorFile and -n snpPanelPosFile), not the InDel BED file and its homopolymer/quality
  filters'
- When candidate variants legitimately reside inside homopolymers (length >= homopolymerLen
  threshold, default 4) — they will be skipped by design
edam_operation: http://edamontology.org/operation_3227
tools:
- name: MacroHapCaller
  role: Java tool whose InDel calling module implements the homopolymer-exclusion
    (homopolymerLen, default 4) and Phred quality (LowQualThreshold, default 15) filters
    over CIGAR I/D operations at BED-configured sites
  repo: github.com/Ge-Lab/MacroHapCaller
- name: Java
  role: Runtime/implementation language; the filter parameters are passed as command-line
    flags to the MacroHapCaller jar
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:d5abbc943d7e4b1320dd1b5f5fe490e95f9a90d94b80d47150e1116743cc61ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MacroHapCaller
  edam_operation: http://edamontology.org/operation_3227
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# homopolymer-context-variant-filtering

## Summary

A filtering step in MacroHapCaller's InDel calling that discards candidate InDels falling inside homopolymer regions (homopolymerLen threshold, default 4) and variant bases with average Phred quality at or below Q15 (LowQualThreshold, default 15), restricting calls to BED-configured target sites.

## When to use

Use when genotyping targeted InDel sites from long-read (e.g., PacBio HiFi) BAM alignments in MacroHapCaller and candidates must be filtered by homopolymer context and insert base quality before reporting calls.

## When NOT to use

- Discovering de novo InDels genome-wide: MacroHapCaller restricts InDel calls to the sites listed in the BED configure file, so unlisted sites are never called
- Calling STR/SNP variants: those variant classes use separate configure files (-a strAnchorFile and -n snpPanelPosFile), not the InDel BED file and its homopolymer/quality filters
- When candidate variants legitimately reside inside homopolymers (length >= homopolymerLen threshold, default 4) — they will be skipped by design

## Inputs

- Read-reference alignment in BAM format (e.g., PacBio HiFi reads aligned to GRCh38/GCA_000001405.15)
- InDel configure file in BED format, tabular plain text (Chrom, ChromStart, ChromEnd, REF, ALT; 0-based coordinates), passed via -d/--inDelBedFile
- Genome reference sequence file in .fasta format (e.g., GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta), passed via -r/--refGenomeFastaFile

## Outputs

- InDel call table in tabular .tsv format (indel_calls.tsv) with one entry per configured InDel site per sample and consistent site counts across samples
- InDel genotypes integrated into the phased macrohaplotype strings produced by MacroHapCaller

## How to apply

Parse the BED-format InDel configure file (Chrom, ChromStart, ChromEnd, REF, ALT; 0-based coordinates) and load the read-reference alignment BAM plus the genome reference FASTA. Scan each aligned read's CIGAR string for I and D operations overlapping the configured sites to extract candidate InDel alleles. Discard candidates located inside homopolymer regions, using the homopolymerLen parameter (default 4) supplied via the -l flag. Filter variant bases by the Phred-scaled LowQualThreshold (default 15) supplied via the -q flag, requiring average insert base quality above Q15. Only sites listed in the BED file are called; for listed sites lacking an InDel in a sample, fill in the corresponding reference bases so all samples share the same number of sites.

## Related tools

- **MacroHapCaller** (Java tool whose InDel calling module implements the homopolymer-exclusion (homopolymerLen, default 4) and Phred quality (LowQualThreshold, default 15) filters over CIGAR I/D operations at BED-configured sites) — github.com/Ge-Lab/MacroHapCaller
- **Java** (Runtime/implementation language; the filter parameters are passed as command-line flags to the MacroHapCaller jar)

## Examples

```
java -jar -Xmx100G MacroHapCaller0.3.jar -i input.bam -o indel_calls.tsv -d MHindels_v0.2.bed -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 4 -q 15
```

## Evaluation signals

- Output indel_calls.tsv exists in .tsv format with one entry per configured InDel site per sample
- Site counts are consistent across all samples (reference bases are filled in at listed sites lacking an InDel)
- No reported InDel lies inside a homopolymer region of length >= homopolymerLen (default 4)
- All reported insert alleles have average base quality above the LowQualThreshold (Q15 by default)
- Rerunning with -l and -q toggled (e.g., -l 2 vs. default 4) changes the retained-call set in the expected direction

## Limitations

- Variants inside homopolymer regions at or above the homopolymerLen threshold (default 4) are unconditionally skipped, so true InDels in such contexts are missed
- Only BED-configured target sites are genotyped; the filter is a targeted-calling filter, not a genome-wide variant discovery filter
- The InDel module requires a reference FASTA and BAM alignment; per the article index, a citation/changelog for MacroHapCaller was not yet available

## Evidence

- [readme] integer, homopolymer length threshold to skip call variant,default [4]: "integer, homopolymer length threshold to skip call variant,default [4]"
- [methods] integer, phred-scale quality threshold for variant bases, default [15]: "integer, phred-scale quality threshold for variant bases, default [15]"
- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read: "calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read"

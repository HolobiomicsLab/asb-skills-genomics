---
name: cigar-indel-candidate-extraction
description: Use when calling targeted InDels from long-read (e.g. PacBio HiFi) BAM
  alignments at BED-defined loci, and you need per-site candidate InDel alleles with
  homopolymer and low-quality filtering for downstream macrohaplotype phasing.
when_to_use_negative:
- The variant of interest is an STR/SSR or SNP — MacroHapCaller handles those through
  separate configure files (-a/--strAnchorFile, -n/--snpPanelPosFile), not the InDel
  BED file
- Input is raw PacBio HiFi BAM rather than an indexed read-reference alignment — run
  the MH_umi_dedup_map_pipe.sh preprocessing pipeline (BAM to fastq, QC, alignment,
  sort, index) first
- You need de novo InDel discovery genome-wide — this skill only extracts candidates
  at sites listed in the BED configure file
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MacroHapCaller
  role: Java software whose InDel calling module scans CIGAR strings for I/D operations
    at BED-configured sites and applies homopolymerLen and LowQualThreshold filters
  repo: github.com/Ge-Lab/MacroHapCaller
- name: Java
  role: Programming language in which the InDel calling and filtering module is implemented
    and integrated into the MacroHapCaller codebase
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:7b4a820470eec75804759c0d873a785e01e3ae245915b2d029a3dd5c446503b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MacroHapCaller
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cigar-indel-candidate-extraction

## Summary

Extract candidate InDel alleles in MacroHapCaller by scanning each aligned read's CIGAR string for I and D operations overlapping BED-configured targeted sites, then filter candidates by homopolymer exclusion and Q15 insert-base quality before outputting an indel_calls.tsv table.

## When to use

Use when calling targeted InDels from long-read (e.g. PacBio HiFi) BAM alignments at BED-defined loci, and you need per-site candidate InDel alleles with homopolymer and low-quality filtering for downstream macrohaplotype phasing.

## When NOT to use

- The variant of interest is an STR/SSR or SNP — MacroHapCaller handles those through separate configure files (-a/--strAnchorFile, -n/--snpPanelPosFile), not the InDel BED file
- Input is raw PacBio HiFi BAM rather than an indexed read-reference alignment — run the MH_umi_dedup_map_pipe.sh preprocessing pipeline (BAM to fastq, QC, alignment, sort, index) first
- You need de novo InDel discovery genome-wide — this skill only extracts candidates at sites listed in the BED configure file

## Inputs

- Read-reference alignment BAM file (with index)
- BED-format InDel configure file (5 columns: Chrom, ChromStart, ChromEnd, REF, ALT; 0-based coordinates)
- Genome reference sequence file in .fasta format (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- indel_calls.tsv — tabular .tsv InDel call table with one entry per configured InDel site per sample and consistent site counts across samples
- Per-site candidate InDel alleles fed into MacroHapCaller's phased macrohaplotype strings

## How to apply

Parse the BED-format InDel configure file (5 columns: Chrom, ChromStart, ChromEnd, REF, ALT with 0-based coordinates) and load the read-reference alignment BAM plus the genome reference FASTA (MacroHapCaller options -d/--inDelBedFile, -i/--input, -r/--refGenomeFastaFile). Scan each aligned read's CIGAR string for I and D operations overlapping the configured sites to extract candidate InDel alleles. Discard candidates lying inside homopolymer regions using the homopolymerLen threshold (-l, default 4), and filter variant bases by the Phred-scaled LowQualThreshold (-q, default 15), requiring average insert base quality above Q15. Restrict calls to BED-listed sites; for listed sites lacking an InDel in a sample, fill in the corresponding reference bases so all samples share the same number of sites. Write the InDel call table in tabular .tsv format (indel_calls.tsv) for integration with MacroHapCaller's Java codebase.

## Related tools

- **MacroHapCaller** (Java software whose InDel calling module scans CIGAR strings for I/D operations at BED-configured sites and applies homopolymerLen and LowQualThreshold filters) — github.com/Ge-Lab/MacroHapCaller
- **Java** (Programming language in which the InDel calling and filtering module is implemented and integrated into the MacroHapCaller codebase)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i sample.bam -o indel_calls.tsv -d MHindels_v0.2.bed -r GRCh38.fasta -l 4 -q 15 -t 12
```

## Evaluation signals

- indel_calls.tsv exists in .tsv format after running on the test alignment
- Output contains one entry per configured InDel site per sample
- Site counts are consistent across all samples (reference bases filled in at sites lacking an InDel)
- No reported InDel lies inside a homopolymer region longer than the homopolymerLen threshold (default 4)
- Reported insert variants have average base quality above the LowQualThreshold (Q15 default)

## Limitations

- Candidates inside homopolymer regions (default threshold 4 bp) are skipped, so true InDels in homopolymers are not called
- Insert variants with average base quality at or below Q15 are discarded as low quality
- Only BED-configured targeted sites are called; the module cannot discover InDels outside the configured loci

## Evidence

- [readme] homopolymer exclusion: "-l,--homopolymerLen   integer, homopolymer length threshold to skip call variant,default [4]"
- [readme] quality filter: [evidence span withheld — anchor status: not_found]
- [readme] InDel BED input: [evidence span withheld — anchor status: not_found]
- [intro] scope of caller: "The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read"
- [methods] reference genome input: "GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta"

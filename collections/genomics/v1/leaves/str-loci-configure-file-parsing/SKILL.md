---
name: str-loci-configure-file-parsing
description: Use when preparing, checking, or supplying the '-a/--strAnchorFile' anchor
  configuration for MacroHapCaller targeted STR genotyping from PacBio HiFi BAM alignments,
  e.g. for forensic CODIS loci on GRCh38.
when_to_use_negative:
- If the locus file is a USAT-style BED file (columns Chrom, ChromStart, ChromEnd,
  Name, Left_offset, Right_offset, Basic_motif_period, Ref_hap_length, Motif, Ref_allele,
  Inner_offset, Min_stutter_threshold) — that format serves USAT haplotype comparison,
  not MacroHapCaller anchor calling
- If the goal is SNP (-n snpPanelPosFile) or InDel (-d inDelBedFile) configuration
  parsing — those use different file formats specified separately
- If reads are short paired-end NGS rather than long PacBio HiFi reads; MacroHapCaller
  is designed for long-read macrohaplotype calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: MacroHapCaller
  role: Consumes the parsed STR anchor configure file via -a/--strAnchorFile to drive
    anchor-based targeted STR genotyping and macrohaplotype phasing
  repo: github.com/Ge-Lab/MacroHapCaller
- name: USAT
  role: Downstream toolkit that instead consumes a BED-format TR locus file for haplotype
    sequence comparison and allele-size visualization
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:c35a192aad2c406b3ab0955e0bc1746d25528ec16fc52e1603e19382f1bcf61e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# str-loci-configure-file-parsing

## Summary

Parses and validates MacroHapCaller's tab-separated STR anchor configure file (one fixed headline, 11 columns per STR locus, e.g. the CODIS example CODISSTR_anchor.XW.config_v0.2.txt) so targeted STR loci can be genotyped with anchor-based calling.

## When to use

Use when preparing, checking, or supplying the '-a/--strAnchorFile' anchor configuration for MacroHapCaller targeted STR genotyping from PacBio HiFi BAM alignments, e.g. for forensic CODIS loci on GRCh38.

## When NOT to use

- If the locus file is a USAT-style BED file (columns Chrom, ChromStart, ChromEnd, Name, Left_offset, Right_offset, Basic_motif_period, Ref_hap_length, Motif, Ref_allele, Inner_offset, Min_stutter_threshold) — that format serves USAT haplotype comparison, not MacroHapCaller anchor calling
- If the goal is SNP (-n snpPanelPosFile) or InDel (-d inDelBedFile) configuration parsing — those use different file formats specified separately
- If reads are short paired-end NGS rather than long PacBio HiFi reads; MacroHapCaller is designed for long-read macrohaplotype calling

## Inputs

- Tab-separated STR anchor configure file with one fixed headline and 11 columns per STR locus (e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- Read-reference alignment BAM with index (e.g. HG002 8-kb PacBio HiFi reads aligned to GRCh38)
- Reference genome FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- Parsed per-locus anchor/motif table consumed by MacroHapCaller's -a option
- Downstream macrohaplotype .tsv allele table (marker name, counts, hapVar) for the targeted STR loci

## How to apply

Load the STR anchor configure file as a tab-separated plain-text file with exactly one fixed headline followed by one line per STR locus and 11 columns per line (Chrom, ChromStartPos_Str, ChromEndPos_Str, Name, Repeat_unit_length, Motif_must_present, Inner_offset, Anchor_left, Anchor_right, LeftAnchorStartPos, RightAnchorEndPos/MaxRefLength per the paper's schema), with integer coordinates starting from 1 (1-based, unlike BED-style 0-based files). Each locus defines left/right anchor sequences that MacroHapCaller matches within reads by edit distance, enforcing MaxMismatches default [1]; the Motif_must_present column supplies the repeat unit used to validate the inter-anchor sequence. Pass the file via the -a option of MacroHapCaller0.3.jar together with the input BAM, the reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta), InDel BED (-d) and SNP position (-n) files. Correctness is judged by whether the caller reproduces the macrohaplotype .tsv output format (e.g. the D2S441 locus on HG002 8-kb PacBio HiFi reads) with allele calls agreeing with the CODIS ground truth.

## Related tools

- **MacroHapCaller** (Consumes the parsed STR anchor configure file via -a/--strAnchorFile to drive anchor-based targeted STR genotyping and macrohaplotype phasing) — github.com/Ge-Lab/MacroHapCaller
- **USAT** (Downstream toolkit that instead consumes a BED-format TR locus file for haplotype sequence comparison and allele-size visualization) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- The file has exactly one fixed headline and 11 tab-separated columns per STR locus line, with coordinates starting from 1
- Anchor matching at each locus succeeds within MaxMismatches default [1] edit distance; loci whose anchors cannot be matched are flagged rather than silently genotyped
- The Motif_must_present repeat unit matches the observed inter-anchor read sequence; homopolymers at or above homopolymerLen default [4] are skipped
- Reported macrohaplotypes pass MinReadCount default [2] and MinReadProportion default [0.001], with variant bases meeting LowQualThreshold default [15]
- Output .tsv matches the MacroHapCaller macrohaplotype format shown for the D2S441 locus on HG002 8-kb PacBio HiFi reads and agrees with CODIS ground-truth alleles

## Limitations

- The anchor configuration schema is MacroHapCaller-specific; USAT expects a different BED-style locus file, so files are not interchangeable
- No changelog is available for MacroHapCaller, and the paper's methods note 'Citation coming soon', limiting version-specific documentation
- If the Motif_must_present or anchor columns are wrong for a locus, reads fail anchor matching or motif validation and that locus is skipped from variant calling

## Evidence

- [methods] STR anchor configure file format: "The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR of human genome is given"
- [readme] 11-column schema with fixed headline: "Each line for one STR locus. There is one fixed headline before starting the detailed STR locus. Every line has 11 columns."
- [methods] Anchor match tolerance parameter: "integer, maximum distance of edit distance in anchor match, default [1]"
- [readme] CLI option for the STR anchor file: [evidence span withheld — anchor status: not_found]

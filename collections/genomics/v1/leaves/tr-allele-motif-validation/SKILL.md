---
name: tr-allele-motif-validation
description: Use when genotyping targeted STR/TR loci from long PacBio HiFi reads
  with a BAM read-reference alignment and an STR anchor configure file, and you must
  locate TR locus boundaries, validate repeat motifs, and report phased haplotype
  alleles.
when_to_use_negative:
- Input is short-read sequencing data — MacroHapCaller is designed for long NGS reads,
  especially PacBio HiFi reads, where anchors span the locus
- The TR loci lack known flanking anchor sequences or a Motif_must_present repeat
  unit — the anchor match and motif validation steps cannot be configured
- The goal is genome-wide de novo STR discovery rather than genotyping a panel of
  targeted loci listed in the anchor configure file
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MacroHapCaller
  role: 'Java program that performs the anchor-based TR calling: extracts read sequence
    adjacent to each targeted TR locus, matches left/right anchors by edit distance,
    validates the motif, and writes the phased TR-allele .tsv'
  repo: https://github.com/XuewenWangUGA/USAT
- name: PacBio HiFi
  role: Source of high-quality long reads (~8-kb amplicons harboring 20 core forensic
    STR loci) over which the anchor-based caller is applied
- name: Python3
  role: Part of the MacroHapCaller pipeline that preprocesses BAM/fastq reads, aligns
    them to the genome, and sorts/indexes the alignment before TR calling
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:e3a51b9aa368a22c42da26d787a031dfd0871b6c50ed84649094253193e03999
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XuewenWangUGA/USAT
  tools:
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# tr-allele-motif-validation

## Summary

Anchor-based tandem repeat (STR) allele calling in MacroHapCaller: sliding-window anchor matching by edit distance around each targeted TR locus, followed by motif validation of the inter-anchor sequence and read-support filtering to produce a phased TR-allele table.

## When to use

Use when genotyping targeted STR/TR loci from long PacBio HiFi reads with a BAM read-reference alignment and an STR anchor configure file, and you must locate TR locus boundaries, validate repeat motifs, and report phased haplotype alleles.

## When NOT to use

- Input is short-read sequencing data — MacroHapCaller is designed for long NGS reads, especially PacBio HiFi reads, where anchors span the locus
- The TR loci lack known flanking anchor sequences or a Motif_must_present repeat unit — the anchor match and motif validation steps cannot be configured
- The goal is genome-wide de novo STR discovery rather than genotyping a panel of targeted loci listed in the anchor configure file

## Inputs

- Read-reference alignment file in BAM format (with BAM index), e.g. from MH_umi_dedup_map_pipe.sh
- STR anchor configure file in tab-separated plain text with one fixed headline and 11 columns per STR locus (e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- Genome reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- TR-allele table in .tsv macrohaplotype output format (total hapVar header, marker name, counts, hapVar) matching the MacroHapCaller output shown for the D2S441 locus on HG002 8-kb PacBio HiFi reads

## How to apply

Load the STR anchor configure file — a tab-separated plain-text file with one fixed headline and 11 columns per STR locus (Chrom, ChromStartPos_Str, ChromEndPos_Str, Name, Repeat_unit_length, Motif_must_present, Inner_offset, Anchor_left, Anchor_right, LeftAnchorStartPos, RightAnchorEndPos, MaxRefLength), coordinates starting from 1 (e.g. CODISSTR_anchor.XW.config_v0.2.txt). Iterate over reads in the input BAM and extract the read sequence adjacent to each targeted TR locus bounded by the left and right anchor coordinates; in each loop a 30 bp (default) chunk adjacent to the locus is screened for an anchor. Slide an anchor-length window across the chunk and locate the best left and right anchor matches by edit distance, enforcing MaxMismatches default [1]. Extract the inter-anchor sequence, validate it against the Motif_must_present repeat unit from the configure file, and skip variant calling at homopolymers at or above homopolymerLen default [4]. Finally tally supported reads per allele applying LowQualThreshold default [15] on base quality, MinReadCount default [2], and MinReadProportion default [0.001] before writing the macrohaplotype output.

## Related tools

- **MacroHapCaller** (Java program that performs the anchor-based TR calling: extracts read sequence adjacent to each targeted TR locus, matches left/right anchors by edit distance, validates the motif, and writes the phased TR-allele .tsv) — https://github.com/XuewenWangUGA/USAT
- **PacBio HiFi** (Source of high-quality long reads (~8-kb amplicons harboring 20 core forensic STR loci) over which the anchor-based caller is applied)
- **Python3** (Part of the MacroHapCaller pipeline that preprocesses BAM/fastq reads, aligns them to the genome, and sorts/indexes the alignment before TR calling) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- The produced TR-allele table matches the MacroHapCaller output format shown for the D2S441 locus on HG002 8-kb PacBio HiFi reads
- Allele calls agree with the ground truth used in the paper for CODIS STR loci in benchmark samples HG002 and HG003
- Anchor matches respect the MaxMismatches edit-distance threshold (default [1]); loci failing anchor matching are skipped rather than miscalled
- Homopolymer regions at or above homopolymerLen (default [4]) are skipped, and reported macrohaplotypes satisfy MinReadCount [2] and MinReadProportion [0.001]

## Limitations

- Variant calling is skipped at homopolymers with length at or above the homopolymerLen threshold (default [4]), so alleles dominated by long homopolymers may not be genotyped
- Loci where the anchor sequence is not found within MaxMismatches (default [1]) edit distance of the chunk sequence cannot be called
- Reported macrohaplotypes depend on read support thresholds (MinReadCount [2], MinReadProportion [0.001]); rare alleles in mixtures may be filtered out

## Evidence

- [methods] During TR calling, each targeted TR site is screened in a loop in which a chunk sequence (30 bp by default) of the read adjacent to the targeted TR locus is extracted to check for the presence of an anchor sequence.: [evidence span withheld — anchor status: not_found]
- [readme] The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR of human genome is given in "CODISSTR_anchor.XW.config_v0.2.txt": "The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR"
- [readme] integer, maximum distance of edit distance in anchor match, default [1]: "-m,--MaxMismatches   integer, maximum distance of edit distance in anchor match, default [1]"
- [abstract] Robust and accurate genotyping and phased macrohaplotypes were obtained with MacroHapCaller compared with the known ground truth: "Robust and accurate genotyping and phased macrohaplotypes were obtained with MacroHapCaller compared with the known ground truth"
- [abstract] High-quality long-sequencing reads, for example, PacBio HiFi reads, provide data to detect macrohaplotypes in multiploidy and DNA mixtures: "High-quality long-sequencing reads, for example, PacBio HiFi reads, provide data to detect macrohaplotypes in multiploidy"

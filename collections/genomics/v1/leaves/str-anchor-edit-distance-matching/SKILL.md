---
name: str-anchor-edit-distance-matching
description: Use when genotyping targeted STR/TR loci (e.g., forensic CODIS markers)
  from long PacBio HiFi reads in a BAM and you need anchor-guided boundary detection
  plus motif-validated allele calling rather than de novo TR discovery.
when_to_use_negative:
- Input is raw PacBio HiFi BAM or fastq reads that have not yet been aligned, sorted,
  and indexed to a reference genome — run the MH_umi_dedup_map_pipe.sh preprocessing
  pipeline first
- The goal is de novo discovery of unknown tandem repeat loci — the algorithm requires
  pre-configured anchor coordinates and Motif_must_present repeat units per targeted
  locus
- Targeted variant classes are only SNPs or InDels without an STR component — use
  the SNP panel (-n) or InDel BED (-d) configure files instead of the STR anchor workflow
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MacroHapCaller
  role: 'Implements the anchor-based TR calling: extracts 30-bp chunks adjacent to
    targeted loci, matches anchors by edit distance (MaxMismatches default 1), validates
    the Motif_must_present unit, and writes the macrohaplotype .tsv'
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: Python3
  role: Used in the pipeline steps (BAM/fastq input, alignment, sort, index) that
    generate the read-reference BAM consumed by the anchor-based caller
- name: USAT
  role: 'Downstream comparison of the haplotype sequences produced by the caller:
    allele length-to-repeat-count calculation and haplotype sequence alignment/comparison'
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:e35292c921c74ec4b4125366a7489921dd06e29ba6665fbe84f6fe93788fb5c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-Lab/MacroHapCaller
  tools:
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  - USAT
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# str-anchor-edit-distance-matching

## Summary

Anchored tandem-repeat (STR) allele calling in MacroHapCaller: for each targeted STR locus in a read-reference BAM, flanking chunk sequences are screened for left/right anchor matches by edit distance, the inter-anchor read sequence is extracted and validated against the configured repeat motif, and read support is tallied into a macrohaplotype table.

## When to use

Use when genotyping targeted STR/TR loci (e.g., forensic CODIS markers) from long PacBio HiFi reads in a BAM and you need anchor-guided boundary detection plus motif-validated allele calling rather than de novo TR discovery.

## When NOT to use

- Input is raw PacBio HiFi BAM or fastq reads that have not yet been aligned, sorted, and indexed to a reference genome — run the MH_umi_dedup_map_pipe.sh preprocessing pipeline first
- The goal is de novo discovery of unknown tandem repeat loci — the algorithm requires pre-configured anchor coordinates and Motif_must_present repeat units per targeted locus
- Targeted variant classes are only SNPs or InDels without an STR component — use the SNP panel (-n) or InDel BED (-d) configure files instead of the STR anchor workflow

## Inputs

- Read-reference alignment file in BAM format (from MH_umi_dedup_map_pipe.sh or user-generated), with BAM index
- STR anchor configure file in tab-separated plain text with one fixed headline and 11 columns per STR locus (e.g., CODISSTR_anchor.XW.config_v0.2.txt)
- Genome reference sequence file in FASTA format (e.g., GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- Macrohaplotype result file in .tsv format (TR-allele table with total hapVar header, marker name, counts, hapVar per locus)

## How to apply

Load the STR anchor configure file (a tab-separated text file with one fixed headline and 11 columns per locus, e.g., Chrom, ChromStartPos_Str, ChromEndPos_Str, Name, Repeat_unit_length, Motif_must_present, Inner_offset, Anchor_left, Anchor_right, LeftAnchorStartPos, RightAnchorEndPos, MaxRefLength, coordinates starting from 1), then iterate over reads in the BAM and extract the ~30-bp default chunk of read sequence adjacent to each targeted TR locus. Slide an anchor-length window across the extracted chunk and locate the best left and right anchor matches by edit distance, enforcing MaxMismatches (default 1); the rationale is that anchors localize TR boundaries robustly in long reads where alignment drift would blur repeat starts. Extract the inter-anchor sequence, validate it against the Motif_must_present repeat unit from the configure file, and skip variant calling at homopolymers at or above homopolymerLen (default 4) to avoid polymerase-noise artifacts. Tally read support per allele applying LowQualThreshold (default 15, phred scale) on base quality, MinReadCount (default 2), and MinReadProportion (default 0.001) before reporting a macrohaplotype. Validation is done by comparing the produced TR-allele table in the .tsv macrohaplotype output format against ground truth allele calls, as demonstrated for the D2S441 locus on HG002 8-kb PacBio HiFi reads.

## Related tools

- **MacroHapCaller** (Implements the anchor-based TR calling: extracts 30-bp chunks adjacent to targeted loci, matches anchors by edit distance (MaxMismatches default 1), validates the Motif_must_present unit, and writes the macrohaplotype .tsv) — https://github.com/Ge-Lab/MacroHapCaller
- **Python3** (Used in the pipeline steps (BAM/fastq input, alignment, sort, index) that generate the read-reference BAM consumed by the anchor-based caller)
- **USAT** (Downstream comparison of the haplotype sequences produced by the caller: allele length-to-repeat-count calculation and haplotype sequence alignment/comparison) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- The produced TR-allele table matches the MacroHapCaller output format shown for the D2S441 locus on HG002 8-kb PacBio HiFi reads (total hapVar header, marker name, counts, hapVar)
- Allele calls agree with the ground truth used in the paper for CODIS STR loci in HG002/HG003
- Every anchor match obeys the edit-distance limit (-m/-MaxMismatches, default 1) and inter-anchor sequences carry the configured Motif_must_present unit
- Reported macrohaplotypes pass the support filters: base quality ≥ LowQualThreshold 15, ≥ MinReadCount 2 reads, and ≥ MinReadProportion 0.001 of reads at the locus
- Homopolymer regions ≥ homopolymerLen (default 4) are excluded from variant calling rather than genotyped

## Limitations

- Anchor matching tolerates at most MaxMismatches (default 1) edit distance, so loci with diverged or mutated anchors may fail boundary detection
- Homopolymer regions at or above homopolymerLen (default 4) are skipped for variant calling, leaving those variants unreported
- Requires per-locus configuration (11-column anchor file with motif and coordinates, 1-based), so new STR loci must be manually curated before calling

## Evidence

- [methods] a chunk sequence (30 bp by default) of the read adjacent to the targeted TR locus is extracted to check for the presence of an anchor sequence: [evidence span withheld — anchor status: not_found]
- [methods] Slide an anchor-length window across the extracted chunk and locate the best left and right anchor matches by edit distance, enforcing MaxMismatches default [1]: [evidence span withheld — anchor status: not_found]
- [methods] validate it against the Motif_must_present repeat unit from the configure file, and skip variant calling at homopolymers at or above homopolymerLen default [4]: [evidence span withheld — anchor status: not_found]
- [readme] integer, maximum distance of edit distance in anchor match, default [1]: "**-m,--MaxMismatches**  &nbsp; integer, maximum distance of edit distance in anchor match, default [1]"
- [readme] Each line for one STR locus. There is one fixed headline before starting the detailed STR locus. Every line has 11 columns.: "Each line for one STR locus. There is one fixed headline before starting the detailed STR locus. Every line has 11 columns."
- [readme] java -jar -Xmx120G MacroHapCaller0.3.jar -i /mnt/data0/xuewen/macrohaplotype/hg002/Q40hifi/hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv: "java -jar -Xmx120G MacroHapCaller0.3.jar -i /mnt/data0/xuewen/macrohaplotype/hg002/Q40hifi/hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv"
- [abstract] MacroHapCaller achieved a higher or consistent genotyping accuracy and faster speed than existing tools HipSTR and DeepVar: "MacroHapCaller achieved a higher or consistent genotyping accuracy and faster speed than existing tools HipSTR and DeepVar"

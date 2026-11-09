---
name: read-backed-haplotype-phasing
description: Use when long reads (e.g., PacBio HiFi BAM alignments) span targeted
  STR/SNP/InDel loci and you need phased macrohaplotypes rather than per-locus genotypes,
  e.g., for forensics, polyploids, or DNA mixtures.
when_to_use_negative:
- Variants (private mutations) absent from the user-supplied STR/InDel/SNP configure
  files — the current version discards them, so the skill cannot report untargeted
  alleles
- Short-read NGS data where reads do not span multiple targeted loci, since physical
  read-backed phasing requires a single read to carry the co-located alleles
- Haplotype comparison, alignment, or allele-size conversion tasks — use USAT downstream,
  which consumes the macrohaplotype output rather than producing it
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_2885
tools:
- name: MacroHapCaller
  role: Java program that calls targeted STRs, SNPs, and InDels per long read, concatenates
    them into ';' separated macrohaplotype strings, clusters them by read support,
    and applies MinReadCount/MinReadProportion filtering
  repo: github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Shell pipeline that preprocesses original PacBio HiFi BAM to fastq, performs
    QC, optional UMI analysis, read-reference alignment, sort, and index to produce
    the input alignment BAM
- name: USAT
  role: Downstream Java GUI toolkit that takes the haplotype output for TR allele
    comparison, plotting, and sequence alignment visualization
  repo: https://github.com/XuewenWangUGA/USAT
- name: HipSTR
  role: 'Comparison tool: MacroHapCaller achieved higher or consistent genotyping
    accuracy and faster speed than HipSTR'
- name: DeepVar
  role: 'Comparison tool: MacroHapCaller achieved higher or consistent genotyping
    accuracy and faster speed than DeepVar'
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:0cee626a3af5db0080284583b7fd98c04635990c4341e3a9f6990c3d559f5ee9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepVar
  - HipSTR
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - USAT
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# read-backed-haplotype-phasing

## Summary

Phase targeted STR, SNP, and InDel alleles called from individual PacBio HiFi long reads into macrohaplotype strings using physical (read-backed) phasing, enabling multi-allelic haplotype detection in diploid, polyploid, and mixed DNA samples.

## When to use

Use when long reads (e.g., PacBio HiFi BAM alignments) span targeted STR/SNP/InDel loci and you need phased macrohaplotypes rather than per-locus genotypes, e.g., for forensics, polyploids, or DNA mixtures.

## When NOT to use

- Variants (private mutations) absent from the user-supplied STR/InDel/SNP configure files — the current version discards them, so the skill cannot report untargeted alleles
- Short-read NGS data where reads do not span multiple targeted loci, since physical read-backed phasing requires a single read to carry the co-located alleles
- Haplotype comparison, alignment, or allele-size conversion tasks — use USAT downstream, which consumes the macrohaplotype output rather than producing it

## Inputs

- Read-reference alignment BAM (generated via MH_umi_dedup_map_pipe.sh or user-generated, with BAM index)
- Genome reference fasta (e.g., GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)
- STR anchor configure file in tabular plain text (e.g., CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (e.g., MHindels_v0.2.bed)
- SNP panel position file in tabular plain text (e.g., MHsnps.pos.txt)

## Outputs

- Tab-separated .tsv macrohaplotype table: total hapVar line, a Position line listing targeted variant coordinates, and macrohaplotype rows with marker name, supporting read counts, hapVar length, and haplotype string with SNP/InDel/STR parts separated by ';'

## How to apply

Match reads to each targeted locus using the STR anchor configure file with a maximum anchor edit distance of 1 (-m, MaxMismatches default 1), then call the targeted SNP, InDel, and STR alleles per read, skipping homopolymers of length >= 4 (-l, homopolymerLen default 4) and variant bases with Phred quality below 15 (-q, LowQualThreshold default 15). Concatenate the SNP, InDel, and STR allele parts from each single read into one macrohaplotype string with the three parts separated by ';' so variants that co-occur on the same physical read are phased together. Cluster identical macrohaplotype strings per locus, count supporting reads, and filter clusters below the minimum read count (-c, MinReadCount default 2) and minimum read proportion (-p, MinReadProportion default 0.001) to suppress noisy or stutter-derived calls. Rank surviving macrohaplotypes by supporting-read count and write them to the output .tsv via -o; the physical phasing basis allows detection of multi-allelic macrohaplotypes that pairwise-statistical phasing would miss.

## Related tools

- **MacroHapCaller** (Java program that calls targeted STRs, SNPs, and InDels per long read, concatenates them into ';' separated macrohaplotype strings, clusters them by read support, and applies MinReadCount/MinReadProportion filtering) — github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Shell pipeline that preprocesses original PacBio HiFi BAM to fastq, performs QC, optional UMI analysis, read-reference alignment, sort, and index to produce the input alignment BAM)
- **USAT** (Downstream Java GUI toolkit that takes the haplotype output for TR allele comparison, plotting, and sequence alignment visualization) — https://github.com/XuewenWangUGA/USAT
- **HipSTR** (Comparison tool: MacroHapCaller achieved higher or consistent genotyping accuracy and faster speed than HipSTR)
- **DeepVar** (Comparison tool: MacroHapCaller achieved higher or consistent genotyping accuracy and faster speed than DeepVar)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- Output .tsv follows the documented layout: first line with total hapVar reads per locus, a Position line listing targeted variant coordinates, then macrohaplotype rows with support counts
- Reported haplotypes match known ground-truth phased alleles for benchmark samples (HG002, HG003 at 20 core forensic STR loci)
- Macrohaplotype strings contain three ';'-separated parts corresponding to SNP, InDel, and STR alleles
- All retained macrohaplotypes have supporting-read count >= MinReadCount (default 2) and read proportion >= MinReadProportion (default 0.001)
- No called InDel appears in the output unless it is present in the InDel configure file

## Limitations

- Private mutations absent from the configure files are not reported; a called InDel is discarded if not in the configure file (planned future extension)
- Homopolymers of length >= the homopolymerLen threshold (default 4) are skipped during variant calling
- Variant bases with Phred quality below LowQualThreshold (default 15) are excluded, and anchor matching tolerates at most MaxMismatches (default 1) edit distance

## Evidence

- [abstract] MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes"
- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings: "calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [methods] integer, minimum read count to report a macrohaplotype, default [2]: "integer, minimum read count to report a macrohaplotype, default [2]"
- [methods] double, minimum read proportion of all reads for each locus to report a macrohaplotype, default [0.001]: "double, minimum read proportion of all reads for each locus to report a macrohaplotype, default [0.001]"
- [methods] integer, phred-scale quality threshold for variant bases, default [15]: "integer, phred-scale quality threshold for variant bases, default [15]"

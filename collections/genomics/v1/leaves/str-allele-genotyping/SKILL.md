---
name: str-allele-genotyping
description: Use when analyzing PacBio HiFi or other long reads of targeted amplicons
  (e.g. ~8-kb forensic STR loci) to genotype STR/SNP/InDel alleles and produce read-backed
  phased macrohaplotypes per locus.
when_to_use_negative:
- Short NGS reads where read length cannot span the targeted STR and flanking SNPs/InDels
  — MacroHapCaller targets long reads such as PacBio HiFi
- Unaligned raw reads — reads must first be aligned, sorted, and indexed to the reference
  (via MH_umi_dedup_map_pipe.sh or equivalent) before calling
- The goal is downstream haplotype sequence comparison/visualization rather than genotyping
  — USAT is the companion tool that takes haplotype output for alignment, plotting,
  and allele-size calculation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MacroHapCaller
  role: Java-based macrohaplotype caller that simultaneously calls targeted STR/SNP/InDel
    alleles per single long read and clusters them into phased haplotype strings
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Preprocess pipeline converting original PacBio HiFi BAM to fastq, performing
    read QC, optional umi-analysis, statistics, and read-reference alignment to produce
    the input BAM
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: USAT
  role: Downstream graphic-interface toolkit that takes the haplotype output to compare
    haplotype sequences, plot allele distributions, and compute allele sizes in repeat
    units
  repo: https://github.com/XuewenWangUGA/USAT
- name: HipSTR
  role: Existing STR genotyper used as a benchmark comparison for accuracy and speed
- name: DeepVar
  role: Existing variant caller used as a benchmark comparison for accuracy and speed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:e71fd6d84262acbe6c34da3702601413a60b8d539660d22b37a854d8d93fb1af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-Lab/MacroHapCaller
  tools:
  - DeepVar
  - HipSTR
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# str-allele-genotyping

## Summary

Genotypes targeted STR/SNP/InDel alleles from long sequencing reads (e.g. PacBio HiFi) by calling each variant class per read, concatenating them into macrohaplotype strings, clustering by string identity, and filtering noisy calls with MinReadCount/MinReadProportion thresholds.

## When to use

Use when analyzing PacBio HiFi or other long reads of targeted amplicons (e.g. ~8-kb forensic STR loci) to genotype STR/SNP/InDel alleles and produce read-backed phased macrohaplotypes per locus.

## When NOT to use

- Short NGS reads where read length cannot span the targeted STR and flanking SNPs/InDels — MacroHapCaller targets long reads such as PacBio HiFi
- Unaligned raw reads — reads must first be aligned, sorted, and indexed to the reference (via MH_umi_dedup_map_pipe.sh or equivalent) before calling
- The goal is downstream haplotype sequence comparison/visualization rather than genotyping — USAT is the companion tool that takes haplotype output for alignment, plotting, and allele-size calculation

## Inputs

- Read-reference alignment file in BAM format (from MH_umi_dedup_map_pipe.sh or user-generated)
- STR anchor configure file (tabular plain text, e.g. CODISSTR_anchor.XW.config_v0.2.txt with 11 columns)
- InDel configure file in BED format (tabular plain text)
- SNP panel position configure file (tabular plain text)
- Genome reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- Macrohaplotype .tsv table (marker name, supporting-read counts, hapVar length, haplotype string with SNP/InDel/STR parts separated by ';', ranked by read count)

## How to apply

Load the read-reference alignment BAM plus three configure files (STR anchor table, InDel BED, SNP panel) and match reads to targeted loci using anchors with MaxMismatches edit distance default 1. For each matched read, call targeted SNP, InDel and STR alleles, skipping homopolymers of length >= homopolymerLen (default 4) and variant bases with Phred quality below LowQualThreshold (default 15). Concatenate each read's SNP/InDel/STR allele calls into one macrohaplotype string with the three parts separated by ';', then cluster identical strings and count supporting reads per locus. Retain clusters passing MinReadCount (default 2) and MinReadProportion of all locus reads (default 0.001), rank surviving macrohaplotypes by supporting-read count, and write them to a .tsv via -o. Physical (read-backed) phasing of the concatenated string allows multi-allelic macrohaplotype detection in diploid, polyploid, and DNA-mixture samples.

## Related tools

- **MacroHapCaller** (Java-based macrohaplotype caller that simultaneously calls targeted STR/SNP/InDel alleles per single long read and clusters them into phased haplotype strings) — https://github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Preprocess pipeline converting original PacBio HiFi BAM to fastq, performing read QC, optional umi-analysis, statistics, and read-reference alignment to produce the input BAM) — https://github.com/Ge-Lab/MacroHapCaller
- **USAT** (Downstream graphic-interface toolkit that takes the haplotype output to compare haplotype sequences, plot allele distributions, and compute allele sizes in repeat units) — https://github.com/XuewenWangUGA/USAT
- **HipSTR** (Existing STR genotyper used as a benchmark comparison for accuracy and speed)
- **DeepVar** (Existing variant caller used as a benchmark comparison for accuracy and speed)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GRCh38_no_alt_analysis_set.fasta -l 4 -m 1 -q 15 -c 2 -p 0.001 -t 12
```

## Evaluation signals

- Output .tsv follows the documented layout: a total hapVar line, a position line listing targeted variant coordinates, and macrohaplotype rows with support counts (e.g. the D2S441 example with 8646 and 7467 supporting reads)
- Reported haplotypes match ground-truth phased alleles for benchmark samples (HG002, HG003, 20 core forensic STR loci)
- Per-locus macrohaplotype clusters all satisfy MinReadCount >= 2 and MinReadProportion >= 0.001 (or user-set values such as -c 100 -p 0.01) of total locus reads
- Genotyping accuracy is higher than or consistent with HipSTR and DeepVar on the same data, and runtime stays near ~2 mins with 12 threads / 120G RAM for four full PacBio SMRTcell HiFi runs
- Each macrohaplotype string contains three ';'-separated parts corresponding to SNP, InDel, and STR alleles

## Limitations

- Homopolymers of length >= 4 (homopolymerLen default) are skipped during variant calling, potentially suppressing some STR complexity
- Anchor matching allows at most MaxMismatches edit distance (default 1), so reads with more divergent anchors fail to match targeted loci
- Variant bases below Phred quality 15 (LowQualThreshold default) are ignored, so low-quality genuine variants may be missed
- Validation reported genotyping accuracy 'higher or consistent' rather than universally superior to HipSTR/DeepVar; benchmarking covered ~8-kb amplicons of 20 core forensic STR loci and whole-genome long-read data

## Evidence

- [intro] MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings: "calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [methods] Filter thresholds for macrohaplotype reporting: "integer, minimum read count to report a macrohaplotype, default [2]"
- [methods] Read proportion filter parameter: "double, minimum read proportion of all reads for each locus to report a macrohaplotype, default [0.001]"
- [abstract] MacroHapCaller validated on targeted PacBio HiFi of 20 forensic STR loci in HG002 and HG003 with read-backed phasing: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample"
- [readme] CLI invocation showing input files and thresholds: "java -jar -Xmx120G MacroHapCaller0.3.jar -i /mnt/data0/xuewen/macrohaplotype/hg002/Q40hifi/hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv"
- [abstract] Benchmark comparison against HipSTR and DeepVar: "MacroHapCaller achieved a higher or consistent genotyping accuracy and faster speed than existing tools HipSTR and DeepVar"

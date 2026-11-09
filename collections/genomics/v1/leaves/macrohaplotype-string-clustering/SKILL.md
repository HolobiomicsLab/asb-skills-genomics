---
name: macrohaplotype-string-clustering
description: Use when you need to phase and genotype targeted STR/SNP/InDel macrohaplotypes
  from individual long reads (e.g., PacBio HiFi BAM), especially for diploid, polyploid,
  or forensic DNA-mixture samples where multi-allelic calls must be resolved.
when_to_use_negative:
- Input is short-read (Illumina) data or read lengths too short to span multiple targeted
  variant loci — read-backed phasing needs long reads such as PacBio HiFi
- Goal is de novo genome-wide variant discovery rather than genotyping a pre-specified
  panel of targeted STR/SNP/InDel loci defined in the configure files
- Only a single variant class is of interest and phased multi-locus strings are unnecessary
  — HipSTR or DeepVar may suffice (MacroHapCaller is built for simultaneous STR+SNP+InDel
  macrohaplotyping)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MacroHapCaller
  role: 'Primary tool: calls STR/SNP/InDel alleles per long read, concatenates them
    into macrohaplotype strings, clusters, filters, and outputs the ranked .tsv'
  repo: github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Preprocessing pipeline that converts PacBio HiFi BAM to fastq, performs read
    QC, optional UMI analysis, statistical summary, and read-reference alignment
- name: USAT
  role: Downstream graphical tool for comparing, aligning, and visualizing the resulting
    TR haplotype sequences
  repo: https://github.com/XuewenWangUGA/USAT
- name: Java
  role: Runtime environment; MacroHapCaller is programmed in Java with parallel computing
    enabled
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:f5e1d73c6fcf42b6141d76ac0afdcbc06866d9f71c67e69773b98760d42d13ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Java
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# macrohaplotype-string-clustering

## Summary

Concatenates per-read SNP, InDel, and STR allele calls from targeted long-read (PacBio HiFi) alignments into a single ';'-separated macrohaplotype string, clusters identical strings, and filters/ranks clusters by read support using read-backed phasing.

## When to use

Use when you need to phase and genotype targeted STR/SNP/InDel macrohaplotypes from individual long reads (e.g., PacBio HiFi BAM), especially for diploid, polyploid, or forensic DNA-mixture samples where multi-allelic calls must be resolved.

## When NOT to use

- Input is short-read (Illumina) data or read lengths too short to span multiple targeted variant loci — read-backed phasing needs long reads such as PacBio HiFi
- Goal is de novo genome-wide variant discovery rather than genotyping a pre-specified panel of targeted STR/SNP/InDel loci defined in the configure files
- Only a single variant class is of interest and phased multi-locus strings are unnecessary — HipSTR or DeepVar may suffice (MacroHapCaller is built for simultaneous STR+SNP+InDel macrohaplotyping)

## Inputs

- read-reference alignment BAM (with index), e.g. produced by MH_umi_dedup_map_pipe.sh from PacBio HiFi reads
- STR anchor configure file in tabular plain text (e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (e.g. MHindels_v0.2.bed)
- SNP panel position file in tabular plain text (e.g. MHsnps.pos.txt)
- genome reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- macrohaplotype .tsv table with marker name, support counts, hapVar length, and haplotype string containing SNP/InDel/STR parts, ranked by supporting-read count

## How to apply

Run MacroHapCaller (Java, e.g. v0.3) on a read-reference alignment BAM plus three configure files: an STR anchor table (-a), an InDel BED (-d), and an SNP panel (-n), with a reference FASTA (-r). Reads are matched to targeted loci using anchors with maximum edit distance 1 (MaxMismatches default [1]), and per-read variant calls skip homopolymers of length >= 4 (homopolymerLen) and variant bases with Phred quality below 15 (LowQualThreshold). For each read, the called SNP, InDel, and STR alleles are concatenated into one macrohaplotype string with the three parts separated by ';'. Identical strings are clustered per locus and support counts recorded; clusters are filtered by MinReadCount (default 2) and MinReadProportion of all reads for the locus (default 0.001), then ranked by supporting-read count and written with -o as a .tsv of marker name, counts, hapVar length, and haplotype string. Thresholds should be tuned per dataset — the README example uses stricter -c 100 and -p 0.01 — and parallel computing (-t, default 12 threads) keeps runtime to ~2 mins for four full PacBio SMRTcell HiFi runs.

## Related tools

- **MacroHapCaller** (Primary tool: calls STR/SNP/InDel alleles per long read, concatenates them into macrohaplotype strings, clusters, filters, and outputs the ranked .tsv) — github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Preprocessing pipeline that converts PacBio HiFi BAM to fastq, performs read QC, optional UMI analysis, statistical summary, and read-reference alignment)
- **USAT** (Downstream graphical tool for comparing, aligning, and visualizing the resulting TR haplotype sequences) — https://github.com/XuewenWangUGA/USAT
- **Java** (Runtime environment; MacroHapCaller is programmed in Java with parallel computing enabled)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- Output .tsv follows the documented layout: a total hapVar line, a position line listing targeted variant coordinates, and macrohaplotype rows with support counts
- Reported macrohaplotypes match ground-truth phased alleles for benchmark samples (e.g., the HG002 D2S441 example with 8646 and 7467 supporting reads)
- Each reported haplotype string has three ';'-separated parts corresponding to SNP, InDel, and STR calls
- No reported macrohaplotype cluster has fewer than MinReadCount reads or below MinReadProportion of total locus reads
- Read-to-anchor matching respects MaxMismatches (edit distance <= 1), and variant bases below Phred 15 or homopolymers >= 4 bp are skipped

## Limitations

- Homopolymers of length >= 4 and variant bases with Phred quality below 15 are skipped, so calls in those regions are absent
- Reads must match targeted-locus anchors within edit distance 1 or they are not assigned to the locus
- Multi-allelic detection depends on read-backed (physical) phasing; loci not spanned by single reads cannot be phased into macrohaplotypes

## Evidence

- [results] All called SNPs, InDels, and TRs from the same read are concatenated into a macrohaplotype string; the macrohaplotypes are then clustered based on the string context and ranked by the number of supported reads sharing the same called string.: [evidence span withheld — anchor status: not_found]
- [methods] MinReadCount default: "integer, minimum read count to report a macrohaplotype, default [2]"
- [methods] MinReadProportion default: "double, minimum read proportion of all reads for each locus to report a macrohaplotype, default [0.001]"
- [abstract] physical phasing abstract finding: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample"
- [readme] README usage: "usage: `java -jar -Xmx100G MacroHapCaller0.3.jar [options]`"
- [methods] homopolymerLen: "integer, homopolymer length threshold to skip call variant,default [4]"

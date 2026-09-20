---
name: long-read-indel-genotyping
description: Use when long-read alignments (BAM) must be genotyped for a predefined
  panel of InDel sites supplied as a BED configure file, applying homopolymer (default
  length 4) and Q15 base-quality filters, or when benchmarking InDel calls against
  DeepVariant at targeted loci.
when_to_use_negative:
- Reads are raw BAM/FASTQ not yet aligned, sorted, and indexed — preprocess first
  with MH_umi_dedup_map_pipe.sh (BAM to fastq, QC, UMI analysis, read-reference alignment)
- The goal is genome-wide discovery of novel InDels rather than genotyping a predefined
  targeted panel — MacroHapCaller only reports sites listed in the BED configure file
- InDels of interest lie inside homopolymer regions (homopolymerLen default 4) — candidates
  inside homopolymers are deliberately skipped by the caller
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: MacroHapCaller
  role: Java tool that scans CIGAR I/D operations at BED-configured InDel sites in
    long-read BAM alignments, applies homopolymerLen and LowQualThreshold filters,
    fills reference bases, and writes indel_calls.tsv
  repo: https://github.com/XuewenWangUGA/USAT
- name: MacroHapCaller 0.3
  role: Specific version invoked as java -jar -Xmx100G MacroHapCaller0.3.jar with
    -d for the InDel BED file, -r for the reference FASTA, -q 15 and -l for filter
    thresholds
- name: DeepVar
  role: 'External benchmark: DeepVariant v1.1.0 SNP/InDel calls on HG002 WGS were
    compared against MacroHapCaller calls restricted to the 57 targeted InDel loci
    (100% allele consistency at the nine sites DeepVariant detected)'
- name: PacBio HiFi
  role: High-quality long-read sequencing data type supplying the BAM input alignments
    for InDel genotyping
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:6d24c0e02df1d1320aef1c0edeb91ab1cce87fb38bb8b5a7dfa7c5181da2d094
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XuewenWangUGA/USAT
  tools:
  - DeepVar
  - MacroHapCaller
  - MacroHapCaller 0.3
  - PacBio HiFi
  - Python3
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# long-read-indel-genotyping

## Summary

Genotyping of targeted InDel loci from long-read (e.g. PacBio HiFi) BAM alignments using MacroHapCaller, which scans CIGAR I/D operations at BED-configured 0-based sites and filters candidates by homopolymer exclusion and Phred Q15 insert quality.

## When to use

Use when long-read alignments (BAM) must be genotyped for a predefined panel of InDel sites supplied as a BED configure file, applying homopolymer (default length 4) and Q15 base-quality filters, or when benchmarking InDel calls against DeepVariant at targeted loci.

## When NOT to use

- Reads are raw BAM/FASTQ not yet aligned, sorted, and indexed — preprocess first with MH_umi_dedup_map_pipe.sh (BAM to fastq, QC, UMI analysis, read-reference alignment)
- The goal is genome-wide discovery of novel InDels rather than genotyping a predefined targeted panel — MacroHapCaller only reports sites listed in the BED configure file
- InDels of interest lie inside homopolymer regions (homopolymerLen default 4) — candidates inside homopolymers are deliberately skipped by the caller

## Inputs

- Read-reference alignment file in BAM format (from MH_umi_dedup_map_pipe.sh or user-generated)
- InDel configure file in BED format (5 tab-separated columns: Chrom, ChromStart, ChromEnd, REF, ALT; 0-based coordinates)
- Genome reference sequence file in FASTA format (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)
- Optionally SNP panel position file (-n) and STR anchor configure file (-a) for joint macrohaplotype calling

## Outputs

- indel_calls.tsv — InDel call table in tab-separated .tsv format with one entry per configured InDel site per sample and consistent site counts across samples (reference-filled where no InDel is present)
- Phased macrohaplotype strings clustering STR, SNP, and InDel variants from each single long read

## How to apply

Parse the BED-format InDel configure file (5 columns: Chrom, ChromStart, ChromEnd, REF, ALT, with 0-based coordinates) and load the read-reference alignment BAM together with the genome reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta). Scan each aligned read's CIGAR string for I and D operations overlapping the configured sites to extract candidate InDel alleles. Discard candidates inside homopolymer regions using the homopolymerLen threshold (default 4) and filter variant bases by the Phred-scaled LowQualThreshold (default 15), requiring average insert base quality above Q15. Restrict calls to sites in the BED file; for listed sites lacking an InDel in a sample, fill in the corresponding reference bases so all samples share the same number of sites. Run MacroHapCaller (Java, e.g. java -jar -Xmx100G MacroHapCaller0.3.jar) with the -d InDel BED option alongside any SNP (-n) and STR anchor (-a) configure files, using defaults MinReadCount 2, MinReadProportion 0.001, LowQualThreshold 15, MaxMismatches 1, homopolymerLen 4.

## Related tools

- **MacroHapCaller** (Java tool that scans CIGAR I/D operations at BED-configured InDel sites in long-read BAM alignments, applies homopolymerLen and LowQualThreshold filters, fills reference bases, and writes indel_calls.tsv) — https://github.com/XuewenWangUGA/USAT
- **MacroHapCaller 0.3** (Specific version invoked as java -jar -Xmx100G MacroHapCaller0.3.jar with -d for the InDel BED file, -r for the reference FASTA, -q 15 and -l for filter thresholds)
- **DeepVar** (External benchmark: DeepVariant v1.1.0 SNP/InDel calls on HG002 WGS were compared against MacroHapCaller calls restricted to the 57 targeted InDel loci (100% allele consistency at the nine sites DeepVariant detected))
- **PacBio HiFi** (High-quality long-read sequencing data type supplying the BAM input alignments for InDel genotyping)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40_GRCh38.bam -o indel_calls.tsv -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 4 -m 1 -q 15 -c 2 -p 0.001 -t 12
```

## Evaluation signals

- indel_calls.tsv exists in .tsv format with one entry per configured InDel site per sample
- Site counts are consistent across all samples, confirming reference-base filling at sites without an InDel
- Calls are restricted to the configured BED loci (e.g. the 57 targeted InDel sites in HG002) — no calls outside the panel
- No called InDel lies inside a homopolymer region longer than homopolymerLen (default 4), and no called insert has average base quality at or below Q15
- Cross-tool check: at HG002 targeted loci, MacroHapCaller InDel alleles were 100% consistent with the nine sites detected by DeepVariant v1.1.0

## Limitations

- InDels inside homopolymer regions (default threshold homopolymerLen 4) are skipped, so such variants are never called
- Inserts whose average base quality does not exceed the LowQualThreshold (Q15 default) are filtered out
- Only sites listed in the InDel BED configure file are genotyped — no de novo InDel discovery
- External benchmarking shows DeepVariant detected only nine of the 57 targeted InDel sites in HG002, so cross-tool validation coverage at these loci is limited

## Evidence

- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read: "The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read"
- [readme] -d InDel BED configure file: [evidence span withheld — anchor status: not_found]
- [readme] homopolymer length threshold: "-l,--homopolymerLen integer, homopolymer length threshold to skip call variant,default [4]"
- [readme] LowQualThreshold default 15: [evidence span withheld — anchor status: not_found]
- [readme] pipeline input requirements: [evidence span withheld — anchor status: not_found]

---
name: long-read-macrohaplotype-calling
description: Use when input is a long-read (e.g., PacBio HiFi) read-reference BAM
  and targeted STR/SNP/InDel configure files, and you need read-backed phased multi-allelic
  macrohaplotypes — including variants (private mutations) not listed in the configure
  files.
when_to_use_negative:
- The analysis goal is short-read (NGS) phasing — MacroHapCaller is designed for long
  reads such as PacBio HiFi.
- 'You need a validated tool for untargeted whole-genome variant discovery: MacroHapCaller
  genotypes targeted STR/SNP/InDel loci defined by configure files, not de novo discovery
  across the genome.'
- Output haplotypes only need STR allele sizing/comparison without phased SNP/InDel/STR
  combinations — USAT handles post-hoc haplotype comparison instead.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0654
tools:
- name: MacroHapCaller
  role: 'Primary caller: genotypes targeted STR/SNP/InDel sites from each single long
    read and clusters variants into phased (read-backed) macrohaplotype strings; Java
    source is modified to retain private mutations absent from configure files'
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Preprocesses original PacBio HiFi BAM to fastq, QC, optional UMI analysis,
    statistical summary, alignment, sort and index to produce the caller's BAM input
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: USAT
  role: Downstream graphic-interface toolkit to view, align, compare, and size the
    haplotype sequences produced by the caller
  repo: https://github.com/XuewenWangUGA/USAT
- name: Java
  role: Runtime and modification target — MacroHapCaller is programmed in Java with
    parallel computing (-t 12 threads, -Xmx100G/-Xmx120G)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:e91ef78157ef774bd9ed88bb2fd863773025ce038e3acac9abab495ece3cec9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-Lab/MacroHapCaller
  tools:
  - Java
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# long-read-macrohaplotype-calling

## Summary

Genotype and phase targeted STR, SNP, and InDel variants into macrohaplotype strings from long PacBio HiFi reads using MacroHapCaller, including an extension to retain private variants absent from the user-supplied configure files (which stock MacroHapCaller discards).

## When to use

Use when input is a long-read (e.g., PacBio HiFi) read-reference BAM and targeted STR/SNP/InDel configure files, and you need read-backed phased multi-allelic macrohaplotypes — including variants (private mutations) not listed in the configure files.

## When NOT to use

- The analysis goal is short-read (NGS) phasing — MacroHapCaller is designed for long reads such as PacBio HiFi.
- You need a validated tool for untargeted whole-genome variant discovery: MacroHapCaller genotypes targeted STR/SNP/InDel loci defined by configure files, not de novo discovery across the genome.
- Output haplotypes only need STR allele sizing/comparison without phased SNP/InDel/STR combinations — USAT handles post-hoc haplotype comparison instead.

## Inputs

- Read-reference alignment BAM (from MH_umi_dedup_map_pipe.sh or user-generated)
- Reference genome fasta (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)
- STR anchor configure file in tabular plain text (e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (e.g. MHindels_v0.2.bed)
- SNP panel position configure file (e.g. MHsnps.pos.txt)

## Outputs

- Tabular .tsv of macrohaplotypes per locus with supported read counts, first line with total reads per locus, a Position line listing targeted variant coordinates, and macrohaplotype rows with counts
- HapVar columns augmented with private variants appended after panel-derived SNP/InDel/STR allele parts, separated by ';'

## How to apply

Preprocess reads with MH_umi_dedup_map_pipe.sh (BAM to fastq, QC, optional UMI analysis, alignment, sort, index) to obtain a read-reference BAM. Run MacroHapCaller 0.3 via java -jar with -i (BAM), -r (reference fasta, e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta), -a (STR anchor config), -d (InDel BED), -n (SNP panel), plus thresholds -l (homopolymerLen, default 4), -m (MaxMismatches, default 1), -q (LowQualThreshold 15), -c (MinReadCount, default 2), -p (MinReadProportion, default 0.001), -t (threads, default 12). To capture private mutations, modify the Java source so variant bases passing LowQualThreshold 15 and the MaxMismatches 1 anchor-match filter that are absent from the STR/InDel/SNP configure files are still genotyped and appended to the phased macrohaplotype strings rather than discarded — the current version discards an InDel if it is not in the configure file. Aggregate per-locus output so private variants are appended to the hapVar columns alongside panel-derived SNP/InDel/STR allele parts separated by ';', and write the result as a tabular .tsv with supported read counts.

## Related tools

- **MacroHapCaller** (Primary caller: genotypes targeted STR/SNP/InDel sites from each single long read and clusters variants into phased (read-backed) macrohaplotype strings; Java source is modified to retain private mutations absent from configure files) — https://github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Preprocesses original PacBio HiFi BAM to fastq, QC, optional UMI analysis, statistical summary, alignment, sort and index to produce the caller's BAM input) — https://github.com/Ge-Lab/MacroHapCaller
- **USAT** (Downstream graphic-interface toolkit to view, align, compare, and size the haplotype sequences produced by the caller) — https://github.com/XuewenWangUGA/USAT
- **Java** (Runtime and modification target — MacroHapCaller is programmed in Java with parallel computing (-t 12 threads, -Xmx100G/-Xmx120G))

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- Output .tsv exists and follows the documented layout: a first line with total reads per locus, a Position line listing targeted variant coordinates, and macrohaplotype rows with counts.
- Loci whose macrohaplotypes fall below MinReadCount 2 or MinReadProportion 0.001 are excluded from the report.
- Variant bases failing LowQualThreshold 15 (phred-scale) or exceeding MaxMismatches 1 in the anchor match are not called.
- Private variants absent from the configure files appear in the hapVar columns, appended after panel-derived parts and separated by ';' — their absence indicates the stock (unmodified) caller behavior.
- Genotyping accuracy and phased macrohaplotypes agree with known ground truth (HG002/HG003 benchmark amplicon data) and can be cross-checked against HipSTR/DeepVar.

## Limitations

- In the current (v0.3) version, targeted variants absent from the configure files are not reported — a called InDel is discarded if not included in the configure file, so private-mutation reporting requires source modification.
- No changelog is available for MacroHapCaller, so version behavior differences are hard to track.
- Haplotype reporting requires meeting both MinReadCount (default 2) and MinReadProportion (default 0.001) per locus; homopolymers at or above homopolymerLen (default 4) cause variants to be skipped.

## Evidence

- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read: "The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read"
- [abstract] read-backed phasing enables multi-allelic macrohaplotypes: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample"
- [methods] minimum read count to report a macrohaplotype: "integer, minimum read count to report a macrohaplotype, default [2]"
- [methods] phred-scale quality threshold for variant bases: "integer, phred-scale quality threshold for variant bases, default [15]"
- [readme] README usage line with options: "usage: `java -jar -Xmx100G MacroHapCaller0.3.jar [options]`"
- [readme] configure files of targeted variant positional information: "2. Configure files of targeted variant positional information for each of STRs, SNPs, and InDels"

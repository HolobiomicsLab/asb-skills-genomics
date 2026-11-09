---
name: macrohaplotype-genotyping
description: Use when long PacBio HiFi reads (BAM or fastq) must be genotyped AND
  phased at predefined targeted STR, SNP, and InDel loci in diploid, polyploid, or
  multi-source DNA-mixture samples.
when_to_use_negative:
- Variant discovery must be genome-wide/un-targeted — MacroHapCaller only calls loci
  listed in its STR anchor, SNP position, and InDel BED configure files
- Input reads are short NGS reads rather than long reads such as PacBio HiFi (the
  tool is built for long-read read-backed phasing)
- Only per-variant genotypes without phased haplotype context are needed and a genome-wide
  caller like DeepVariant suffices
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: MacroHapCaller
  role: 'Core caller: genotypes targeted STR/SNP/InDel loci from each long read and
    clusters variants into phased macrohaplotype strings'
  repo: github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Preprocesses PacBio HiFi BAM to fastq, QC, optional UMI dedup, read-reference
    alignment, sort, and index before calling
- name: Java
  role: Runtime for the parallelized MacroHapCaller jar (java -jar -Xmx100G MacroHapCaller0.3.jar)
- name: USAT
  role: Downstream graphic comparison of TR haplotype sequences, allele sizes, and
    distribution plots from MacroHapCaller haplotype output
  repo: https://github.com/XuewenWangUGA/USAT
- name: DeepVar
  role: Benchmarking comparator for genotyping accuracy and speed at targeted SNP/InDel
    loci
- name: HipSTR
  role: Benchmarking comparator for STR genotyping accuracy and speed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:3d3d96ab3de72769c7e3be8cda7b1c186c1dca3fb85a077a5cc5074213c52306
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DeepVar
  - HipSTR
  - Java
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - Python3
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# macrohaplotype-genotyping

## Summary

Simultaneously genotypes and read-back phases targeted STR/SSR, SNP, and InDel loci from long PacBio HiFi read alignments using MacroHapCaller 0.3 (Java), producing phased multi-allelic macrohaplotype strings per locus for diploid, polyploid, and DNA-mixture samples.

## When to use

Use when long PacBio HiFi reads (BAM or fastq) must be genotyped AND phased at predefined targeted STR, SNP, and InDel loci in diploid, polyploid, or multi-source DNA-mixture samples.

## When NOT to use

- Variant discovery must be genome-wide/un-targeted — MacroHapCaller only calls loci listed in its STR anchor, SNP position, and InDel BED configure files
- Input reads are short NGS reads rather than long reads such as PacBio HiFi (the tool is built for long-read read-backed phasing)
- Only per-variant genotypes without phased haplotype context are needed and a genome-wide caller like DeepVariant suffices

## Inputs

- Read-reference alignment BAM with index (from MH_umi_dedup_map_pipe.sh or user-generated), especially PacBio HiFi reads
- Genome reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)
- STR anchor configure file in tabular plain text (11 columns, one head line, e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- SNP panel position configure file in tabular plain text (e.g. MHsnps.pos.txt)
- InDel loci configure file in BED format (e.g. MHindels_v0.2.bed)

## Outputs

- Macrohaplotype result file (TSV) with per-locus phased multi-allelic haplotype strings
- Concordance table (e.g. concordance_table.tsv) tallying concordant/discordant genotypes vs benchmark callers such as DeepVariant, with called-locus counts per tool and variant class

## How to apply

Preprocess original PacBio HiFi reads with MH_umi_dedup_map_pipe.sh (BAM→fastq, read QC, optional UMI analysis, statistical summary, read-reference alignment, sort, index). Run MacroHapCaller 0.3 on the resulting read-reference BAM against the target reference FASTA (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta), supplying three configure files: a tabular STR anchor config (e.g. CODISSTR_anchor config, 11 columns), a tabular SNP panel position file, and a BED-format InDel file. Use tool defaults MinReadCount 2, MinReadProportion 0.001, LowQualThreshold 15 (phred-scale), MaxMismatches 1, homopolymerLen 4, with ~12 threads and 100–120 GB RAM; the Java implementation finishes in ~2 minutes for merged data from four full SMRTcell HiFi runs. Variant bases failing the phred-15 threshold or sitting in homopolymers ≥4 bp are skipped, and anchors are matched with edit distance ≤1. Restrict call sets to the configured targeted loci (e.g. 970 SNP loci, 57 InDel loci, 20 core forensic CODIS STR loci) and tally genotype concordance per variant class against benchmarks such as DeepVariant v1.1.0.

## Related tools

- **MacroHapCaller** (Core caller: genotypes targeted STR/SNP/InDel loci from each long read and clusters variants into phased macrohaplotype strings) — github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Preprocesses PacBio HiFi BAM to fastq, QC, optional UMI dedup, read-reference alignment, sort, and index before calling)
- **Java** (Runtime for the parallelized MacroHapCaller jar (java -jar -Xmx100G MacroHapCaller0.3.jar))
- **USAT** (Downstream graphic comparison of TR haplotype sequences, allele sizes, and distribution plots from MacroHapCaller haplotype output) — https://github.com/XuewenWangUGA/USAT
- **DeepVar** (Benchmarking comparator for genotyping accuracy and speed at targeted SNP/InDel loci)
- **HipSTR** (Benchmarking comparator for STR genotyping accuracy and speed)

## Examples

```
java -jar -Xmx100G MacroHapCaller0.3.jar -i hg002.8kampl.Q40_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta
```

## Evaluation signals

- On HG002 WGS HiFi data, restricting to the 57 configured InDel loci, DeepVariant v1.1.0 detected nine sites whose alleles were 100% consistent with MacroHapCaller calls
- Concordance table (concordance_table.tsv) contains numeric called-locus counts per tool and per variant class (SNP vs InDel)
- Output macrohaplotype TSV reports multi-allelic, read-back-phased haplotype strings per configured locus rather than independent variant calls
- Reported parameters match defaults: MinReadCount 2, MinReadProportion 0.001, LowQualThreshold 15, MaxMismatches 1, homopolymerLen 4, 12 threads
- Runtime stays on the order of ~2 minutes for four full PacBio SMRTcell HiFi runs with 12 threads and ≤120G RAM

## Limitations

- Only loci listed in the STR anchor, SNP, and InDel configure files are called; un-targeted or novel variant discovery is out of scope
- Variants in homopolymers of length ≥4 bp are skipped (homopolymerLen threshold), and low-support haplotypes below MinReadCount 2 / MinReadProportion 0.001 are not reported
- No changelog is available for MacroHapCaller (version history cannot be audited; citation listed as 'coming soon' at methods time)

## Evidence

- [abstract] MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample"
- [intro] calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings: "calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [methods] java -jar -Xmx100G MacroHapCaller0.3.jar [options]: "usage: java -jar -Xmx100G MacroHapCaller0.3.jar [options]"
- [methods] integer, minimum read count to report a macrohaplotype, default [2]: "integer, minimum read count to report a macrohaplotype, default [2]"
- [readme] It takes ~ 2 mins for merged data from four full runs of PacBio SMRTcell HiFi reads on a Linux machine with 12 threads, and a maximum allowed 120G RAM.: "It takes ~ 2 mins for merged data from four full runs of PacBio SMRTcell HiFi reads on a Linux machine with 12 threads, and a maximum allowed 120G RAM."

---
name: forensic-str-macrohaplotype-calling
description: Use when the input is PacBio HiFi (or other long-read) BAM/fastq data
  targeting STR loci such as the 20 CODIS core loci, and phased multi-allelic macrohaplotypes
  combining SNP + InDel + STR alleles are needed.
when_to_use_negative:
- Input reads are short Illumina reads rather than long reads (e.g. PacBio HiFi) —
  read-backed phasing across STR loci requires long reads spanning the targeted region
- No targeted variant position configure files (STR anchors, InDel BED, SNP panel)
  are available — MacroHapCaller requires positional information for each STR, SNP,
  and InDel locus
- The goal is downstream allele-size/length comparison or visualization of already-called
  haplotypes — USAT, not MacroHapCaller, is the tool for haplotype sequence alignment
  and allele length plotting
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: MacroHapCaller
  role: Java-based caller that simultaneously calls targeted STRs/SSRs, SNPs, and
    InDels from each single long read and clusters them into phased haplotype strings
    via read-backed phasing
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: 'Preprocess pipeline: converts original PacBio HiFi BAM to fastq, performs
    read QC, optional UMI analysis, statistical summary, and read-reference alignment
    with sort/index'
- name: USAT
  role: Downstream Java GUI toolkit to view, sort, plot, and align MacroHapCaller
    haplotype outputs and convert haplotype length in bp to repeat count (allele size)
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:aab4ebf3d88c78c0540b08886ed55914667e6946fddf1eabdf9bccd2cca8ac19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-Lab/MacroHapCaller
  tools:
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  - USAT
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# forensic-str-macrohaplotype-calling

## Summary

Calls phased macrohaplotypes (STR/SSR, SNP, and InDel blocks on single long reads) from PacBio HiFi BAMs with MacroHapCaller, using CODIS STR anchor, InDel BED, and SNP panel configure files for forensic-style genotyping of diploid, polyploid, and mixture samples.

## When to use

Use when the input is PacBio HiFi (or other long-read) BAM/fastq data targeting STR loci such as the 20 CODIS core loci, and phased multi-allelic macrohaplotypes combining SNP + InDel + STR alleles are needed.

## When NOT to use

- Input reads are short Illumina reads rather than long reads (e.g. PacBio HiFi) — read-backed phasing across STR loci requires long reads spanning the targeted region
- No targeted variant position configure files (STR anchors, InDel BED, SNP panel) are available — MacroHapCaller requires positional information for each STR, SNP, and InDel locus
- The goal is downstream allele-size/length comparison or visualization of already-called haplotypes — USAT, not MacroHapCaller, is the tool for haplotype sequence alignment and allele length plotting

## Inputs

- Read-reference alignment in BAM format (produced by MH_umi_dedup_map_pipe.sh or user-generated)
- Original PacBio HiFi reads in BAM or fastq
- STR anchor configure file (tabular, 11 columns per locus, e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (e.g. MHindels_v0.2.bed)
- SNP panel position file (e.g. MHsnps.pos.txt)
- Reference genome FASTA (GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- Per-sample .tsv macrohaplotype report with #Total hapVar header, marker names, Counts and HapVarLen columns; macrohaplotypes as SNP, InDel and STR allele blocks separated by ';', each with supported read count and haplotype length
- Aligned, sorted, and indexed BAM with statistical summary reports from the preprocess pipeline

## How to apply

Preprocess original PacBio HiFi reads with MH_umi_dedup_map_pipe.sh (BAM-to-fastq conversion, read quality control, optional UMI analysis, statistical summary, alignment to GRCh38 GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta, sort and index). Supply three configure files: the STR anchor config (e.g. CODISSTR_anchor.XW.config_v0.2.txt, tab-separated, 11 columns, 1-based), an InDel BED (MHindels_v0.2.bed, 0-based with REF/ALT), and a SNP position file (MHsnps.pos.txt, 1-based, IDs matching STR names). Run MacroHapCaller (java -jar -Xmx100G, -t 12 threads) with -i BAM, -r reference FASTA, -a STR anchors, -d InDel BED, and -n SNP panel. Apply default thresholds unless justified: MinReadCount 2, MinReadProportion 0.001, LowQualThreshold 15 (phred-scaled), MaxMismatches (anchor edit distance) 1, homopolymerLen 4 (skip variant calls in long homopolymers). Physical read-backed phasing clusters variants on each single read into phased haplotype strings, enabling multi-allelic calls suitable for mixtures and polyploid samples.

## Related tools

- **MacroHapCaller** (Java-based caller that simultaneously calls targeted STRs/SSRs, SNPs, and InDels from each single long read and clusters them into phased haplotype strings via read-backed phasing) — https://github.com/Ge-Lab/MacroHapCaller
- **MH_umi_dedup_map_pipe.sh** (Preprocess pipeline: converts original PacBio HiFi BAM to fastq, performs read QC, optional UMI analysis, statistical summary, and read-reference alignment with sort/index)
- **USAT** (Downstream Java GUI toolkit to view, sort, plot, and align MacroHapCaller haplotype outputs and convert haplotype length in bp to repeat count (allele size)) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx100G MacroHapCaller0.3.jar -i sample_GRCh38.bam -o sample.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -t 12
```

## Evaluation signals

- Each per-sample output .tsv exists and contains the #Total hapVar header with marker names, Counts and HapVarLen columns
- Reported macrohaplotypes meet the reporting thresholds (default MinReadCount 2 and MinReadProportion 0.001 of all reads per locus)
- Calls on public PacBio Revio WGS HiFi reads for HG002 at the 20 CODIS loci are consistent with calls derived from amplicon-based HiFi reads of the same sample
- Called genotypes match known ground truth (HG002/HG003 benchmark samples) for robustness and accuracy
- Anchor matching succeeds with edit distance within MaxMismatches default 1; variant bases pass the phred-scale LowQualThreshold 15

## Limitations

- No changelog is available for MacroHapCaller; the citation in methods was 'coming soon' at curation time
- Homopolymers of length >= homopolymerLen (default 4) cause variant calls at those positions to be skipped, which can affect haplotype content
- Multi-allelic detection relies on read-backed (physical) phasing, so coverage of a locus on single reads determines whether all alleles in a mixture can be recovered
- Reported thresholds (-c, -p) alter sensitivity: e.g. the README example uses -c 100 -p 0.01, much stricter than the defaults

## Evidence

- [abstract] MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes"
- [intro] The pipeline takes the BAM or fastq reads as input, align reads to the genome, sort, and index alignment: "The pipeline takes the BAM or fastq reads as input, align reads to the genome, sort, and index alignment"
- [readme] java -jar -Xmx100G MacroHapCaller0.3.jar [options] with default MinReadCount [2], MinReadProportion [0.001], LowQualThreshold [15]: "usage: `java -jar -Xmx100G MacroHapCaller0.3.jar [options]`"
- [readme] This pipeline will preprocess the originall PacBio HiFi reads in BAM to fastq, read quality control, umi-analysis (optional), statistical summary, read-reference alignment, sort, and index.: "This pipeline will preprocess the originall PacBio HiFi reads in BAM to fastq, read quality control, umi-analysis (optional), statistical summary"
- [methods] Preprocessed HG002/HG003/HG004 PacBio Revio WGS HiFi reads with the default CODIS configure files yielded macrohaplotypes at all 20 CODIS loci consistent with amplicon-based calls: "An example file for forensic CODIS STR of human genome is given in "CODISSTR_anchor.XW.config_v0.2.txt""

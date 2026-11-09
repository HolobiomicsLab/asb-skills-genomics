---
name: macrohaplotype-tsv-output-formatting
description: Use when running MacroHapCaller v0.3 on a read-reference BAM of long
  reads (e.g. PacBio HiFi of HG002 ~8-kb STR amplicons) and you must emit or validate
  the phased macrohaplotype .tsv allele table.
when_to_use_negative:
- Input is already a finished haplotype sequence table (marker <tab> haplotype <tab>
  SampleID) for comparison/plotting — that is USAT's role, not MacroHapCaller's .tsv
  emission
- Reads are short Illumina NGS reads without read-backed phasing support across the
  targeted loci, since macrohaplotype strings depend on physical phasing within single
  long reads (e.g. PacBio HiFi)
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2885
tools:
- name: MacroHapCaller
  role: Java program (v0.3) that performs the phased allele calling and writes the
    macrohaplotype .tsv output
  repo: github.com/Ge-Lab/MacroHapCaller
- name: Python3
  role: Used in the preprocessing pipeline that produces the BAM/fastq input consumed
    by MacroHapCaller
- name: MH_umi_dedup_map_pipe.sh
  role: Shell pipeline that preprocesses PacBio HiFi BAM to fastq, performs QC and
    read-reference alignment to generate the BAM input for the .tsv output
- name: USAT
  role: Downstream Java GUI toolkit that consumes haplotype sequences (marker/haplotype/SampleID
    table) for allele comparison and visualization
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:572e39befee2ceece88db7c09d697bee0847899d2aca24356eae5059b71e835c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  - USAT
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2885
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# macrohaplotype-tsv-output-formatting

## Summary

Formatting of MacroHapCaller genotype calls into the tab-separated macrohaplotype (.tsv) output, where targeted STR/SNP/InDel variants called per single long read are clustered into phased haplotype strings and written as a marker-by-allele table with read counts.

## When to use

Use when running MacroHapCaller v0.3 on a read-reference BAM of long reads (e.g. PacBio HiFi of HG002 ~8-kb STR amplicons) and you must emit or validate the phased macrohaplotype .tsv allele table.

## When NOT to use

- Input is already a finished haplotype sequence table (marker <tab> haplotype <tab> SampleID) for comparison/plotting — that is USAT's role, not MacroHapCaller's .tsv emission
- Reads are short Illumina NGS reads without read-backed phasing support across the targeted loci, since macrohaplotype strings depend on physical phasing within single long reads (e.g. PacBio HiFi)

## Inputs

- read-reference alignment file in BAM format (plus BAM index)
- STR anchor configure file (tab-separated, fixed headline, 11 columns per STR locus, e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (tabular plain text)
- SNP panel position file (tabular plain text)
- genome reference sequence file in .fasta format (GRCh38 GCA_000001405.15 no_alt analysis set)

## Outputs

- macrohaplotype result file (.tsv) with total hapVar header, one row per marker (e.g. CODIS STR locus) listing phased hapVar allele strings and supporting read counts

## How to apply

Run MacroHapCaller0.3.jar with -i (BAM), -a (STR anchor config), -d (InDel BED), -n (SNP positions), and -r (GRCh38 fasta), supplying filters such as MaxMismatches [1] for anchor edit distance, homopolymerLen [4] to skip homopolymer variant calls, LowQualThreshold [15] on phred-scaled base quality, MinReadCount [2], and MinReadProportion [0.001]. Per targeted STR locus, reads are screened by extracting a ~30 bp chunk adjacent to the locus and matching left/right anchors by edit distance (bounded by MaxMismatches), then the inter-anchor sequence is validated against the Motif_must_present repeat unit before allele tallying. Supported reads are aggregated per allele and only alleles passing MinReadCount and MinReadProportion are reported, so the .tsv table reflects read-backed (physical) phasing rather than statistical phasing. Validate output by checking the total hapVar header and that marker rows (e.g. D2S441 on HG002) carry allele haplotype strings with consistent read counts.

## Related tools

- **MacroHapCaller** (Java program (v0.3) that performs the phased allele calling and writes the macrohaplotype .tsv output) — github.com/Ge-Lab/MacroHapCaller
- **Python3** (Used in the preprocessing pipeline that produces the BAM/fastq input consumed by MacroHapCaller)
- **MH_umi_dedup_map_pipe.sh** (Shell pipeline that preprocesses PacBio HiFi BAM to fastq, performs QC and read-reference alignment to generate the BAM input for the .tsv output)
- **USAT** (Downstream Java GUI toolkit that consumes haplotype sequences (marker/haplotype/SampleID table) for allele comparison and visualization) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -r GRCh38_no_alt_analysis_set.fasta -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- Output .tsv begins with the total hapVar header and one marker name per targeted STR locus (e.g. CODIS loci, D2S441)
- Allele calls agree with the known ground truth for CODIS STR loci in HG002/HG003 8-kb PacBio HiFi data
- Every reported macrohaplotype allele has read count >= MinReadCount [2] and proportion >= MinReadProportion [0.001]; variant bases passed LowQualThreshold [15]
- Anchor matching within the ~30 bp chunk succeeded within MaxMismatches [1] edit distance and inter-anchor sequence contains Motif_must_present; homopolymers >= homopolymerLen [4] are skipped
- Runtime sanity check: ~2 mins for merged data from four full runs of PacBio SMRTcell HiFi reads with 12 threads and 120G RAM

## Limitations

- Variant calling is skipped at homopolymers at or above the homopolymerLen threshold (default 4)
- Anchor matching tolerates only MaxMismatches (default 1) edit distance, so heavily divergent flanking sequences may fail anchor detection
- Alleles below MinReadCount [2] or MinReadProportion [0.001] are not reported, potentially dropping rare alleles in mixtures
- No changelog available for MacroHapCaller per the repository discussion

## Evidence

- [intro] phased allele calling output: "calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [readme] STR config file structure: "Each line for one STR locus. There is one fixed headline before starting the detailed STR locus. Every line has 11 columns."
- [methods] read count filter: "integer, minimum read count to report a macrohaplotype, default [2]"
- [methods] read proportion filter: "double, minimum read proportion of all reads for each locus to report a macrohaplotype, default [0.001]"
- [methods] quality threshold filter: "integer, phred-scale quality threshold for variant bases, default [15]"
- [abstract] physical phasing rationale: "MacroHapCaller uses physical phasing (i.e., read-backed phasing) to identify macrohaplotypes, and thus it can detect multi-allelic macrohaplotypes for a given sample"

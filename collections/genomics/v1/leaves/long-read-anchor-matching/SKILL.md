---
name: long-read-anchor-matching
description: Use when calling targeted macrohaplotypes from a read-reference alignment
  BAM and per-locus variant configure files (STR anchor table, InDel BED, SNP panel)
  and you need to decide which reads belong to which targeted locus before allele
  calling.
when_to_use_negative:
- Input is raw PacBio HiFi BAM/fastq that has not yet been aligned to the reference
  — run the MH_umi_dedup_map_pipe.sh preprocessing (BAM-to-fastq, QC, UMI analysis,
  alignment, sort, index) first
- You need de novo variant discovery genome-wide rather than genotyping of a pre-specified
  panel of targeted STR/SNP/InDel loci
- Reads are short NGS reads where a single read cannot span the anchor plus all targeted
  variant sites of a macrohaplotype
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MacroHapCaller
  role: Java program that performs anchor-based read-to-locus matching (via -m/--MaxMismatches)
    before calling STR/SNP/InDel alleles per read and clustering macrohaplotypes
  repo: https://github.com/XuewenWangUGA/USAT
- name: MH_umi_dedup_map_pipe.sh
  role: Preprocessing pipeline that produces the read-reference alignment BAM consumed
    by the anchor-matching step
- name: PacBio HiFi
  role: Long-read sequencing platform whose high-quality reads supply the alignment
    input for anchor matching
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:0ca2977c4be48fc63aefa424bb259648c34b9f22705a9487738d92d2c9bb9bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XuewenWangUGA/USAT
  tools:
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - PacBio HiFi
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# long-read-anchor-matching

## Summary

Matching long reads (e.g., PacBio HiFi) to targeted STR/SNP/InDel loci in MacroHapCaller using anchor-based matching with a bounded edit distance, so only reads that actually cover a targeted locus are used for macrohaplotype calling.

## When to use

Use when calling targeted macrohaplotypes from a read-reference alignment BAM and per-locus variant configure files (STR anchor table, InDel BED, SNP panel) and you need to decide which reads belong to which targeted locus before allele calling.

## When NOT to use

- Input is raw PacBio HiFi BAM/fastq that has not yet been aligned to the reference — run the MH_umi_dedup_map_pipe.sh preprocessing (BAM-to-fastq, QC, UMI analysis, alignment, sort, index) first
- You need de novo variant discovery genome-wide rather than genotyping of a pre-specified panel of targeted STR/SNP/InDel loci
- Reads are short NGS reads where a single read cannot span the anchor plus all targeted variant sites of a macrohaplotype

## Inputs

- Read-reference alignment BAM file (plus BAM index) from MH_umi_dedup_map_pipe.sh or user-generated alignment
- STR anchor configure file in tabular plain text (11 columns per locus, e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel target configure file in BED format (tabular plain text)
- SNP panel position configure file in tabular plain text
- Genome reference sequence file in .fasta format (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)

## Outputs

- Read-to-locus assignments (matched reads per targeted locus) used internally for per-locus allele calling
- Downstream ranked macrohaplotype table (.tsv) via the -o option, containing marker name, support counts, hapVar length, and haplotype strings with SNP/InDel/STR parts

## How to apply

Load the STR anchor configure file (tabular text, one 11-column line per STR locus, e.g. CODISSTR_anchor.XW.config_v0.2.txt) together with the InDel BED and SNP panel position files, and the read-reference alignment BAM. For each targeted locus, match reads to the locus using the anchor sequence defined in the STR configure file, tolerating a maximum anchor edit distance controlled by -m/--MaxMismatches (default [1]). Reads that match an anchor within this threshold proceed to per-locus SNP, InDel, and STR allele calling; reads exceeding the edit-distance tolerance are not assigned to the locus. Raising MaxMismatches increases locus recovery on noisier alignments but risks mis-assigning reads; the default of 1 balances sensitivity and specificity for high-quality PacBio HiFi alignments produced by MH_umi_dedup_map_pipe.sh.

## Related tools

- **MacroHapCaller** (Java program that performs anchor-based read-to-locus matching (via -m/--MaxMismatches) before calling STR/SNP/InDel alleles per read and clustering macrohaplotypes) — https://github.com/XuewenWangUGA/USAT
- **MH_umi_dedup_map_pipe.sh** (Preprocessing pipeline that produces the read-reference alignment BAM consumed by the anchor-matching step)
- **PacBio HiFi** (Long-read sequencing platform whose high-quality reads supply the alignment input for anchor matching)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.8kampl.Q40.fastq.gz_GRCh38.bam -o hg002.8kampl.Q40_GRCh38.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GRCh38_no_alt_analysis_set.fasta -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- All reads contributing to a locus's macrohaplotype clusters matched the anchor within the MaxMismatches edit-distance threshold (default 1)
- The output .tsv follows the documented layout: a total hapVar line, a position line of targeted variant coordinates, and macrohaplotype rows with support counts
- Reported macrohaplotypes match ground-truth phased alleles for benchmark samples (e.g. HG002/HG003 ~8-kb amplicons of 20 core forensic STR loci), e.g. the D2S441 example with 8646 and 7467 supporting reads
- Clusters failing MinReadCount (default 2) or MinReadProportion (default 0.001) of total reads for the locus are absent from the final ranked table

## Limitations

- Anchor matching is bounded by the MaxMismatches edit distance (default [1]); reads with more anchor mismatches are dropped, which can lose true-locus reads in noisier alignments
- Matching quality depends on the read-reference alignment produced upstream; misalignment of the input BAM propagates into wrong locus assignments
- Variant bases with Phred quality below LowQualThreshold (default 15) and homopolymers of length >= 4 are skipped during downstream calling, so anchor-matched reads may still yield incomplete macrohaplotype strings

## Evidence

- [readme] integer, maximum distance of edit distance in anchor match, default [1]: "integer, maximum distance of edit distance in anchor match, default [1]"
- [methods] Load the read-reference alignment BAM and the three variant configure files and match reads to each targeted locus using anchors: [evidence span withheld — anchor status: not_found]
- [readme] The configure file listed the STR loci information in a tabular separated text file: "The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR of human genome is given in "CODISSTR_anchor.XW.config_v0.2.txt""
- [intro] The pipeline takes the BAM or fastq reads as input, align reads to the genome, sort, and index alignment: "The pipeline takes the BAM or fastq reads as input, align reads to the genome, sort, and index alignment"

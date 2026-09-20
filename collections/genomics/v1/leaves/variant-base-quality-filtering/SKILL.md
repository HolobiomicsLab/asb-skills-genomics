---
name: variant-base-quality-filtering
description: Use when calling targeted SNPs, InDels, or STR alleles from long-read
  (e.g. PacBio HiFi) alignments and low-quality bases or homopolymer-adjacent indel
  artifacts could contaminate per-read macrohaplotype strings.
when_to_use_negative:
- When the goal is to retain all raw alleles for downstream re-scoring or error modeling
  rather than produce a cleaned macrohaplotype call set
- When data are short reads or non-HiFi long reads where homopolymer slippage patterns
  differ and the homopolymerLen=4 skip rule may not be calibrated
- When variants of interest lie inside homopolymer runs (e.g. some STR-adjacent indels),
  since the default homopolymerLen threshold will systematically skip them
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: MacroHapCaller
  role: Java-based targeted STR/SNP/InDel caller whose -q (LowQualThreshold, default
    15) and -l (homopolymerLen, default 4) options implement the base-quality and
    homopolymer filters during per-read allele calling
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_001
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:c4c1fa16027599e20a93e02d9b5a24feb059d9fe42c150f95208434e2108aa52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XuewenWangUGA/USAT
  tools:
  - MH_umi_dedup_map_pipe.sh
  - MacroHapCaller
  - PacBio HiFi
  - Python3
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Variant base quality filtering

## Summary

Discards variant bases and candidate InDel calls whose Phred-scaled base quality falls below a threshold (default Q15) and skips variants inside homopolymer regions of length >= 4 during targeted variant calling with MacroHapCaller, suppressing sequencing-error-derived alleles before macrohaplotype clustering.

## When to use

Use when calling targeted SNPs, InDels, or STR alleles from long-read (e.g. PacBio HiFi) alignments and low-quality bases or homopolymer-adjacent indel artifacts could contaminate per-read macrohaplotype strings.

## When NOT to use

- When the goal is to retain all raw alleles for downstream re-scoring or error modeling rather than produce a cleaned macrohaplotype call set
- When data are short reads or non-HiFi long reads where homopolymer slippage patterns differ and the homopolymerLen=4 skip rule may not be calibrated
- When variants of interest lie inside homopolymer runs (e.g. some STR-adjacent indels), since the default homopolymerLen threshold will systematically skip them

## Inputs

- Read-reference alignment BAM (e.g. PacBio HiFi reads aligned to GRCh38 GCA_000001405.15_no_alt_analysis_set.fasta)
- Genome reference FASTA (for homopolymer context detection)
- Configured targeted variant sites: STR anchor config, InDel BED file, SNP position file

## Outputs

- Quality- and homopolymer-filtered per-read allele calls feeding the macrohaplotype .tsv (macrohaplotype rows with supporting-read counts)
- Filtered per-site InDel call table (indel_calls.tsv) when used in the InDel calling module

## How to apply

During MacroHapCaller's per-read variant calling, apply the -q/--LowQualThreshold parameter (Phred-scale quality threshold for variant bases, default 15): require the average quality of insert/variant bases to exceed Q15 before an allele is accepted. Independently, apply the -l/--homopolymerLen threshold (default 4): skip calling variants that lie inside homopolymer regions of length >= 4, since polymerase slippage in homopolymers confounds indel placement. These filters operate on the read-reference alignment BAM before alleles are concatenated into the ';'-separated macrohaplotype string, so noisy alleles are excluded rather than merely down-weighted. The threshold parameters are configurable on the command line, e.g. -l 2 -q 15, allowing stricter homopolymer exclusion or quality cutoffs for lower-Q score datasets.

## Related tools

- **MacroHapCaller** (Java-based targeted STR/SNP/InDel caller whose -q (LowQualThreshold, default 15) and -l (homopolymerLen, default 4) options implement the base-quality and homopolymer filters during per-read allele calling) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i sample.bam -o out.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GRCh38.fasta -l 4 -q 15
```

## Evaluation signals

- No reported allele should derive from variant bases with Phred quality at or below 15 when the default LowQualThreshold is used
- No candidate InDel should be called at positions inside homopolymer regions of length >= the homopolymerLen threshold (default 4)
- Output .tsv macrohaplotype rows should still meet MinReadCount (default 2) and MinReadProportion (default 0.001) reporting thresholds after filtering
- Called macrohaplotypes at the 20 CODIS loci should match ground-truth phased alleles for benchmark samples (HG002/HG003) and remain consistent between amplicon-based and WGS HiFi runs
- All listed InDel sites should retain an entry (reference-filled when no variant called) so site counts are consistent across samples in indel_calls.tsv

## Limitations

- The homopolymer skip rule (default length 4) excludes variant calls within homopolymers, which trades sensitivity for specificity and may drop true alleles in homopolymer-rich loci
- A single global Phred threshold (default 15) is applied to all variant bases; it cannot adapt to per-locus or per-strand error profiles
- Filters are tuned for PacBio HiFi long-read error profiles; behavior on other platforms is not characterized in the article

## Evidence

- [readme] integer, phred-scale quality threshold for variant bases, default [15]: "integer, phred-scale quality threshold for variant bases, default [15]"
- [readme] homopolymer skip rule: "integer, homopolymer length threshold to skip call variant,default [4]"
- [methods] InDel filtering criteria: [evidence span withheld — anchor status: not_found]
- [abstract] validated macrohaplotypes vs ground truth: "Robust and accurate genotyping and phased macrohaplotypes were obtained with MacroHapCaller compared with the known ground truth"

---
name: variant-panel-config-authoring
description: Use when preparing a MacroHapCaller run and you must define which STR
  loci, SNPs, and InDels are genotyped, or when planned private mutations must be
  added to the panel because unlisted variants are not reported.
when_to_use_negative:
- You expect de novo or private mutations absent from the configure files to be reported
  — MacroHapCaller v0.3 discards InDels (and other targeted variants) not present
  in the configure files
- You need genotyping of untargeted genome-wide variants rather than a user-supplied
  panel of STRs, SNPs, and InDels
- Your variant coordinates refer to a different reference build than the BAM alignment
  (e.g. not GRCh38 GCA_000001405.15 no-alt analysis set), since positions will not
  match
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: MacroHapCaller
  role: Consumes the authored STR/InDel/SNP configure files via -a, -d, -n to genotype
    and phase targeted loci into macrohaplotype strings
  repo: github.com/Ge-Lab/MacroHapCaller
- name: USAT
  role: Downstream visualization/comparison tool whose BED-format TR locus definition
    mirrors the panel design conventions (Chrom, ChromStart, ChromEnd, Name, motif,
    Ref_allele)
  repo: https://github.com/XuewenWangUGA/USAT
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:1bd1b0183dd3d7750b545ad003574f78f57b16b1efe631d564aa9ab181d6ab7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MacroHapCaller
  - USAT
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# variant-panel-config-authoring

## Summary

Author the three MacroHapCaller configure files — STR anchor config, InDel BED, and SNP panel position file — that define the targeted variant loci genotyped and phased from long reads. The configure files gate which variants are called: any variant absent from them is silently discarded, so panel content directly determines macrohaplotype composition.

## When to use

Use when preparing a MacroHapCaller run and you must define which STR loci, SNPs, and InDels are genotyped, or when planned private mutations must be added to the panel because unlisted variants are not reported.

## When NOT to use

- You expect de novo or private mutations absent from the configure files to be reported — MacroHapCaller v0.3 discards InDels (and other targeted variants) not present in the configure files
- You need genotyping of untargeted genome-wide variants rather than a user-supplied panel of STRs, SNPs, and InDels
- Your variant coordinates refer to a different reference build than the BAM alignment (e.g. not GRCh38 GCA_000001405.15 no-alt analysis set), since positions will not match

## Inputs

- STR anchor configure file in tabular plain text (11 columns per locus plus fixed headline, e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format (e.g. MHindels_v0.2.bed)
- SNP panel position file in tabular plain text (e.g. MHsnps.pos.txt)
- Reference genome fasta (GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta) for coordinate checking

## Outputs

- Completed STR anchor config file ready for the -a option
- Completed InDel BED file ready for the -d option
- Completed SNP panel position file ready for the -n option
- Macrohaplotype .tsv output whose Position line and hapVar strings reflect only panel-listed variants

## How to apply

Build three tabular plain-text configure files that are passed via -a, -d, and -n. The STR anchor file (e.g. CODISSTR_anchor.XW.config_v0.2.txt) lists one STR locus per line with 11 columns preceded by a fixed headline, where the last integer column is the estimated anchor property. The InDel file uses BED format (e.g. MHindels_v0.2.bed) and the SNP file (e.g. MHsnps.pos.txt) lists targeted SNP positions. At runtime, variants passing the LowQualThreshold (default 15) and MaxMismatches (default 1) filters but absent from these files are discarded, so every variant intended to appear in the phased macrohaplotype strings must be pre-registered in the panel; reporting private mutations outside the configure files is a planned future extension, not current functionality. Validate panels against the reference (e.g. GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta) coordinates so targeted variant positions match the alignment BAM's reference build.

## Related tools

- **MacroHapCaller** (Consumes the authored STR/InDel/SNP configure files via -a, -d, -n to genotype and phase targeted loci into macrohaplotype strings) — github.com/Ge-Lab/MacroHapCaller
- **USAT** (Downstream visualization/comparison tool whose BED-format TR locus definition mirrors the panel design conventions (Chrom, ChromStart, ChromEnd, Name, motif, Ref_allele)) — https://github.com/XuewenWangUGA/USAT

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.bam -o hg002.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 2 -m 1 -q 15 -c 100 -p 0.01 -t 12
```

## Evaluation signals

- Each STR locus line in the anchor config has exactly 11 columns and the file begins with the fixed headline
- InDel and SNP panel positions match the coordinates of the reference fasta used for alignment (GRCh38 GCA_000001405.15 no-alt analysis set)
- The resulting macrohaplotype .tsv Position line lists exactly the targeted variant coordinates from the configure files, with no extra variants
- Loci whose macrohaplotypes fall below MinReadCount 2 (or the chosen -c value) or MinReadProportion 0.001 (chosen -p) are absent from the output
- A known variant deliberately omitted from the panel does not appear in the output hapVar strings, confirming panel-gated calling

## Limitations

- In the current version (v0.3), targeted variants absent from the configure files are not reported — a called InDel not in the configure file is discarded; private-mutation reporting is a planned future extension
- The STR anchor config requires the Name, Basic_motif_period, and Inner_offset columns to be set, and the first four columns must be unique across loci
- No changelog is available for MacroHapCaller, so panel-format changes between versions are undocumented

## Evidence

- [readme] The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR of human genome is given in "CODISSTR_anchor.XW.config_v0.2.txt": "The configure file listed the STR loci information in a tabular separated text file. An example file for forensic CODIS STR of human genome is given in "CODISSTR_anchor.XW.config_v0.2.txt""
- [other] a called In Del is discarded if it is not included in the configure file: [evidence span withheld — anchor status: not_found]
- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read: "The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [readme] **-d,--inDelBedFile** <arg> configure file of InDels in BED format in tabular plain text: "**-d,--inDelBedFile** <arg> &nbsp; configure file of InDels in BED format in tabular plain text"
- [readme] The first four columns must be unique across all TR loci. The Name, Basic_motif_period, and Inner_offset must be set.: [evidence span withheld — anchor status: external_doc]

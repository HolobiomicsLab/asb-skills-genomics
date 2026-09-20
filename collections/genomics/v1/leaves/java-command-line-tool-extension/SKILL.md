---
name: java-command-line-tool-extension
description: Use when a targeted variant caller discards variants not listed in its
  configure files and the analysis goal requires reporting those private mutations
  alongside the panel-defined STR/SNP/InDel alleles.
when_to_use_negative:
- When all variants of interest are already covered by the STR/InDel/SNP configure
  files — stock MacroHapCaller v0.3 handles targeted calling without source modification
- When reads are short Illumina reads rather than long PacBio HiFi reads — MacroHapCaller
  is designed for read-backed phasing on long reads
- When only post-hoc haplotype sequence comparison or allele sizing is needed — use
  USAT on existing haplotype output instead of modifying the caller
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3673
tools:
- name: MacroHapCaller
  role: Java command-line macrohaplotype caller whose source is extended to report
    private variants
  repo: https://github.com/Ge-Lab/MacroHapCaller
- name: Java
  role: Language in which MacroHapCaller is programmed and recompiled; parallel computing
    enabled
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/elps.202300143
    title: A complete pipeline enables haplotyping and phasing macrohaplotype in long
      sequencing reads for polyploidy samples and a multi‐source DNA mixture
  source_content_hash: sha256:e4e360be31263c367e500c1056709c7782988cac189cc0cee07554530b9f8a30
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Ge-Lab/MacroHapCaller
  tools:
  - Java
  - MacroHapCaller
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# java-command-line-tool-extension

## Summary

Modify the Java source of a published command-line genotyping tool (MacroHapCaller v0.3) so that variant calls passing the existing quality/anchor filters but absent from the user-supplied configure files are still genotyped and reported as private variants in the phased macrohaplotype output.

## When to use

Use when a targeted variant caller discards variants not listed in its configure files and the analysis goal requires reporting those private mutations alongside the panel-defined STR/SNP/InDel alleles.

## When NOT to use

- When all variants of interest are already covered by the STR/InDel/SNP configure files — stock MacroHapCaller v0.3 handles targeted calling without source modification
- When reads are short Illumina reads rather than long PacBio HiFi reads — MacroHapCaller is designed for read-backed phasing on long reads
- When only post-hoc haplotype sequence comparison or allele sizing is needed — use USAT on existing haplotype output instead of modifying the caller

## Inputs

- PacBio HiFi read-reference alignment BAM (e.g. HG002 whole-genome long-read alignments)
- GRCh38 reference fasta (GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta)
- STR anchor configure file in tabular plain text (e.g. CODISSTR_anchor.XW.config_v0.2.txt)
- InDel configure file in BED format
- SNP panel position configure file (tabular plain text)
- MacroHapCaller Java source code (MacroHapCaller0.3.jar)

## Outputs

- Tabular .tsv of macrohaplotypes with supported read counts, containing a first line of total reads per locus, a Position line of targeted variant coordinates, and macrohaplotype rows with counts; private variants appended to hapVar columns alongside ';' separated panel-derived allele parts
- Recompiled MacroHapCaller jar with private-variant calling enabled

## How to apply

Clone the tool's repository (github.com/Ge-Lab/MacroHapCaller), locate the code path where a called InDel or SNP is discarded because it is missing from the STR anchor (-a), InDel BED (-d), or SNP panel (-n) configure files, and change it so bases passing the LowQualThreshold (phred 15) and MaxMismatches (1) anchor-edit-distance filter are genotyped and appended to the phased macrohaplotype strings as private-variant entries. Recompile the jar (e.g. MacroHapCaller0.3.jar) and run with the standard option set: -i BAM, -r reference fasta, -l homopolymerLen 4, -m 1, -q 15, -c MinReadCount 2, -p MinReadProportion 0.001, -t 12 threads. Aggregate per-locus output so private variants are appended to the hapVar columns next to the ';' separated STR/InDel/SNP allele parts, and emit a tabular .tsv. Rationale: the authors confirm current versions discard targeted variants absent from configure files, so private-mutation calling requires source modification rather than a CLI flag.

## Related tools

- **MacroHapCaller** (Java command-line macrohaplotype caller whose source is extended to report private variants) — https://github.com/Ge-Lab/MacroHapCaller
- **Java** (Language in which MacroHapCaller is programmed and recompiled; parallel computing enabled)

## Examples

```
java -jar -Xmx120G MacroHapCaller0.3.jar -i hg002.hifi.bam -o hg002.tsv -a CODISSTR_anchor.XW.config_v0.2.txt -d MHindels_v0.2.bed -n MHsnps.pos.txt -r GCA_000001405.15_GRCh38_no_alt_analysis_set.fasta -l 4 -m 1 -q 15 -c 2 -p 0.001 -t 12
```

## Evaluation signals

- Output .tsv exists and follows the documented layout: first line with total reads per locus, a Position line listing targeted variant coordinates, and macrohaplotype rows with counts
- Loci below MinReadCount 2 or MinReadProportion 0.001 are excluded from the output
- Private variants appear in the hapVar columns appended next to the ';' separated STR/InDel/SNP allele parts
- Variant bases kept in the output pass LowQualThreshold 15 and anchor-match MaxMismatches 1
- Ground-truth comparison: genotyped macrohaplotypes for HG002 loci remain consistent with unmodified caller results for panel variants

## Limitations

- Private-mutation calling is not present in the current release: a called InDel is discarded if it is not included in the configure file; support is a planned future extension requiring source modification
- No changelog is available for MacroHapCaller, so version-to-version behavior differences cannot be verified from the repo
- Homopolymer variants at or above homopolymerLen (default 4) are skipped during variant calling, limiting private-mutation recovery in homopolymer regions

## Evidence

- [intro] The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings: "The MacroHapCaller calls targeted STRs/SSRs, SNPs, and InDels sites simultaneously in a row from each single NGS read and clusters the variants into phased haplotype strings"
- [methods] usage options: "integer, phred-scale quality threshold for variant bases, default [15]"
- [methods] filters: "integer, minimum read count to report a macrohaplotype, default [2]"
- [discussion] no changelog: "_No changelog found._"
- [readme] repo description: "MacroHapCaller is programmed in Java with parallel computing enabled so it can run on any computing platform."

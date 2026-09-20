---
name: perl-pipeline-configuration
description: Use when running InDelGT to genotype InDels across a mapping population
  (CP, BC1, BC2, F2) from paired-end NGS reads and you must set up parameters.ini
  (folders, parameters, data files) before invoking the Perl pipeline.
when_to_use_negative:
- Input data are not paired-end reads of a two-parent mapping population (InDelGT
  targets BC1, BC2, F2 inbred and CP hybrid populations, not unrelated samples or
  single-sample variant calling)
- You only need general variant discovery independent of Mendelian segregation filtering
  — BCFtools alone without the InDelGT segregation-ratio filtering (PVALUE:0.01) would
  be more appropriate
- Parental InDel catalogs already exist and only genotyping is needed — use the --nocatalog
  option to skip the first step instead of re-running catalog generation
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: Perl pipeline configured via parameters.ini and run with one command line
    to generate InDel catalogs, call genotypes, and analyze segregation
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Read alignment engine whose installation path (BWA_FOLD) must be set in parameters.ini
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Alignment sorting, installed and its path (SAMTOOLS_FOLD) declared in parameters.ini
- name: BCFtools 1.9
  role: Variant calling within the pipeline, path declared as BCFTOOLS_FOLD in parameters.ini
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:504d3e9276cca8279c2aa4baab0f554c1fcadfc66451efab2615cb9beb89dc40
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/tongchf/InDelGT
  repo_url: https://github.com/tongchf/InDelGT
  tools:
  - BCFtools
  - BCFtools 1.9
  - BWA
  - BWA 0.7.17
  - InDelGT
  - SAMtools
  - SAMtools 1.9
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# perl-pipeline-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure InDelGT, a Perl pipeline for extracting InDel genotypes from a hybrid population, by preparing the parameters.ini file that specifies software paths, quality/depth/segregation thresholds, and reference/FASTQ data files before a single-command run.

## When to use

Use when running InDelGT to genotype InDels across a mapping population (CP, BC1, BC2, F2) from paired-end NGS reads and you must set up parameters.ini (folders, parameters, data files) before invoking the Perl pipeline.

## When NOT to use

- Input data are not paired-end reads of a two-parent mapping population (InDelGT targets BC1, BC2, F2 inbred and CP hybrid populations, not unrelated samples or single-sample variant calling)
- You only need general variant discovery independent of Mendelian segregation filtering — BCFtools alone without the InDelGT segregation-ratio filtering (PVALUE:0.01) would be more appropriate
- Parental InDel catalogs already exist and only genotyping is needed — use the --nocatalog option to skip the first step instead of re-running catalog generation

## Inputs

- reference genome FASTA file (reference.fasta)
- Paired-end FASTQ read files for two parents and all progeny (e.g. 2 parents + 20 progeny from exampledata.tar.gz, figshare 15131649)
- parameters.ini configuration file (folders, parameters, data files sections)

## Outputs

- Parental InDel catalogs (step 1 output)
- InDel genotype calls across the population, e.g. CP segregation-type files aaxab.txt, aaxbc.txt, abxaa.txt, abxcc.txt, abxab.txt, abxac.txt, abxcd.txt
- InDel segregation analysis results (Mendelian segregation-ratio tests per site)

## How to apply

Write a parameters.ini with three sections: [folders] listing BWA_FOLD (e.g. bwa-0.7.17), SAMTOOLS_FOLD (samtools-1.9), BCFTOOLS_FOLD (bcftools-1.9), INDELGT_FOLD, and DATAFOLD; [parameters] with GQ:30 (minimum InDel genotype score), MAPQ:5 (minimum read mapping quality), PROGENY_NUMBER (e.g. 20), MISPCT:20 (percent missing genotypes per InDel), HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01 (segregation-ratio test threshold), and THREADS:20; and [data files] with REFERENCE_FILE:reference.fasta plus the first and second read files of the two parents and all progeny. BWA 0.7.17, SAMtools 1.9, and BCFtools 1.9 must be installed first since the pipeline internally calls them for alignment, sorting, and variant calling. Then run the pipeline with a single command line, e.g. 'perl PathToInDelGT/InDelGT.pl -p CP -o directory' for a hybrid (CP) population. Rationale: the depth thresholds reflect that heterozygote calling requires higher coverage than homozygotes, and the PVALUE filter keeps only InDel sites passing the Mendelian segregation-ratio test.

## Related tools

- **InDelGT** (Perl pipeline configured via parameters.ini and run with one command line to generate InDel catalogs, call genotypes, and analyze segregation) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Read alignment engine whose installation path (BWA_FOLD) must be set in parameters.ini) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Alignment sorting, installed and its path (SAMTOOLS_FOLD) declared in parameters.ini)
- **BCFtools 1.9** (Variant calling within the pipeline, path declared as BCFTOOLS_FOLD in parameters.ini)

## Examples

```
perl PathToInDelGT/InDelGT.pl -p CP -o directory
```

## Evaluation signals

- parameters.ini parses with all three sections ([folders], [parameters], [data files]) and contains GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20 and PROGENY_NUMBER matching the actual progeny count
- The pipeline runs end-to-end with the single command 'perl PathToInDelGT/InDelGT.pl -p CP -o directory' without path errors, confirming BWA/SAMtools/BCFtools folders resolve
- Output directory contains the expected CP segregation-type genotype files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc.txt, abxab.txt, abxac.txt, abxcd.txt)
- Per-InDel missingness is within MISPCT:20 and retained sites pass the PVALUE:0.01 Mendelian segregation-ratio test
- Downstream validation: genotypes at known loci agree with an independent method (e.g. 86/90, ~95%, concordance with PAGE genotypes)

## Limitations

- No version number is stated for InDelGT itself in the article; the README references InDelGT-1.0 in the INDELGT_FOLD path
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1-5 bp) and require higher coverage depth
- Determining heterozygotes requires a higher coverage depth than homozygotes, so low-depth samples may be miscalled despite the HETEROZYGOUS_DEPTH:3 setting
- The paths in the example parameters.ini (e.g. /home/pan/bwa-0.7.17) are environment-specific and must be edited for local installations

## Evidence

- [methods] # the minimum score of InDel genotypes
    GQ:30: "# the minimum score of InDel genotypes
    GQ:30"

---
name: cross-pollination-population-marker-segregation-analysis
description: Use when you have InDel genotype calls for two parents and their outbred
  F1 progeny (CP population) and need to filter markers by Mendelian segregation ratio
  tests and assign each marker a segregation type before linkage map construction.
when_to_use_negative:
- Population is a traditional inbred-line type (BC1, BC2, F2) where the seven CP segregation
  types and their expected ratios do not apply — use the appropriate -p type instead
  of CP-specific classification
- Genotypes come from another variant caller rather than the InDelGT genotyping step,
  so the GQ/MAPQ/MISPCT/depth thresholds and genotype coding the segregation test
  expects are not in place
- The goal is variant discovery or catalog generation (steps 1–2 of InDelGT), not
  marker filtering and segregation-type assignment
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0625
tools:
- name: InDelGT
  role: Runs the segregation analysis (step 3) on InDelGT genotype calls; invoked
    with --nofilter to skip or by default executes the segregation-type analysis
  repo: https://github.com/tongchf/InDelGT
- name: Statistics::Distributions
  role: Perl module used to compute P-values for testing each InDel site's segregation
    ratio against expected Mendelian ratios
- name: Parallel::ForkManager
  role: Perl module used to parallelize per-site segregation testing across THREADS
    20
- name: Getopt::Long
  role: Perl module for parsing InDelGT.pl command-line options such as --nofilter
    and -p CP
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:a70f36e39b3dac4251bc417990a176d2b1c962cadf50d170fca429c769527418
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
  - Getopt::Long
  - InDelGT
  - Parallel::ForkManager
  - Statistics::Distributions
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cross-pollination-population-marker-segregation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Step 3 of InDelGT: test each cataloged InDel site's genotype segregation in an F1 cross-pollination (CP) hybrid population (e.g., 2 parents and 20 Populus progeny) against expected Mendelian ratios, then classify retained sites into the seven CP segregation types (ab×aa, aa×ab, ab×ab, ab×cc, aa×bc, ab×ac, ab×cd) for genetic linkage mapping.

## When to use

Use when you have InDel genotype calls for two parents and their outbred F1 progeny (CP population) and need to filter markers by Mendelian segregation ratio tests and assign each marker a segregation type before linkage map construction.

## When NOT to use

- Population is a traditional inbred-line type (BC1, BC2, F2) where the seven CP segregation types and their expected ratios do not apply — use the appropriate -p type instead of CP-specific classification
- Genotypes come from another variant caller rather than the InDelGT genotyping step, so the GQ/MAPQ/MISPCT/depth thresholds and genotype coding the segregation test expects are not in place
- The goal is variant discovery or catalog generation (steps 1–2 of InDelGT), not marker filtering and segregation-type assignment

## Inputs

- InDel genotype matrix for 2 parents and 20 progeny from the InDelGT genotyping step (FASTQ-derived, filtered by GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5)
- parameters.ini with [parameters] section including PVALUE:0.01 and THREADS:20
- Parental InDel catalog of candidate sites (e.g., 50,532 indel sites between the two Populus parents)

## Outputs

- Per-segregation-type text files: aaxab.txt, aaxbc.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt, abxaa (or abxaa.txt)
- Counts of markers per segregation type passing the P > 0.01 Mendelian test (e.g., 1539 ab×aa, 1262 aa×ab, 3 ab×ab, 1 ab×ac = 2805 of 50,532)

## How to apply

Load the InDel genotypes produced by InDelGT's genotyping step (from FASTQ reads of 2 parents and progeny), applying the parameter-file thresholds GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5. For each cataloged InDel site, tally observed parental and progeny genotype counts and test the segregation ratio against the expected Mendelian ratios for the seven CP segregation types using the Statistics::Distributions Perl module, rejecting any site with PVALUE < 0.01. Classify each retained site into its segregation type and write per-type result files (aaxab.txt, aaxbc.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt, abxaa). Parallelize per-site computation with Parallel::ForkManager (THREADS 20). The rationale is that filtering genotypes to Mendelian segregation improves marker accuracy for downstream parental linkage map construction in outbred species where traditional BC1/BC2/F2 expectation frameworks do not apply.

## Related tools

- **InDelGT** (Runs the segregation analysis (step 3) on InDelGT genotype calls; invoked with --nofilter to skip or by default executes the segregation-type analysis) — https://github.com/tongchf/InDelGT
- **Statistics::Distributions** (Perl module used to compute P-values for testing each InDel site's segregation ratio against expected Mendelian ratios)
- **Parallel::ForkManager** (Perl module used to parallelize per-site segregation testing across THREADS 20)
- **Getopt::Long** (Perl module for parsing InDelGT.pl command-line options such as --nofilter and -p CP)

## Examples

```
perl PathToInDelGT/InDelGT.pl -p CP -o directory --nocatalog --nocall
```

## Evaluation signals

- Number of markers in each segregation-type output file matches the reported outcome: 1539 ab×aa, 1262 aa×ab, 3 ab×ab, 1 ab×ac
- Total retained markers (2805) equals the sum of the per-type file line counts and is a subset of the 50,532 cataloged InDel sites
- Every retained site has a segregation test P-value above the PVALUE:0.01 threshold in parameters.ini
- Each retained site's observed parental/progeny genotype pattern is consistent with one of the seven CP segregation types, and no site appears in two type files
- Output directory contains the expected per-type result files (aaxab.txt, aaxbc.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt, abxaa)

## Limitations

- Segregation test rejects sites with PVALUE < 0.01, so genuinely deviating markers (e.g., distorted segregants) are discarded and unavailable for mapping
- Results depend on upstream genotyping thresholds (GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5); large indels (>5 bp) can distort correct read mapping and require higher coverage depth, affecting genotype accuracy feeding this step
- InDelGT is designed for mapping populations (CP, BC1, BC2, F2); only the CP segregation-type framework (ab×aa, aa×ab, ab×ab, ab×cc, aa×bc, ab×ac, ab×cd) is covered by this skill
- No version number is stated for InDelGT, and the per-site P-value computation assumes the Statistics::Distributions module is installed via cpan

## Evidence

- [results] Out of 50,532 indel sites identified between the two parents, 2805 passed Mendelian segregation (P > 0.01), classified into segregation types ab × aa (1539), aa × ab (1262), ab × ab (3), and ab × ac (1).: [evidence span withheld — anchor status: not_found]
- [discussion] InDelGT can not only extract indel genotype data but also perform Mendelian segregation ratio tests on the identified indel sites: "InDelGT can not only extract indel genotype data but also perform Mendelian segregation ratio tests on the identified indel sites"
- [methods] PVALUE:0.01 parameter definition from the parameter file: "# percent of missing genotypes at each InDel
    MISPCT:20"
- [readme] README listing of the seven CP segregation types and per-type output files: [evidence span withheld — anchor status: external_doc]
- [readme] README note on required Perl modules for the segregation analysis: "users need to install several perl modules, including `Parallel::ForkManager`, `Getopt::Long` and `Statistics::Distributions`."

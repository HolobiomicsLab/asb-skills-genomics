---
name: mendelian-segregation-ratio-testing
description: Use when you have InDel genotype calls for two parents and their F1 progeny
  (e.g., 2 parents + 20 progeny from InDelGT genotyping) and need to retain only markers
  following Mendelian segregation before linkage mapping.
when_to_use_negative:
- Input genotypes were not called with InDelGT's genotyping step or lack the parental/progeny
  structure needed for segregation-ratio expectation (e.g., unrelated individuals
  or a single sample).
- You intend to keep segregation-distorted markers (e.g., for distortion analysis)
  rather than filtering them out for linkage map construction.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: Perl pipeline implementing the segregation analysis step; writes per-type
    result files and is invoked with --nofilter to skip this step
  repo: https://github.com/tongchf/InDelGT
- name: Statistics::Distributions
  role: Perl module used to compute the chi-square segregation-ratio test P-value
    per InDel site
- name: Parallel::ForkManager
  role: Perl module used to parallelize per-site segregation tests across THREADS
    (20)
- name: Getopt::Long
  role: Perl module for command-line option parsing of the InDelGT pipeline (e.g.,
    --nofilter, -p CP, -o directory)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:505cccd7f002f5490fb0186204d5cca03c89fa35bbb9fec737f576da7ae182d8
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# mendelian-segregation-ratio-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical filter (step 3 of InDelGT) that tests each cataloged InDel site's observed parental/progeny genotype counts against expected Mendelian segregation ratios for cross-pollination (CP) populations, retaining only sites with P-value >= 0.01 and classifying them into seven segregation-type output files.

## When to use

Use when you have InDel genotype calls for two parents and their F1 progeny (e.g., 2 parents + 20 progeny from InDelGT genotyping) and need to retain only markers following Mendelian segregation before linkage mapping.

## When NOT to use

- Input genotypes were not called with InDelGT's genotyping step or lack the parental/progeny structure needed for segregation-ratio expectation (e.g., unrelated individuals or a single sample).
- You intend to keep segregation-distorted markers (e.g., for distortion analysis) rather than filtering them out for linkage map construction.

## Inputs

- InDel genotype data for 2 parents and 20 progeny from InDelGT genotyping step (filtered at GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5)
- parameters.ini with PVALUE:0.01, PROGENY_NUMBER:20, THREADS:20
- Population type flag (CP, BC1, BC2, or F2) selecting the expected segregation ratios

## Outputs

- Per-segregation-type text files: aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt
- Counts of markers retained per segregation type (e.g., 1539 ab×aa, 1262 aa×ab, 3 ab×ab, 1 ab×ac; 2805 Mendelian total)
- Set of segregation-distorted InDel sites rejected at PVALUE < 0.01

## How to apply

Load the InDel genotypes produced by InDelGT step 2, which were called under parameter-file thresholds (GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5). For each of the cataloged InDel sites (50,532 in the Populus F1 example), tally observed parental and progeny genotype counts and test the segregation ratio against the expected ratio for the appropriate CP segregation type using the Statistics::Distributions Perl module, rejecting sites with PVALUE < 0.01 (set as PVALUE:0.01 in parameters.ini). Classify each retained site into one of seven segregation types (ab×aa, aa×ab, ab×ab, ab×cc, aa×bc, ab×ac, ab×cd) and write per-type result files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt) to the output directory. Parallelize per-site computation with Parallel::ForkManager (THREADS 20) and command-line parsing with Getopt::Long. Validation: on the example Populus data, 2805 of 50,532 indels should pass, split as 1539 ab×aa, 1262 aa×ab, 3 ab×ab, and 1 ab×ac.

## Related tools

- **InDelGT** (Perl pipeline implementing the segregation analysis step; writes per-type result files and is invoked with --nofilter to skip this step) — https://github.com/tongchf/InDelGT
- **Statistics::Distributions** (Perl module used to compute the chi-square segregation-ratio test P-value per InDel site)
- **Parallel::ForkManager** (Perl module used to parallelize per-site segregation tests across THREADS (20))
- **Getopt::Long** (Perl module for command-line option parsing of the InDelGT pipeline (e.g., --nofilter, -p CP, -o directory))

## Examples

```
perl PathToInDelGT/InDelGT.pl -p CP -o directory
```

## Evaluation signals

- Number of markers retained per segregation-type file matches the reported outcome: 1539 ab×aa, 1262 aa×ab, 3 ab×ab, 1 ab×ac.
- Total retained markers equals 2805 of the 50,532 cataloged indels (~5.6% passing rate on the example Populus data).
- All seven expected output files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt) are present in the output directory.
- Every retained site has a segregation-ratio test P-value >= 0.01 (PVALUE threshold in parameters.ini).
- Each retained site is assigned to exactly one of the seven CP segregation types with no cross-file duplicates.

## Limitations

- Only 5.6% of cataloged indels (2805/50,532) passed the Mendelian test in the example, so most sites are discarded; downstream linkage maps rely on the two most abundant segregation types.
- Heterozygote determination requires higher read depth than homozygotes, so mis-genotyped heterozygotes at low depth can distort segregation counts.
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), which can propagate errors into genotype counts feeding the segregation test.

## Evidence

- [readme] seven segregation types and per-type files: [evidence span withheld — anchor status: external_doc]

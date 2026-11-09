---
name: genotype-quality-filtering
description: Use when InDel genotypes have been called for 2 parents and progeny (e.g.,
  an F1 Populus hybrid population with 20 progeny) and non-Mendelian or low-quality
  marker sites must be removed before linkage mapping.
when_to_use_negative:
- When the goal is variant discovery or raw genotype calling rather than filtering
  already-called InDel genotypes — steps 1 and 2 (catalog generation, genotyping)
  handle that
- When the input population type is unsupported by the pipeline's segregation models,
  or when markers should be retained regardless of segregation distortion (the PVALUE:0.01
  cutoff will discard distorted sites)
- When the pipeline is missing Perl modules (Parallel::ForkManager, Getopt::Long,
  Statistics::Distributions) or BWA/SAMtools/BCFtools paths are not configured in
  parameters.ini
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0160
tools:
- name: InDelGT
  role: Pipeline whose third step performs the segregation analysis and filtering
    of InDel genotypes; invoked as a single command with an output directory
  repo: https://github.com/tongchf/InDelGT
- name: Statistics::Distributions
  role: Perl module used to compute the P-value of each InDel site's segregation ratio
    test against expected Mendelian ratios
- name: Parallel::ForkManager
  role: Perl module used to parallelize the per-site segregation testing and filtering
    across THREADS (20) worker processes
- name: Getopt::Long
  role: Perl module for parsing InDelGT.pl command-line options (e.g., -p population
    type, -o output directory, --nofilter)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:cfbd6175dfe45c35737de2ad74b6a0c877583b5e2ce781ddbe24de7d53e16819
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0160
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# genotype-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filters InDel genotypes called across a hybrid population by testing each cataloged InDel site's parental/progeny genotype counts against expected Mendelian segregation ratios (P-value threshold 0.01), after applying genotype-quality thresholds (GQ 30, MAPQ 5, MISPCT 20, HETEROZYGOUS_DEPTH 3, HOMOZYGOUS_DEPTH 5) set in parameters.ini.

## When to use

Use when InDel genotypes have been called for 2 parents and progeny (e.g., an F1 Populus hybrid population with 20 progeny) and non-Mendelian or low-quality marker sites must be removed before linkage mapping.

## When NOT to use

- When the goal is variant discovery or raw genotype calling rather than filtering already-called InDel genotypes — steps 1 and 2 (catalog generation, genotyping) handle that
- When the input population type is unsupported by the pipeline's segregation models, or when markers should be retained regardless of segregation distortion (the PVALUE:0.01 cutoff will discard distorted sites)
- When the pipeline is missing Perl modules (Parallel::ForkManager, Getopt::Long, Statistics::Distributions) or BWA/SAMtools/BCFtools paths are not configured in parameters.ini

## Inputs

- InDel genotype calls for 2 parents and 20 progeny produced by InDelGT step 2 (calling InDel genotypes)
- Parental InDel catalog (50,532 InDel sites identified between the two parents in the example dataset)
- parameters.ini with folders, parameters (GQ, MAPQ, PROGENY_NUMBER, MISPCT, HETEROZYGOUS_DEPTH, HOMOZYGOUS_DEPTH, PVALUE, THREADS), and data files sections
- Population type flag: CP, BC1, BC2, or F2

## Outputs

- Per-segregation-type marker files for CP populations: aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt
- Single genotype file for BC1, BC2, and F2 populations (aaxab.txt, abxaa.txt, or abxab.txt respectively)
- Filtered marker set passing the Mendelian segregation test (2,805 of 50,532 sites in the example), suitable for linkage map construction

## How to apply

Run step 3 of the InDelGT pipeline on the genotype data produced by the genotyping step, using a parameters.ini that sets GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, and THREADS:20. For each of the cataloged InDel sites (50,532 between the two Populus parents in the example), tally observed parental and progeny genotype counts, then test the site's segregation ratio against the expected ratios for the seven cross-pollination (CP) segregation types using the Statistics::Distributions Perl module; reject sites with PVALUE < 0.01. Classify each retained site by segregation type and write per-type output files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt). Parallelize per-site computation with Parallel::ForkManager across THREADS worker processes. In the demonstration, 2,805 of 50,532 sites passed: 1539 ab × aa, 1262 aa × ab, 3 ab × ab, and 1 ab × ac.

## Related tools

- **InDelGT** (Pipeline whose third step performs the segregation analysis and filtering of InDel genotypes; invoked as a single command with an output directory) — https://github.com/tongchf/InDelGT
- **Statistics::Distributions** (Perl module used to compute the P-value of each InDel site's segregation ratio test against expected Mendelian ratios)
- **Parallel::ForkManager** (Perl module used to parallelize the per-site segregation testing and filtering across THREADS (20) worker processes)
- **Getopt::Long** (Perl module for parsing InDelGT.pl command-line options (e.g., -p population type, -o output directory, --nofilter))

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -p CP -o directory &
```

## Evaluation signals

- Marker counts per segregation-type file must match the reported outcome: 1539 ab × aa, 1262 aa × ab, 3 ab × ab, 1 ab × ac
- Total retained markers must equal 2,805 out of 50,532 cataloged InDel sites passing the Mendelian test at P > 0.01
- Every retained site must have a segregation P-value ≥ 0.01 under its assigned CP segregation type, and no site with PVALUE < 0.01 should appear in the output files
- Output must contain up to seven CP files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, abxcd.txt) whose line counts sum to the number of retained markers
- Sites exceeding MISPCT:20 percent missing genotypes, or falling below GQ:30 / MAPQ:5 / depth thresholds (heterozygote ≥ 3, homozygote ≥ 5), must be absent from the filtered output

## Limitations

- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth for reliable genotype calls that feed the filter
- Determining heterozygotes requires higher coverage depth than homozygotes (the pipeline uses HETEROZYGOUS_DEPTH 3 vs HOMOZYGOUS_DEPTH 5), so heterozygote calls are the more error-prone input to the segregation test
- The PVALUE:0.01 cutoff deliberately discards Mendelian-distorted sites, which may remove genuinely segregating markers; only 2,805 of 50,532 cataloged sites survived in the example run

## Evidence

- [discussion] the genotype data generated from InDelGT are filtered to follow the Mendelian segregation ratio, which could further improve the data accuracy overall: "the genotype data generated from InDelGT are filtered to follow the Mendelian segregation ratio, which could further improve the data accuracy overall"
- [methods] # the minimum score of InDel genotypes GQ:30: "# the minimum score of InDel genotypes
    GQ:30"
- [readme] the minimum P-value allowed for testing the segregation ratio at a InDel site and the number of threads used for parallel computing: [evidence span withheld — anchor status: external_doc]
- [readme] up to seven files for CP as aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, and abxcd.txt: "up to seven files for CP as aaxab.txt, aaxbc.txt, abxaa.txt, abxcc, abxab.txt, abxac.txt, and abxcd.txt"
- [abstract] The algorithm is implemented in three steps: generating an indel catalog, calling indel genotypes, and analyzing indel segregation.: "The InDelGT algorithm is implemented in three steps: generating an indel catalog, calling indel genotypes, and analyzing indel segregation."

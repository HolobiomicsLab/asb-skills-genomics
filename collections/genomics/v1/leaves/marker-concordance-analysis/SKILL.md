---
name: marker-concordance-analysis
description: Use when sequencing-based genotype calls (e.g., InDelGT output from BWA/SAMtools/BCFtools)
  need experimental validation against an independent assay such as PAGE gels, with
  concordance computed only over amplified genotype positions.
when_to_use_negative:
- The genotype data do not come from a bi-parental mapping population supported by
  InDelGT (BC1, BC2, F2 inbred lines, or CP hybrid population in outbred species)
- No independent orthogonal genotype calls (e.g., PAGE gel data) exist for the same
  loci and individuals, so no concordance comparison is possible
- Large indels (>5 bp) at the target loci lack sufficient coverage depth — the article
  notes large indels complicate or distort correct read mapping and require higher
  coverage, which can confound concordance interpretation
edam_operation: http://edamontology.org/operation_3196
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: InDelGT
  role: Pipeline whose indel genotype calls (extracted from next-generation sequencing
    data) are the sequencing side of the concordance comparison
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns the paired-end reads of parents and progeny to the reference genome
    to produce alignments underlying InDelGT genotype calls
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Sorts alignments (with BCFtools) during InDelGT's internal variant-calling
    step feeding the genotypes being validated
- name: BCFtools 1.9
  role: Performs variant calling inside the InDelGT pipeline prior to genotype extraction
    and PAGE concordance assessment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:bdec6c5efd26218355d4866ef21ea6dce5ef6d6c890d50ce49d2c71b8ec099fc
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
  edam_operation: http://edamontology.org/operation_3196
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# marker-concordance-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-compare sequencing-derived InDelGT indel genotype calls against orthogonal PAGE gel-derived genotypes locus-by-locus and progeny-by-progeny, computing concordance restricted to successfully amplified positions (reported as 86/90, ~95%, for 8 indels in 12 Populus progeny).

## When to use

Use when sequencing-based genotype calls (e.g., InDelGT output from BWA/SAMtools/BCFtools) need experimental validation against an independent assay such as PAGE gels, with concordance computed only over amplified genotype positions.

## When NOT to use

- The genotype data do not come from a bi-parental mapping population supported by InDelGT (BC1, BC2, F2 inbred lines, or CP hybrid population in outbred species)
- No independent orthogonal genotype calls (e.g., PAGE gel data) exist for the same loci and individuals, so no concordance comparison is possible
- Large indels (>5 bp) at the target loci lack sufficient coverage depth — the article notes large indels complicate or distort correct read mapping and require higher coverage, which can confound concordance interpretation

## Inputs

- InDelGT CP segregation-type genotype output files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc.txt, abxab.txt, abxac.txt, abxcd.txt)
- PAGE gel-derived genotype calls for the same marker loci and progeny (e.g., Table 2 of the article)
- List of validation loci (eight randomly selected indels) and validation progeny (12 of the 20 progeny)

## Outputs

- indel_validation_concordance.csv with per-locus/per-progeny agreement labels (consistent vs discordant)
- Concordance count and percentage over amplified positions (86/90 ≈ 95%)

## How to apply

First run the InDelGT pipeline on the example data (figshare dataset 15131649: reference genome plus paired-end reads of 2 parents and 20 progeny) with parameters.ini set to GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20, using BWA 0.7.17 for alignment, SAMtools 1.9 for sorting, and BCFtools 1.9 for variant calling. Parse the resulting CP segregation-type genotype files (aaxab.txt, aaxbc.txt, abxaa.txt, abxcc.txt, abxab.txt, abxac.txt, abxcd.txt) to extract calls at the eight Table 2 loci for the 12 validation progeny. Load the PAGE gel genotype calls for the same eight loci and 12 progeny and compare call-by-call; because all but five genotypes were amplified, restrict agreement counting to the 90 successfully amplified positions rather than all 96 possible genotype positions. Compute the concordance count and percentage (expected: 86/90 ≈ 95%) and record each locus-progeny pair as consistent or discordant. Rationale: InDelGT genotype data are additionally filtered to follow the Mendelian segregation ratio, so concordance validates both the genotype calls and the filtering logic against a gel-based ground truth.

## Related tools

- **InDelGT** (Pipeline whose indel genotype calls (extracted from next-generation sequencing data) are the sequencing side of the concordance comparison) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns the paired-end reads of parents and progeny to the reference genome to produce alignments underlying InDelGT genotype calls) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Sorts alignments (with BCFtools) during InDelGT's internal variant-calling step feeding the genotypes being validated)
- **BCFtools 1.9** (Performs variant calling inside the InDelGT pipeline prior to genotype extraction and PAGE concordance assessment)

## Examples

```
perl PathToInDelGT/InDelGT.pl -p CP -o directory
```

## Evaluation signals

- Concordance over amplified positions reproduces 86/90 (~95%) agreement between InDelGT and PAGE genotypes for the 8 loci × 12 progeny
- Exactly 90 of 96 possible genotype positions are counted, matching the article's statement that all but five genotypes were amplified
- Each of the 96 locus-progeny pairs is explicitly labeled consistent or discordant in indel_validation_concordance.csv
- Upstream InDelGT run used the documented parameters.ini thresholds (GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20) with BWA 0.7.17, SAMtools 1.9, BCFtools 1.9
- Parsed calls are drawn only from the seven CP segregation-type output files, not from intermediate VCF/BAM artifacts

## Limitations

- Five of 96 genotypes failed amplification and are excluded from the concordance denominator, so concordance is conditional on PAGE assay success
- No version number is stated for InDelGT, and the coverage-depth supporting detail is in Appendix S1, which is not included in the provided text
- Large indels (>5 bp) can complicate or distort correct read mapping relative to small indels (1–5 bp), potentially biasing sequencing-derived calls used in the comparison
- Determining heterozygotes requires higher coverage depth than homozygotes, so discordances at low-depth loci may reflect coverage rather than true call errors

## Evidence

- [results] all but five genotypes were amplified, and 86 (95%) of the amplified genotypes were consistent with the InDelGT genotyping results.: [evidence span withheld — anchor status: not_found]
- [other] In the PAGE validation of eight indels across 12 progeny, all but five genotypes were amplified, and 86 (95%) of the amplified genotypes were consistent: [evidence span withheld — anchor status: not_found]

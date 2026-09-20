---
name: homozygous-allele-verification-across-catalog-span
description: Use when calling progeny indel genotypes from VCF evidence at known parental
  InDel catalog sites and needing to distinguish homozygous from heterozygous calls
  without direct VCF support for the homozygous state.
when_to_use_negative:
- When the goal is general SNV/indel variant discovery from VCFs — this skill only
  genotypes sites already present in a parental InDel catalog.
- When reads lack sufficient depth (below HOMOZYGOUS_DEPTH:5 across the full span)
  — homozygous calls cannot be verified and sites will be missing.
- For large indels (>5 bp) at low coverage, where the article warns read mapping is
  complicated or distorted and higher coverage depth is required.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: implements the homozygous span-verification genotyping logic in the progeny_genotyping
    module
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: aligns progeny paired-end FASTQ reads to the reference genome; alignments
    below MAPQ:5 are discarded before genotyping
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: processes alignments for variant-evidence extraction at catalog sites
  repo: http://samtools.sourceforge.net/
- name: BCFtools 1.9
  role: extracts VCF records supporting heterozygous calls at parental-catalog InDel
    sites
  repo: http://samtools.sourceforge.net/
- name: Parallel::ForkManager
  role: parallelizes per-progeny genotype calling across THREADS:20 samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:73e2c3e0b3f95704124d50c5f8a51f04f222db565321e427ebaf2ac78b92eefd
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
  - Getopt::Long
  - InDelGT
  - Parallel::ForkManager
  - SAMtools
  - SAMtools 1.9
  - Statistics::Distributions
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# homozygous-allele-verification-across-catalog-span

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In the InDelGT genotyping step, a homozygous indel genotype is called at a parental catalog site only when the alternate allele is absent at every position across the site's full start-to-end span (as recorded in the parental InDel catalog) at read depth of at least HOMOZYGOUS_DEPTH:5, with genotypes below GQ:30 discarded.

## When to use

Use when calling progeny indel genotypes from VCF evidence at known parental InDel catalog sites and needing to distinguish homozygous from heterozygous calls without direct VCF support for the homozygous state.

## When NOT to use

- When the goal is general SNV/indel variant discovery from VCFs — this skill only genotypes sites already present in a parental InDel catalog.
- When reads lack sufficient depth (below HOMOZYGOUS_DEPTH:5 across the full span) — homozygous calls cannot be verified and sites will be missing.
- For large indels (>5 bp) at low coverage, where the article warns read mapping is complicated or distorted and higher coverage depth is required.

## Inputs

- Parental InDel catalog with per-site start and end positions
- Progeny VCF records at catalog InDel sites (from BCFtools 1.9 over BWA 0.7.17 alignments, MAPQ >= 5)
- parameters.ini with HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, GQ:30, THREADS:20

## Outputs

- Per-sample genotype files recording InDel site positions, allele nucleotides, and genotypes
- Genotypes passing the minimum InDel genotype score (GQ:30), later filtered by Mendelian segregation tests at PVALUE:0.01

## How to apply

After aligning each progeny's paired-end FASTQ reads to reference.fasta with BWA 0.7.17 (retaining MAPQ >= 5) and extracting variant evidence at parental-catalog sites with SAMtools 1.9 and BCFtools 1.9, assign heterozygous genotypes from VCF records when depth reaches HETEROZYGOUS_DEPTH:3. Because VCF records directly support only heterozygous calls, confirm a homozygous indel genotype by checking allele absence across the entire catalog InDel start-to-end span at depth of at least HOMOZYGOUS_DEPTH:5 — every position from the site's recorded start to end must be homozygous. Discard any genotype scoring below GQ:30. Parallelize calling across all progeny (e.g., 20 samples) with Parallel::ForkManager using THREADS:20, then write per-sample genotype files recording InDel site positions, allele nucleotides, and genotypes. Rationale: requiring full-span absence prevents partial-support false homozygous calls, and the higher homozygous depth threshold complements the fact that heterozygote determination requires more coverage than homozygote determination.

## Related tools

- **InDelGT** (implements the homozygous span-verification genotyping logic in the progeny_genotyping module) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (aligns progeny paired-end FASTQ reads to the reference genome; alignments below MAPQ:5 are discarded before genotyping) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (processes alignments for variant-evidence extraction at catalog sites) — http://samtools.sourceforge.net/
- **BCFtools 1.9** (extracts VCF records supporting heterozygous calls at parental-catalog InDel sites) — http://samtools.sourceforge.net/
- **Parallel::ForkManager** (parallelizes per-progeny genotype calling across THREADS:20 samples)

## Examples

```
perl PathToInDelGT/InDelGT.pl --nocatalog  # run genotype-calling stage from work dir with parameters.ini (HOMOZYGOUS_DEPTH:5, GQ:30, THREADS:20)
```

## Evaluation signals

- Every emitted genotype meets the minimum InDel genotype score (GQ:30).
- Homozygous calls are supported by allele absence at every position from the catalog site's start to end at depth >= HOMOZYGOUS_DEPTH:5.
- Heterozygous calls appear in the progeny VCF records with depth >= HETEROZYGOUS_DEPTH:3.
- Downstream segregation-type genotype files (e.g., aaxab.txt for BC1/CP) conform to Mendelian segregation tested at PVALUE:0.01.
- Missing genotypes per InDel site stay within MISPCT:20 before segregation analysis.

## Limitations

- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1-5 bp), requiring higher coverage depth.
- Determining heterozygotes requires a higher coverage depth than homozygotes, so depth thresholds cannot be set uniformly.
- Genotype accuracy ultimately depends on the Mendelian segregation filter, which improves data accuracy overall but discards non-conforming markers.

## Evidence

- [full_text] InDelGT calls a homozygous indel genotype at a parental catalog site only when every position from the site's start to end is homozygous, using the site start and end positions recorded in the parental indel catalog.: [evidence span withheld — anchor status: not_found]
- [full_text] Assign heterozygous genotypes from the VCF records when depth reaches HETEROZYGOUS_DEPTH:3, confirm homozygous genotypes by checking allele absence across the full catalog InDel start-to-end span at depth of at least HOMOZYGOUS_DEPTH:5, and discard genotypes scoring below GQ:30.: [evidence span withheld — anchor status: not_found]
- [methods] the minimum score of InDel genotypes
    GQ:30: "# the minimum score of InDel genotypes
    GQ:30"
- [discussion] The determination of heterozygotes requires a higher coverage depth than that of homozygotes (McKernan et al., 2009): "The determination of heterozygotes requires a higher coverage depth than that of homozygotes (McKernan et al., 2009)"
- [discussion] Compared with small indels (1–5 bp), large indels (>5 bp) will complicate or distort the correct mapping of reads: "Compared with small indels (1–5 bp), large indels (>5 bp) will complicate or distort the correct mapping of reads"

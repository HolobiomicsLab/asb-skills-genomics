---
name: short-read-indel-variant-calling
description: Use when you have parental paired-end FASTQ reads (e.g. EcoRI RAD-seq)
  and a reference FASTA, and need a catalog of indel sites between two parents of
  a mapping population for downstream genotyping.
when_to_use_negative:
- The goal is SNP calling or whole-genome deep-coverage variant discovery rather than
  indel genotyping at restriction-site loci for linkage mapping
- You only have a single sample rather than two parents of a segregating (BC1, BC2,
  F2, or CP hybrid) population
- A parental indel catalog already exists and only downstream genotyping is needed
  (use the --nocatalog option to skip this step)
edam_operation: http://edamontology.org/operation_0452
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: InDelGT
  role: Integrated Perl pipeline that orchestrates parental indel catalog generation
    from aligned parental reads
  repo: https://github.com/tongchf/InDelGT
- name: BWA 0.7.17
  role: Aligns parental paired-end reads to the reference genome
  repo: https://github.com/lh3/bwa/releases/tag/v0.7.17
- name: SAMtools 1.9
  role: Processes BWA alignments of parental reads prior to indel calling
  repo: http://samtools.sourceforge.net/
- name: BCFtools 1.9
  role: Calls indels at parental restriction-site loci from processed alignments
  repo: http://samtools.sourceforge.net/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/aps3.11499
    title: 'InDelGT: An integrated pipeline for extracting indel genotypes for genetic
      mapping in a hybrid population using next‐generation sequencing data'
  source_content_hash: sha256:fbdedc06024c226d394ed70e83612d36f2dd48727a7d1ccef4907f5ae132cfce
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
  edam_operation: http://edamontology.org/operation_0452
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# short-read-indel-variant-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Call indel variants between two parents from paired-end RAD-seq reads aligned to a reference genome using the InDelGT pipeline (BWA 0.7.17 + SAMtools 1.9/BCFtools 1.9), producing a merged parental indel catalog (parent.cls), e.g. 50,532 indel sites between P. deltoides and P. simonii against the P. trichocarpa v4.0 reference.

## When to use

Use when you have parental paired-end FASTQ reads (e.g. EcoRI RAD-seq) and a reference FASTA, and need a catalog of indel sites between two parents of a mapping population for downstream genotyping.

## When NOT to use

- The goal is SNP calling or whole-genome deep-coverage variant discovery rather than indel genotyping at restriction-site loci for linkage mapping
- You only have a single sample rather than two parents of a segregating (BC1, BC2, F2, or CP hybrid) population
- A parental indel catalog already exists and only downstream genotyping is needed (use the --nocatalog option to skip this step)

## Inputs

- Reference genome FASTA (e.g. P. trichocarpa v4.0 reference.fasta)
- Paired-end FASTQ reads (R1/R2) for parent 1 (e.g. P. deltoides)
- Paired-end FASTQ reads (R1/R2) for parent 2 (e.g. P. simonii)
- parameters.ini configuration file with [folders], [parameters], and [data files] sections

## Outputs

- Merged parental indel catalog file parent.cls (~50,532 indel sites in the Populus case study)

## How to apply

Install BWA 0.7.17, SAMtools 1.9 with BCFtools 1.9, and the Perl modules Parallel::ForkManager, Getopt::Long, and Statistics::Distributions via cpan. Fill in the parameters.ini file: the [folders] section with software install paths, the [parameters] section with thresholds such as GQ:30, MAPQ:5, MISPCT:20, HETEROZYGOUS_DEPTH:3, HOMOZYGOUS_DEPTH:5, PVALUE:0.01, THREADS:20, and the [data files] section pointing REFERENCE_FILE to reference.fasta and PARENT1/PARENT2 to the parental R1/R2 FASTQ files. From the work directory, run the first InDelGT step (parental InDel catalog generation) in the background with 'nohup perl PathToInDelGT/InDelGT.pl -o directory &' so the long run is not interrupted. Under the hood, parental paired-end reads are aligned to the reference with BWA and processed with SAMtools/BCFtools to call indels at parental restriction-site loci, merging both parents into one catalog. Validate that the -o output directory contains parent.cls with approximately the expected number of indel sites (50,532 in the Populus case study).

## Related tools

- **InDelGT** (Integrated Perl pipeline that orchestrates parental indel catalog generation from aligned parental reads) — https://github.com/tongchf/InDelGT
- **BWA 0.7.17** (Aligns parental paired-end reads to the reference genome) — https://github.com/lh3/bwa/releases/tag/v0.7.17
- **SAMtools 1.9** (Processes BWA alignments of parental reads prior to indel calling) — http://samtools.sourceforge.net/
- **BCFtools 1.9** (Calls indels at parental restriction-site loci from processed alignments) — http://samtools.sourceforge.net/

## Examples

```
nohup perl PathToInDelGT/InDelGT.pl -o directory &
```

## Evaluation signals

- The -o output directory contains the merged parental indel catalog file parent.cls
- parent.cls contains approximately 50,532 indel sites, the count reported for the Populus case study
- Alignment and calling used the configured thresholds (GQ:30, MAPQ:5) from parameters.ini
- Both parents' reads were aligned and their indel calls merged into a single catalog rather than two separate catalogs
- The full run was launched with nohup in the background so it was not interrupted (the run can take hours to days)

## Limitations

- The full catalog-generation run can take hours to days, so it must be launched in the background (e.g. nohup) to avoid interruption
- Large indels (>5 bp) complicate or distort correct read mapping compared with small indels (1–5 bp), requiring higher coverage depth
- Determining heterozygotes requires higher coverage depth than homozygotes
- Coverage is restricted to restriction sites (EcoRI here), covering at most ~20% of the poplar genome

## Evidence

- [results] Applying InDelGT to the F1 hybrid Populus population, a total of 50,532 indel sites were identified between the two parents, of which 2805 followed Mendelian segregation ratios (P > 0.01) in the progeny.: [evidence span withheld — anchor status: not_found]
- [methods] Under the hood, align the parental paired-end reads to the reference genome with BWA and process the alignments with SAMtools/BCFtools to call indels at parental restriction-site loci, merging both parents into a single catalog.: [evidence span withheld — anchor status: not_found]

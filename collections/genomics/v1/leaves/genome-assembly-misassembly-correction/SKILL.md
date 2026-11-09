---
name: genome-assembly-misassembly-correction
description: Use when a draft genome assembly (e.g., an ABySS or long-read assembler
  draft FASTA) plus a matching long-read dataset (e.g., FASTQ/FASTA such as SRR10028109)
  is available and misassembly detection/cutting is needed without full read alignments.
when_to_use_negative:
- The goal is scaffolding (ordering/orienting contigs and gap-filling) rather than
  cutting misassemblies — use ntLink's Basic Protocols 1–3 instead.
- Only a polished, reference-validated assembly is available with no long-read data
  to re-map, so no mapping-discrepancy windows can be computed.
- The draft was not built from (or is not compatible with) the available long reads;
  ntLink mappings require long reads that span the assembly contigs.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Tigmint-long
  role: Identifies windows of mapping discrepancy and cuts the draft assembly at putative
    misassemblies
  repo: https://github.com/BirolLab/tigmint
- name: ntLink
  role: Computes lightweight minimizer-based long-read to contig mappings used in
    place of read alignments
  repo: https://github.com/bcgsc/ntLink
- name: QUAST
  role: Evaluates corrected vs. baseline assemblies for contiguity and correctness
    (Table 6 comparison)
  repo: https://github.com/ablab/quast
- name: sra-tools
  role: Downloads the long-read dataset (e.g., SRR10028109) from SRA
  repo: https://github.com/ncbi/sra-tools
- name: ABySS
  role: Source of the example draft assembly (celegans-abyss) corrected in Alternate
    Protocol 2
  repo: https://github.com/BirolLab/abyss
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/cpz1.733
    title: 'ntLink: A Toolkit for <i>De Novo</i> Genome Assembly Scaffolding and Mapping
      Using Long Reads'
  source_content_hash: sha256:02de635659ec04434f85126b73c1588f3b6ad11fedd1048fa5aa58e17712081d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BirolLab/tigmint
  tools:
  - ABySS
  - QUAST
  - Tigmint-long
  - ntLink
  - sra-tools
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# genome-assembly-misassembly-correction

## Summary

Corrects putative misassemblies in a long-read genome assembly draft by running Tigmint-long on minimizer-based ntLink mappings of long reads to contigs, cutting the draft at windows of mapping discrepancy, and validating the corrected FASTA with QUAST.

## When to use

Use when a draft genome assembly (e.g., an ABySS or long-read assembler draft FASTA) plus a matching long-read dataset (e.g., FASTQ/FASTA such as SRR10028109) is available and misassembly detection/cutting is needed without full read alignments.

## When NOT to use

- The goal is scaffolding (ordering/orienting contigs and gap-filling) rather than cutting misassemblies — use ntLink's Basic Protocols 1–3 instead.
- Only a polished, reference-validated assembly is available with no long-read data to re-map, so no mapping-discrepancy windows can be computed.
- The draft was not built from (or is not compatible with) the available long reads; ntLink mappings require long reads that span the assembly contigs.

## Inputs

- Draft genome assembly FASTA (e.g., ABySS draft celegans-abyss)
- Long-read sequencing dataset (gzipped FASTQ or FASTA, e.g., C. elegans SRR10028109 from SRA)
- ntLink (BirolLab/ntLink) and Tigmint (BirolLab/tigmint) installed via tigmint-make, with btllib >= 1.7.1

## Outputs

- Corrected assembly FASTA with cuts at putative misassemblies (celegans-abyss.cut500.tigmint.fa)
- ntLink minimizer-based long-read to contig mappings used by Tigmint-long
- QUAST comparison of corrected vs. uncorrected assemblies (contiguity and correctness metrics, cf. Table 6)

## How to apply

Obtain the draft assembly (e.g., celegans-abyss) and the C. elegans long-read dataset SRR10028109 (gzipped FASTQ or FASTA, downloaded with ncbi/sra-tools). Run tigmint-make tigmint-long with draft=celegans-abyss, reads=SRR10028109, mapping=ntLink, t=5 and span=2, so ntLink computes lightweight minimizer-based long-read-to-contig mappings in place of read alignments. Tigmint-long then identifies windows of mapping discrepancy and cuts the draft at putative misassemblies, producing the corrected FASTA celegans-abyss.cut500.tigmint.fa. Rationale: ntLink's minimizer-based mappings (sketch targets and reads, map minimizers, find joins suggested by long-read evidence) are resource-efficient substitutes for alignment-based misassembly detection. Validate by running QUAST (ablab/quast) on both the corrected assembly and the uncorrected baseline and comparing contiguity/correctness metrics as in Table 6 of the article.

## Related tools

- **Tigmint-long** (Identifies windows of mapping discrepancy and cuts the draft assembly at putative misassemblies) — https://github.com/BirolLab/tigmint
- **ntLink** (Computes lightweight minimizer-based long-read to contig mappings used in place of read alignments) — https://github.com/bcgsc/ntLink
- **QUAST** (Evaluates corrected vs. baseline assemblies for contiguity and correctness (Table 6 comparison)) — https://github.com/ablab/quast
- **sra-tools** (Downloads the long-read dataset (e.g., SRR10028109) from SRA) — https://github.com/ncbi/sra-tools
- **ABySS** (Source of the example draft assembly (celegans-abyss) corrected in Alternate Protocol 2) — https://github.com/BirolLab/abyss

## Examples

```
tigmint-make tigmint-long draft=celegans-abyss reads=SRR10028109 mapping=ntLink t=5 span=2
```

## Evaluation signals

- tigmint-make tigmint-long completes with mapping=ntLink and produces the expected corrected FASTA (celegans-abyss.cut500.tigmint.fa) named with the cut distance (500).
- The corrected assembly differs from the draft by cuts at putative misassemblies (sequence is split at discrepancy windows), not by reordering or reassembling contigs.
- QUAST metrics for the corrected assembly show improved contiguity/correctness consistent with the reported Table 6 comparison against the uncorrected baseline.
- ntLink's minimizer-based mappings are used instead of read alignments (no minimap2/BAM alignments required), reflecting the alignment-free design.
- Parameters t=5 and span=2 match the published Alternate Protocol 2 settings.

## Limitations

- Cutting removes sequence at putative misassemblies, so the corrected assembly may be more fragmented than the draft; gains must be judged on correctness/contiguity trade-offs via QUAST (Table 6).
- Quality of misassembly detection depends on ntLink's minimizer-based mappings being informative — coverage or read quality problems in the long-read dataset (e.g., SRR10028109) reduce discrepancy detection.
- The protocol is demonstrated on the C. elegans draft/data pair; other genomes may require tuning t and span.

## Evidence

- [abstract] Alternate Protocol 2: Using ntLink mappings for genome assembly correction with Tigmint-long: "Alternate Protocol 2: Using ntLink mappings for genome assembly correction with Tigmint-long"
- [abstract] minimizer mappings for downstream misassembly detection: "the lightweight minimizer-based mappings that enable ntLink scaffolding can also be utilized for other downstream applications, such as misassembly detection"
- [methods] minimizer sketching and mapping workflow: "1. Compute ordered minimizer sketches of the input target assembly and long reads"
- [methods] scaffold graph construction from long-read evidence: "Find contig pairs, where joins are suggested by the long-read mapping evidence"
- [abstract] resource-efficient long-read scaffolding claim: "ntLink is a flexible and resource-efficient genome scaffolding tool that utilizes long-read sequencing data to improve upon draft genome assemblies"

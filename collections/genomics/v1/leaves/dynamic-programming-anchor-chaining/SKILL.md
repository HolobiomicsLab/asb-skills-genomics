---
name: dynamic-programming-anchor-chaining
description: Use when mapping long noisy reads (PacBio/Oxford Nanopore FASTA/FASTQ)
  to a reference genome and inversion structural variants must be detected via anchor
  chaining rather than standard co-linear alignment.
when_to_use_negative:
- Reads are short Illumina reads — invMap targets long noisy PacBio/Oxford Nanopore
  reads that can span SV breakpoints
- The task is generic co-linear alignment with no interest in inversion structural
  variants, where standard mappers suffice
- Available RAM is below 16G when mapping to large reference genomes, which invMap
  requires
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3577
tools:
- name: invMap
  role: Performs seed lookup, dynamic-programming anchor chaining, nonlinear-to-linear
    anchor transformation for inversion detection, and KSW2-based gap filling
  repo: https://github.com/zhang134/invMap.git
- name: KSW2
  role: Fills remaining gaps in the chained alignment within invMap's pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:adc32e5f9d72c9d466b1ae0b66f8ac5b1bf19cca41a6b1c192893c4dc8d57e26
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - KSW2
  - invMap
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dynamic-programming-anchor-chaining

## Summary

A seed-anchoring and dynamic-programming chaining strategy used by invMap to locate the aligned region of long noisy PacBio/Oxford Nanopore reads against a reference genome, followed by non-linear-to-linear anchor transformation to expose inversion chains.

## When to use

Use when mapping long noisy reads (PacBio/Oxford Nanopore FASTA/FASTQ) to a reference genome and inversion structural variants must be detected via anchor chaining rather than standard co-linear alignment.

## When NOT to use

- Reads are short Illumina reads — invMap targets long noisy PacBio/Oxford Nanopore reads that can span SV breakpoints
- The task is generic co-linear alignment with no interest in inversion structural variants, where standard mappers suffice
- Available RAM is below 16G when mapping to large reference genomes, which invMap requires

## Inputs

- Reference genome in FASTA format (genome.fa)
- Long noisy sequencing reads in FASTA or FASTQ format (PacBio or Oxford Nanopore)
- Minimizer hash-table index built from the reference genome

## Outputs

- Mapped alignments in SAM format (invMap_aligned.sam) containing aligned records and candidate inversion SV calls

## How to apply

Build a minimizer hash-table index of the reference genome (genome.fa) for rapid seed lookup. For each long noisy read, retrieve seed anchors from the index and apply invMap's specifically designed dynamic-programming scoring method for chaining to locate the aligned region. Then examine the remaining anchors inside the located aligned region, transforming non-co-linear (nonlinear) anchors into co-linear (linear) form with boundary region extension, so that inversion chains — including small inversions — become detectable. Finally, fill remaining gaps in the chained alignment with the KSW2 algorithm and emit the result as invMap_aligned.sam in SAM format.

## Related tools

- **invMap** (Performs seed lookup, dynamic-programming anchor chaining, nonlinear-to-linear anchor transformation for inversion detection, and KSW2-based gap filling) — https://github.com/zhang134/invMap.git
- **KSW2** (Fills remaining gaps in the chained alignment within invMap's pipeline)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap_aligned.sam exists, is parseable SAM, and contains aligned records
- Inversion events, including small ones, are reported as candidate SV calls in the alignments
- Chained alignments locate the correct aligned region per read as judged by the specifically designed DP scoring method
- On simulated datasets, aligned-region and SV-calling accuracy for inversions exceeds competing methods across genomes and coverages
- On the NA12878 real human dataset, more candidate inversion variant calls are found than competing methods

## Limitations

- invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Noisy long reads mean seed anchors can be non-co-linear; without the nonlinear-to-linear transformation with boundary region extension, small inversions may be missed
- No changelog found for the repository, so version-to-version behavioral changes are not documented (ver1.0.0, 2023/09/08)

## Evidence

- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [full_text] Nonlinear-to-linear anchor modification enabling discovery of small inversions: [evidence span withheld — anchor status: not_found]
- [readme] Compile and run steps from README: "Step 1: compile codes using make command"
- [intro] RAM requirement: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."
- [abstract] Long reads span SV breakpoints: "Longer reads produced by PacBio or Oxford Nanopore sequencers could more frequently span the breakpoints of structural variations (SVs) than shorter reads"

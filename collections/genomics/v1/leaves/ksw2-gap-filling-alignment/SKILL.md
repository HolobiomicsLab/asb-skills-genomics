---
name: ksw2-gap-filling-alignment
description: Use when chaining anchors from long noisy PacBio or Oxford Nanopore reads
  leaves unaligned gaps between chained anchors and a full read-to-reference alignment
  is needed for inversion SV calling.
when_to_use_negative:
- Reads are short (Illumina-style) — invMap is designed for long noisy PacBio/Oxford
  Nanopore reads that frequently span SV breakpoints
- The task is de novo assembly or variant calling without a reference genome — the
  pipeline requires a reference genome.fa to index and map against
- Machine has less than 16G RAM and the target is a large reference genome — invMap
  requires a minimum of 16G of RAM for mapping to large reference genomes
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3175
tools:
- name: invMap
  role: Primary pipeline that builds the minimizer index, chains anchors, detects
    inversion chains, and invokes KSW2 to fill remaining gaps before writing SAM output
  repo: https://github.com/zhang134/invMap.git
- name: KSW2
  role: Alignment algorithm used to fill remaining gaps in the chained alignment (step
    4 of the invMap pipeline)
- name: PacBio
  role: Source of the long noisy reads that the gap-filled alignments are computed
    for
- name: Oxford Nanopore
  role: Alternative source of long noisy reads that the gap-filled alignments are
    computed for
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad726
    title: 'invMap: a sensitive mapping tool for long noisy reads with inversion structural
      variants'
  source_content_hash: sha256:48435598cf18e27532c4cbfb24a313593e04d2d014297eb3b54342c6dcd2ebb8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhang134/invMap.git
  tools:
  - KSW2
  - Oxford Nanopore
  - PacBio
  - invMap
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3175
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# ksw2-gap-filling-alignment

## Summary

The final stage of the invMap four-step long-read mapping pipeline in which remaining unchained gaps between anchors are filled using the KSW2 alignment algorithm, producing complete per-read alignments reported in SAM format.

## When to use

Use when chaining anchors from long noisy PacBio or Oxford Nanopore reads leaves unaligned gaps between chained anchors and a full read-to-reference alignment is needed for inversion SV calling.

## When NOT to use

- Reads are short (Illumina-style) — invMap is designed for long noisy PacBio/Oxford Nanopore reads that frequently span SV breakpoints
- The task is de novo assembly or variant calling without a reference genome — the pipeline requires a reference genome.fa to index and map against
- Machine has less than 16G RAM and the target is a large reference genome — invMap requires a minimum of 16G of RAM for mapping to large reference genomes

## Inputs

- Reference genome in FASTA format (genome.fa)
- Long noisy sequencing reads in FASTA or FASTQ format (PacBio or Oxford Nanopore)
- Chained anchor alignments with remaining gaps (intermediate from invMap chaining)
- Minimizer hash-table index of the reference genome (built in step 1)

## Outputs

- Gap-filled full read-to-reference alignments (invMap_aligned.sam in SAM format)
- Candidate inversion SV calls reported within the SAM alignments

## How to apply

After building a minimizer hash-table index of the reference genome and chaining seed anchors with invMap's dynamic-programming scoring method, examine remaining anchors inside the located aligned region, transforming non-co-linear (nonlinear) anchors to co-linear form with boundary region extension to detect potential inversion chains. Any gaps that remain in the chained alignment are then filled with the KSW2 alignment algorithm (step 4 of the four-step pipeline: index, main chain, inversion chain, alignment). Run the compiled invMap binary on the genome FASTA and reads FASTA/FASTQ, directing the final mapped alignments to invMap_aligned.sam in SAM format. Validate that invMap_aligned.sam exists, is parseable SAM, and that reported inversions constitute candidate SV calls, consistent with the paper's claim that invMap locates aligned regions and calls inversion SVs more accurately than competing methods.

## Related tools

- **invMap** (Primary pipeline that builds the minimizer index, chains anchors, detects inversion chains, and invokes KSW2 to fill remaining gaps before writing SAM output) — https://github.com/zhang134/invMap.git
- **KSW2** (Alignment algorithm used to fill remaining gaps in the chained alignment (step 4 of the invMap pipeline))
- **PacBio** (Source of the long noisy reads that the gap-filled alignments are computed for)
- **Oxford Nanopore** (Alternative source of long noisy reads that the gap-filled alignments are computed for)

## Examples

```
./invMap genome.fa reads.fa > invMap_aligned.sam
```

## Evaluation signals

- invMap_aligned.sam exists and is parseable as valid SAM format
- SAM output contains aligned records for input reads with gaps filled by KSW2 (no unaligned stretches within the located aligned region)
- Inversion events, including small ones, are reported as candidate SV calls, reflecting the non-co-linear anchor transformation step
- Mapped alignments on simulated datasets across different genomes and sequencing coverages show more accurately located aligned regions and SV calls than competing methods
- On the NA12878 real human genome sequencing dataset, more candidate inversion variant calls are found than by competing methods

## Limitations

- Requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes
- Requires approximately 800 M of available disk space on the drive or partition
- Input must be in FASTA or FASTQ format; other formats are not supported
- No changelog found in the repository, so version-specific behavior changes are undocumented

## Evidence

- [other] Fill remaining gaps in the chained alignment using the KSW2 alignment algorithm.: [evidence span withheld — anchor status: not_found]
- [abstract] invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions: "invMap first locates the aligned region with a specifically designed scoring method for chaining, then checks the remaining anchors in the aligned region to discover potential inversions"
- [readme] The final mapping result is invMap_aligned.sam in SAM format.: "The final mapping result is invMap_aligned.sam in SAM format."
- [intro] invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes.: "invMap requires a minimum of 16G of RAM for mapping sequence datasets to large reference genomes."

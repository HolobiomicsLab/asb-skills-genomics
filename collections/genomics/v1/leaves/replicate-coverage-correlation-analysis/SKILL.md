---
name: replicate-coverage-correlation-analysis
description: Use when short-read and long-read BAM alignments of the same mRNA vaccine
  transcript are available and you need to quantify coverage homogeneity, per-nucleotide
  error profiles, and consensus accuracy across replicates or chemistries.
when_to_use_negative:
- You need to measure mRNA chemistry or nucleoside modifications (e.g. N1-methylpseudouridine),
  which requires direct RNA sequencing rather than alignment-based coverage comparison
- The alignment BAMs come from plasmid-template sequencing quality checks where the
  poly(A) tail region is not the target of the comparison
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Mana
  role: Command-line toolkit orchestrating coverage, error-profile, and consensus
    statistics from mRNA alignment BAMs
  repo: https://github.com/scchess/Mana
- name: pysamstats
  role: Computes the per-nucleotide error profile of mapped reads relative to the
    reference sequence
  repo: https://github.com/alimanfoo/pysamstats
- name: SAMtools
  role: Generates sorted and indexed BAM files from the alignments and extracts primary
    aligned reads (view -F 2048)
- name: BCFtools
  role: Variant calling and consensus sequence generation used for consensus accuracy
    comparison
- name: Minimap2
  role: Aligns ONT reads to the plasmid reference with -ax map-ont before coverage
    analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:1c8a4d630e7efbe5dec8fad85dbefd02a1fbaf0ebcd10b8e2cd9231be59bbc3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Illumina
  - Mana
  - Minimap2
  - Pysam
  - SAMtools
  - pysamstats
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Replicate coverage correlation analysis

## Summary

Compares per-nucleotide coverage and error profiles of Illumina TruSeq and ONT cDNA alignments over an mRNA vaccine transcript — especially the low-complexity 3' poly(A) tail — using Mana, and validates coverage homogeneity by checking that replicate Illumina coverage profiles correlate with R2 approximately 0.99.

## When to use

Use when short-read and long-read BAM alignments of the same mRNA vaccine transcript are available and you need to quantify coverage homogeneity, per-nucleotide error profiles, and consensus accuracy across replicates or chemistries.

## When NOT to use

- You need to measure mRNA chemistry or nucleoside modifications (e.g. N1-methylpseudouridine), which requires direct RNA sequencing rather than alignment-based coverage comparison
- The alignment BAMs come from plasmid-template sequencing quality checks where the poly(A) tail region is not the target of the comparison

## Inputs

- Sorted, indexed Illumina TruSeq alignment BAM (primary aligned reads)
- Sorted, indexed ONT cDNA alignment BAM (aligned with Minimap2 -ax map-ont, secondary alignments removed via samtools view -F 2048)
- Reference FASTA containing the template-encoded mRNA sequence including the 3' poly(A) tail

## Outputs

- Per-nucleotide coverage profiles over the full mRNA and the 3' poly(A) tail region for both chemistries
- Per-nucleotide error profile of mapped reads relative to the reference
- Consensus statistics and consensus sequence from BCFtools variant calling
- Replicate coverage correlation with reported R2 (approximately 0.99)

## How to apply

Mount the Illumina TruSeq alignment BAM, the ONT cDNA alignment BAM, and the reference FASTA into the Mana docker image with -v ${PWD}:/src and run 'python3 mana.py --mrna' with -b pointing to each BAM, -f to the FASTA, and -o to an output directory; Mana internally invokes SAMtools, BCFtools, Pysam, and bedtools. From the outputs, extract per-nucleotide coverage and per-nucleotide error profiles (determined relative to the reference via pysamstats) over the 3' poly(A) tail, where template-encoded low-complexity sequence causes Illumina misalignment. Compute coverage homogeneity statistics showing that ONT cDNA reads cover the entire mRNA length uniformly while Illumina coverage is heterogeneous and disrupted at the poly(A) tail. Finally, compute the correlation between Illumina coverage profiles of sequencing replicates and validate that the squared correlation reaches the reported R2 = 0.99.

## Related tools

- **Mana** (Command-line toolkit orchestrating coverage, error-profile, and consensus statistics from mRNA alignment BAMs) — https://github.com/scchess/Mana
- **pysamstats** (Computes the per-nucleotide error profile of mapped reads relative to the reference sequence) — https://github.com/alimanfoo/pysamstats
- **SAMtools** (Generates sorted and indexed BAM files from the alignments and extracts primary aligned reads (view -F 2048))
- **BCFtools** (Variant calling and consensus sequence generation used for consensus accuracy comparison)
- **Minimap2** (Aligns ONT reads to the plasmid reference with -ax map-ont before coverage analysis)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/illumina.bam -f /src/reference.fasta -o /src/out
```

## Evaluation signals

- Mana completes both --mrna runs (Illumina and ONT BAMs) without error, with analysis time of roughly 10 minutes per run
- ONT cDNA coverage is full and homogeneous across the entire mRNA vaccine length, while Illumina coverage is heterogeneous and disrupted at the poly(A) tail
- Per-nucleotide error profile shows elevated Illumina errors and consensus accuracy drop specifically over the low-complexity poly(A) tail
- Squared correlation of replicate Illumina coverage profiles reaches R2 approximately 0.99

## Limitations

- Short-read Illumina data misalign at the template-encoded poly(A) tail, so consensus accuracy there reflects the low-complexity limitation of short reads rather than true sequence error
- The study data are proof of concept, ranging from n = 1 to n = 4, with no statistical method used to predetermine sample size and experiments not randomised
- NanoPore analysis relied on a custom pipeline (Mana) whose correctness must be checked against the reference-encoded poly(A) sequence

## Evidence

- [discussion] Per-nucleotide error profile via pysamstats: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] Homogeneous ONT coverage of full mRNA length: "Nanopore cDNA sequencing was sufficient to confirm sequence identity with full and homogenous coverage of the entire mRNA vaccine length"
- [readme] Mana command-line usage from README: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] Mana docker invocation pattern from README: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam"

---
name: polya-tail-misalignment-quantification
description: Use when comparing short-read (Illumina TruSeq) vs long-read (ONT cDNA)
  alignment behaviour at a template-encoded 3' poly(A) tail, to quantify short-read
  misalignment errors and the resulting consensus accuracy drop.
when_to_use_negative:
- When the goal is overall mRNA quality attributes (sequence identity, integrity,
  poly(A) tail length, contamination) rather than a focused comparison of alignment
  accuracy at the poly(A) tail — use the broader VAX-seq/Mana QC reporting instead.
- When neither input BAM contains a template-encoded 3' poly(A) tail (e.g. enzymatically
  polyadenylated RNA or non-mRNA references), since the comparison relies on the reference-encoded
  poly(A) sequence as ground truth.
- When only one sequencing platform is available — the skill is inherently a short-read
  vs long-read comparative analysis.
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0080
tools:
- name: Mana
  role: Dockerised command-line toolkit that runs the --mrna analysis, internally
    invoking SAMtools, BCFtools, Pysam, and bedtools to generate per-nucleotide coverage,
    error-profile, and consensus statistics from the alignment BAMs
  repo: https://github.com/scchess/Mana
- name: pysamstats
  role: Computes per-nucleotide statistics (coverage, variation, matches/mismatches/insertions/deletions)
    against genome positions from the SAM/BAM alignments
  repo: https://github.com/alimanfoo/pysamstats
- name: Illumina
  role: Source of short-read TruSeq alignments whose misalignment at the poly(A) tail
    is quantified
- name: Oxford Nanopore Technologies (ONT)
  role: Source of long-read cDNA alignments serving as the uniform-coverage comparator
    at the poly(A) tail
- name: SAMtools
  role: Internal Mana dependency for BAM sorting/indexing and read handling
- name: BCFtools
  role: Internal Mana dependency for variant calling and consensus sequence generation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:c48e18d7c16c3c22026f298836f801c59438ce3570918856d2ce19242348ea75
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - BCFtools
  - Illumina
  - Mana
  - Oxford Nanopore Technologies (ONT)
  - Pysam
  - SAMtools
  - pysamstats
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# polya-tail-misalignment-quantification

## Summary

Quantifies how short-read Illumina TruSeq sequencing misaligns at the low-complexity 3' poly(A) tail of an eGFP mRNA relative to long-read ONT cDNA sequencing, by running the Mana --mrna pipeline on both alignment BAMs and comparing per-nucleotide coverage, error profiles, and consensus accuracy against the reference-encoded poly(A) sequence.

## When to use

Use when comparing short-read (Illumina TruSeq) vs long-read (ONT cDNA) alignment behaviour at a template-encoded 3' poly(A) tail, to quantify short-read misalignment errors and the resulting consensus accuracy drop.

## When NOT to use

- When the goal is overall mRNA quality attributes (sequence identity, integrity, poly(A) tail length, contamination) rather than a focused comparison of alignment accuracy at the poly(A) tail — use the broader VAX-seq/Mana QC reporting instead.
- When neither input BAM contains a template-encoded 3' poly(A) tail (e.g. enzymatically polyadenylated RNA or non-mRNA references), since the comparison relies on the reference-encoded poly(A) sequence as ground truth.
- When only one sequencing platform is available — the skill is inherently a short-read vs long-read comparative analysis.

## Inputs

- Illumina TruSeq alignment BAM (sorted/indexed, aligned to the plasmid reference)
- ONT cDNA alignment BAM (aligned with Minimap2 -ax map-ont)
- Reference FASTA containing the eGFP mRNA/plasmid sequence with template-encoded poly(A) tail

## Outputs

- Per-nucleotide coverage over the 3' poly(A) tail region for both Illumina and ONT alignments
- Per-nucleotide error profile relative to the reference over the poly(A) tail
- Consensus statistics and consensus accuracy metrics from Mana's --mrna report
- Coverage homogeneity statistics across the full mRNA length
- Replicate correlation statistic (R2) between Illumina coverage profiles

## How to apply

Place the Illumina TruSeq alignment BAM, the ONT cDNA alignment BAM, and the reference FASTA in one directory and mount it into the Mana docker image with -v ${PWD}:/src. Run 'python3 mana.py --mrna -b <BAM> -f <FASTA> -o <outdir>' once per alignment, letting Mana invoke SAMtools, BCFtools, Pysam, and bedtools internally to produce per-nucleotide coverage, error profiles (via pysamstats-style statistics), and consensus statistics. Extract the per-nucleotide coverage and error profile over the 3' poly(A) tail region for both alignments and quantify Illumina misalignment and the consensus accuracy drop relative to the reference-encoded poly(A) sequence. Compute coverage homogeneity statistics: ONT cDNA reads should cover the entire mRNA length uniformly, while Illumina coverage should be heterogeneous and disrupted at the poly(A) tail. Finally, correlate Illumina coverage profiles of sequencing replicates; the squared correlation should reach R2 = 0.99, confirming that the heterogeneous Illumina coverage is reproducible rather than random.

## Related tools

- **Mana** (Dockerised command-line toolkit that runs the --mrna analysis, internally invoking SAMtools, BCFtools, Pysam, and bedtools to generate per-nucleotide coverage, error-profile, and consensus statistics from the alignment BAMs) — https://github.com/scchess/Mana
- **pysamstats** (Computes per-nucleotide statistics (coverage, variation, matches/mismatches/insertions/deletions) against genome positions from the SAM/BAM alignments) — https://github.com/alimanfoo/pysamstats
- **Illumina** (Source of short-read TruSeq alignments whose misalignment at the poly(A) tail is quantified)
- **Oxford Nanopore Technologies (ONT)** (Source of long-read cDNA alignments serving as the uniform-coverage comparator at the poly(A) tail)
- **SAMtools** (Internal Mana dependency for BAM sorting/indexing and read handling)
- **BCFtools** (Internal Mana dependency for variant calling and consensus sequence generation)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/illumina.bam -f /src/reference.fasta -o /src/out
```

## Evaluation signals

- Both Mana --mrna runs (Illumina BAM and ONT cDNA BAM) complete without error, producing coverage, error-profile, and consensus outputs
- Per-nucleotide error profile over the 3' poly(A) tail shows elevated Illumina errors and poor consensus accuracy relative to the reference-encoded poly(A) sequence
- ONT cDNA coverage is homogeneous across the entire mRNA length, while Illumina coverage is heterogeneous and disrupted at the poly(A) tail
- Squared correlation of Illumina replicate coverage profiles reaches R2 ≈ 0.99, demonstrating reproducible heterogeneous coverage
- Consensus sequences generated by BCFtools inside Mana reproduce the reference outside the poly(A) tail, localising accuracy loss to the low-complexity region

## Limitations

- Short-read misalignment at the low-complexity poly(A) tail causes many errors and poor consensus accuracy, so Illumina consensus calls at the tail are unreliable by design
- Illumina coverage is heterogeneous across the mRNA and disrupted at the poly(A) tail, limiting downstream per-nucleotide conclusions from short reads
- Mana is designed for ONT alignments (.BAM); applying it to Illumina BAMs for this comparison is a deliberate repurposing, and the analysis is specific to template-encoded (reference-encoded) poly(A) tails

## Evidence

- [results] Illumina misalignment at the poly(A) tail and consensus accuracy drop: [evidence span withheld — anchor status: not_found]
- [discussion] Nanopore homogeneous coverage of mRNA length: "Nanopore cDNA sequencing was sufficient to confirm sequence identity with full and homogenous coverage of the entire mRNA vaccine length"
- [abstract] Mana QC reports from ONT BAMs: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] Mana invocation from README: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta \
               -o /src/out"
- [readme] pysamstats per-nucleotide statistics: "A Python utility for calculating statistics against genome positions
based on sequence alignments from a SAM or BAM file."
- [methods] Evaluation via Mana docker run with -b Illumina BAM, -f reference FASTA, -o output directory: [evidence span withheld — anchor status: not_found]

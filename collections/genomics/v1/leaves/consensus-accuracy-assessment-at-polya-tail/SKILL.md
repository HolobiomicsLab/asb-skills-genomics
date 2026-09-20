---
name: consensus-accuracy-assessment-at-polya-tail
description: Use when comparing short-read vs long-read alignment and consensus behaviour
  over a low-complexity region such as a template-encoded poly(A) tail in an mRNA
  vaccine construct, or when short-read coverage appears heterogeneous/disrupted at
  the 3' end of an mRNA reference.
when_to_use_negative:
- When the goal is total-readcount or read-length QC only (e.g. NanoPlot run statistics)
  rather than position-level consensus accuracy
- When the construct lacks a template-encoded poly(A) tail or other low-complexity
  region, since the skill's failure-mode detection is specific to the tail
- When only ONT cDNA data are available and no short-read comparison is required —
  Mana's --mrna report on the ONT BAM alone suffices without the contrast analysis
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3524
tools:
- name: Mana
  role: Command-line toolkit run via its docker image ('python3 mana.py --mrna -b
    BAM -f FASTA -o out') that orchestrates SAMtools, BCFtools, Pysam, and bedtools
    to produce coverage, error-profile, and consensus reports from an alignment BAM
  repo: https://github.com/scchess/Mana
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read cDNA sequencing modality whose alignment provides the homogeneous-coverage
    benchmark at the poly(A) tail
- name: Illumina
  role: Short-read TruSeq sequencing modality whose coverage and consensus accuracy
    at the poly(A) tail is quantified as the misalignment case
- name: BCFtools
  role: Variant calling and consensus sequence generation invoked by Mana to assess
    consensus accuracy against the reference
- name: Pysam
  role: Pysam/pysamstats computes the per-nucleotide error profile of mapped reads
    relative to the reference index sequence
  repo: https://github.com/alimanfoo/pysamstats
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:60651b36980caede09799401ade98fb1f180389f0d2f5f1d4dbeb42fa0621ebb
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# consensus-accuracy-assessment-at-polya-tail

## Summary

Quantifies how short-read Illumina TruSeq versus long-read ONT cDNA sequencing perform at the template-encoded 3' poly(A) tail of an eGFP mRNA, using the Mana toolkit to derive per-nucleotide coverage, error profiles, and consensus statistics. It demonstrates that low-complexity poly(A) sequence drives Illumina misalignment and poor consensus accuracy while ONT covers the mRNA homogeneously.

## When to use

Use when comparing short-read vs long-read alignment and consensus behaviour over a low-complexity region such as a template-encoded poly(A) tail in an mRNA vaccine construct, or when short-read coverage appears heterogeneous/disrupted at the 3' end of an mRNA reference.

## When NOT to use

- When the goal is total-readcount or read-length QC only (e.g. NanoPlot run statistics) rather than position-level consensus accuracy
- When the construct lacks a template-encoded poly(A) tail or other low-complexity region, since the skill's failure-mode detection is specific to the tail
- When only ONT cDNA data are available and no short-read comparison is required — Mana's --mrna report on the ONT BAM alone suffices without the contrast analysis

## Inputs

- Illumina TruSeq alignment BAM (reads quality-trimmed to Q20 with TrimGalore)
- ONT cDNA alignment BAM (nanopore reads filtered at Q>9, aligned with Minimap2 -ax map-ont)
- Reference FASTA containing the mRNA/plasmid sequence with the reference-encoded poly(A) tail
- Mana docker image (scchess/mana)

## Outputs

- Per-nucleotide coverage profiles over the mRNA including the poly(A) tail, for both Illumina and ONT alignments
- Per-nucleotide error profiles relative to the reference sequence
- Consensus statistics / consensus sequence comparison at the poly(A) tail
- Coverage homogeneity statistics across the full mRNA length
- Replicate correlation of Illumina coverage profiles (expected R2 ≈ 0.99)

## How to apply

Mount the Illumina TruSeq alignment BAM, the ONT cDNA alignment BAM, and the reference FASTA into the Mana docker image (docker pull scchess/mana, -v ${PWD}:/src). Run 'python3 mana.py --mrna -b <ilmn.bam> -f <ref.fasta> -o <out>' once per alignment; Mana internally invokes SAMtools, BCFtools, Pysam, and bedtools to generate per-nucleotide coverage, error profiles, and consensus statistics. Extract the per-nucleotide coverage and error profile over the 3' poly(A) tail region from both runs and quantify Illumina misalignment and the drop in consensus accuracy relative to the reference-encoded poly(A) sequence. Compute coverage homogeneity statistics across the full mRNA length, then check Illumina replicate concordance, expecting a squared coverage-profile correlation near R2 = 0.99. The rationale is that pysamstats-derived per-nucleotide error profiles relative to the reference reveal where consensus calling degrades, which for low-complexity tails is a known short-read failure mode.

## Related tools

- **Mana** (Command-line toolkit run via its docker image ('python3 mana.py --mrna -b BAM -f FASTA -o out') that orchestrates SAMtools, BCFtools, Pysam, and bedtools to produce coverage, error-profile, and consensus reports from an alignment BAM) — https://github.com/scchess/Mana
- **Oxford Nanopore Technologies (ONT)** (Long-read cDNA sequencing modality whose alignment provides the homogeneous-coverage benchmark at the poly(A) tail)
- **Illumina** (Short-read TruSeq sequencing modality whose coverage and consensus accuracy at the poly(A) tail is quantified as the misalignment case)
- **BCFtools** (Variant calling and consensus sequence generation invoked by Mana to assess consensus accuracy against the reference)
- **Pysam** (Pysam/pysamstats computes the per-nucleotide error profile of mapped reads relative to the reference index sequence) — https://github.com/alimanfoo/pysamstats

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Mana --mrna completes without error for both the Illumina BAM and the ONT cDNA BAM runs (each takes roughly 10 minutes)
- ONT cDNA coverage is homogeneous across the entire mRNA vaccine length, including the poly(A) tail
- Illumina coverage is heterogeneous and disrupted at the poly(A) tail with elevated per-nucleotide error rates relative to the reference-encoded poly(A) sequence
- Squared correlation between Illumina short-read coverage profiles of sequencing replicates reaches approximately R2 = 0.99
- BCFtools consensus sequence called from the ONT BAM matches the reference at the poly(A) tail, whereas the Illumina-derived consensus shows errors in that region

## Limitations

- Short-read misalignment at the poly(A) tail produces many errors and poor consensus accuracy, so Illumina data cannot be trusted for low-complexity tail assessment
- Data are proof of concept with small sample sizes (n = 1 to n = 4) and experiments were not randomised; template quality is the dominant determinant of sequencing quality
- Mana analysis time is about 10 minutes per run and the tool is specific to ONT/Illumina alignment BAM inputs against a reference FASTA

## Evidence

- [discussion] ONT coverage homogeneity vs Illumina heterogeneity: "Nanopore cDNA sequencing was sufficient to confirm sequence identity with full and homogenous coverage of the entire mRNA vaccine length"
- [discussion] per-nucleotide error profile method: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] consensus generation step: "Variant calling and consensus sequence generation was completed using bcftools (v1.15)"
- [readme] Mana command-line invocation: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta"
- [abstract] Mana purpose: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."

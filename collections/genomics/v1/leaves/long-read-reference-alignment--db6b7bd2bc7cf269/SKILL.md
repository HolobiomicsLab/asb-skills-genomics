---
name: long-read-reference-alignment
description: Use when you have long reads (FASTA/FASTQ, e.g. PacBio CLR or ONT) from
  raw or error-corrected pipelines and need to map them to reference genomes to measure
  residual error rates or validate correction performance.
when_to_use_negative:
- The reads are short Illumina paired-end reads — use a short-read aligner/preset
  (`-ax sr`) or a short-read mapper instead
- The goal is read-to-read overlap detection rather than alignment to a reference
  — use minimap2's `-x ava-pb`/`-x ava-ont` overlap mode or VeChat's internal overlap
  computation instead
- The reference is a variation graph rather than linear FASTA sequences — minimap2
  as used here aligns to linear reference genomes
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: minimap2
  role: Aligns raw and VeChat-corrected PacBio CLR reads back to the three E. coli
    strain reference genomes so residual error rates can be quantified
  repo: https://github.com/lh3/minimap2
- name: VeChat
  role: Produces the error-corrected read sets (reads.corrected.fa) whose residual
    error rate is measured by reference alignment
  repo: https://github.com/HaploKit/vechat
- name: PBSIM2
  role: Simulates the PacBio CLR read sets at 10x and 50x coverage per haplotype that
    serve as alignment input
- name: racon
  role: Consensus/polishing module referenced alongside minimap2 and yacrd in the
    paper's methods toolchain
- name: yacrd
  role: Chimeric-read detection tool listed in the paper's methods alongside minimap2
    and fpa
- name: fpa
  role: Pairwise-alignment filter listed in the paper's methods alongside minimap2
    and yacrd
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34381-8
    title: 'VeChat: correcting errors in long reads using variation graphs'
  source_content_hash: sha256:8a01156b4176e04fb0a44b35b08565625c4953132e779f663137f63393d4758e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - PBSIM2
  - VeChat
  - fpa
  - minimap2
  - racon
  - yacrd
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# long-read-reference-alignment

## Summary

Align raw and error-corrected long reads (e.g., PacBio CLR simulated from a triploid E. coli genome) back to their reference sequences using minimap2, in order to quantify residual sequencing error rates after correction with tools like VeChat.

## When to use

Use when you have long reads (FASTA/FASTQ, e.g. PacBio CLR or ONT) from raw or error-corrected pipelines and need to map them to reference genomes to measure residual error rates or validate correction performance.

## When NOT to use

- The reads are short Illumina paired-end reads — use a short-read aligner/preset (`-ax sr`) or a short-read mapper instead
- The goal is read-to-read overlap detection rather than alignment to a reference — use minimap2's `-x ava-pb`/`-x ava-ont` overlap mode or VeChat's internal overlap computation instead
- The reference is a variation graph rather than linear FASTA sequences — minimap2 as used here aligns to linear reference genomes

## Inputs

- Raw long reads in FASTQ/FASTQ.gz format (e.g. PBSIM2-simulated PacBio CLR reads at 10x/50x coverage per haplotype)
- Error-corrected reads in FASTA format (e.g. VeChat output reads.corrected.fa)
- Reference genome sequences in FASTA format (e.g. three E. coli strain reference genomes)

## Outputs

- Alignment files in SAM or PAF format mapping each read set to the references
- Per-read-set residual sequencing error rates tabulated across coverage levels

## How to apply

Run minimap2 with a long-read preset matched to the sequencing platform (e.g. `-ax map-pb` for PacBio CLR genomic reads, `-ax map-ont` for ONT) to align raw and corrected read sets back to the reference strains. In the VeChat coverage-scaling benchmark, both the 10x and 50x PBSIM2-simulated triploid E. coli read sets (raw and VeChat-corrected) are aligned to the three E. coli strain references with minimap2, and residual sequencing error rates are then quantified per read set. Minimap2 accepts gzip'd FASTA/FASTQ directly and produces PAF or SAM alignment output suitable for downstream error-rate tabulation. Compare per-set error rates across conditions — e.g., the expected drop from ~0.311% at 10x to ~0.017% at 50x coverage after VeChat correction.

## Related tools

- **minimap2** (Aligns raw and VeChat-corrected PacBio CLR reads back to the three E. coli strain reference genomes so residual error rates can be quantified) — https://github.com/lh3/minimap2
- **VeChat** (Produces the error-corrected read sets (reads.corrected.fa) whose residual error rate is measured by reference alignment) — https://github.com/HaploKit/vechat
- **PBSIM2** (Simulates the PacBio CLR read sets at 10x and 50x coverage per haplotype that serve as alignment input)
- **racon** (Consensus/polishing module referenced alongside minimap2 and yacrd in the paper's methods toolchain)
- **yacrd** (Chimeric-read detection tool listed in the paper's methods alongside minimap2 and fpa)
- **fpa** (Pairwise-alignment filter listed in the paper's methods alongside minimap2 and yacrd)

## Examples

```
minimap2 -ax map-pb ecoli_strains.fa reads.corrected.fa > corrected_vs_ref.sam
```

## Evaluation signals

- Error-rate summary shows the corrected-read error rate decreasing from ~0.311% (10x) to ~0.017% (50x) coverage, matching the paper's reported finding
- Minimap2 runs complete with a platform-appropriate preset (map-pb for PacBio CLR) and produce non-empty SAM/PAF alignments against each of the three reference strains
- Corrected read sets align with substantially lower mismatch/error profiles than the corresponding raw read sets
- Alignment output covers reads from all three haplotypes of the triploid E. coli mixture, confirming haplotype-aware correction did not mask low-frequency variants

## Limitations

- Indexing parameters such as -k, -w, -H and -I cannot be changed once a minimap2 index is built, so multiple indexes are needed for different data types
- Minimap2 is designed for read error rates up to ~15% and species divergence below ~15%; noisier inputs may require tuning
- minimap2.com is a phishing site — obtain the tool only via the official GitHub repository

## Evidence

- [other] Align raw and corrected reads for both coverage levels back to the three reference strains with minimap2 and quantify residual sequencing error rates per read set.: [evidence span withheld — anchor status: not_found]
- [readme] -ax map-pb ref.fa pacbio.fq.gz > aln.sam       # PacBio CLR genomic reads: [evidence span withheld — anchor status: external_doc]
- [readme] Minimap2 seamlessly works with gzip'd FASTA and FASTQ formats as input. You don't need to convert between FASTA and FASTQ or decompress gzip'd files first.: "Minimap2 seamlessly works with gzip'd FASTA and FASTQ formats as input. You
don't need to convert between FASTA and FASTQ or decompress gzip'd files first."
- [other] the error rate decreasing from approximately 0.311% at 10x coverage to approximately 0.017% at 50x coverage: [evidence span withheld — anchor status: not_found]
- [readme] finding overlaps between long reads with error rate up to ~15%: "(2)
finding overlaps between long reads with error rate up to ~15%;"

---
name: rna-sequence-error-rate-quantification
description: Use when aligned ONT direct RNA reads (BAM) against an mRNA reference
  are available and you need a total error rate plus mismatch/deletion/insertion decomposition,
  or to compare dRNA-specific per-nucleotide error biases against matched plasmid/cDNA
  sequencing.
when_to_use_negative:
- The goal is consensus sequence identity confirmation rather than error-rate estimation
  — the authors found cDNA sequencing more reliable than direct RNA sequencing for
  identity calls
- Only short-read Illumina data is available, since this skill is grounded in ONT
  alignment analysis via Mana and pysamstats
- You need an error rate including the poly(A) tail — this method explicitly excludes
  the poly(A) region, where dRNA shows deletion of low-quality nucleotides
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns SQK-RNA002 Nanopore reads to the reference with the '-ax map-ont' preset
- name: SAMtools v1.15
  role: Converts Minimap2 SAM output to sorted and indexed BAM files for downstream
    QC and error profiling
- name: Mana
  role: Command-line tool run in --mrna mode (docker scchess/mana) that analyses ONT
    alignments (.BAM) to report mRNA quality control statistics
  repo: https://github.com/scchess/Mana
- name: Pysam
  role: Dependency of Mana (0.20.0) providing pysamstats, used to compute the per-nucleotide
    error profile of mapped reads
  repo: https://github.com/alimanfoo/pysamstats
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:a1e12080113bd81fdb18a71383ac28650d298389ebda2c438907f6232de08fdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools
  - SAMtools v1.15
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# rna-sequence-error-rate-quantification

## Summary

Quantify the mean per-nucleotide sequencing error rate of Oxford Nanopore direct RNA reads (e.g. SQK-RNA002 on an eGFP mRNA vaccine) and decompose it into mismatch, deletion and insertion classes using Minimap2, SAMtools, Mana and pysamstats, while excluding the poly(A) tail.

## When to use

Use when aligned ONT direct RNA reads (BAM) against an mRNA reference are available and you need a total error rate plus mismatch/deletion/insertion decomposition, or to compare dRNA-specific per-nucleotide error biases against matched plasmid/cDNA sequencing.

## When NOT to use

- The goal is consensus sequence identity confirmation rather than error-rate estimation — the authors found cDNA sequencing more reliable than direct RNA sequencing for identity calls
- Only short-read Illumina data is available, since this skill is grounded in ONT alignment analysis via Mana and pysamstats
- You need an error rate including the poly(A) tail — this method explicitly excludes the poly(A) region, where dRNA shows deletion of low-quality nucleotides

## Inputs

- Nanopore direct RNA FASTQ reads (SQK-RNA002) with barcodes removed (e.g. via Porechop v0.2.4)
- Reference mRNA/plasmid sequence in FASTA format
- Sorted and indexed BAM alignment (SAMtools v1.15) of reads mapped with Minimap2 '-ax map-ont'

## Outputs

- Mean sequencing error rate over the mRNA body excluding the poly(A) tail (e.g. 7.78% for the eGFP mRNA vaccine)
- Error-type decomposition into mismatch (2.61%), deletion (5.18%) and insertion (1.43%)
- Per-nucleotide error profile of mapped reads relative to the reference index sequence (pysamstats variation-style statistics)
- Comparative dRNA vs plasmid/cDNA per-nucleotide error bias table

## How to apply

Align the SQK-RNA002 direct RNA reads to the reference sequence with Minimap2 (Release 2.20-r1064) using the '-ax map-ont' preset for Nanopore reads, then process the SAM file with SAMtools v1.15 to produce a sorted and indexed BAM. Run Mana via its pre-built docker image (scchess/mana) in --mrna mode, mounting the working directory and passing -b /src/alignment.bam, -f /src/file.fasta and -o /src/out (expected runtime about 10 minutes). Compute the per-nucleotide error profile of mapped reads relative to the reference index sequence with pysamstats (Pysam 0.20.0 dependency of Mana), restricting the calculation to primary aligned reads and excluding the poly(A) tail region. Decompose errors into mismatch, deletion and insertion classes and total the mean error rate across the mRNA body. Finally, compare the dRNA per-nucleotide biases against matched plasmid and cDNA error profiles to isolate dRNA-specific error modes such as deletion of low-quality poly(A) tail nucleotides.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns SQK-RNA002 Nanopore reads to the reference with the '-ax map-ont' preset)
- **SAMtools v1.15** (Converts Minimap2 SAM output to sorted and indexed BAM files for downstream QC and error profiling)
- **Mana** (Command-line tool run in --mrna mode (docker scchess/mana) that analyses ONT alignments (.BAM) to report mRNA quality control statistics) — https://github.com/scchess/Mana
- **Pysam** (Dependency of Mana (0.20.0) providing pysamstats, used to compute the per-nucleotide error profile of mapped reads) — https://github.com/alimanfoo/pysamstats

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Reproduced total error rate matches the reported 7.78% for the eGFP mRNA vaccine with poly(A) tail excluded
- Error decomposition matches the reported 2.61% mismatch, 5.18% deletion and 1.43% insertion
- Per-nucleotide statistics are restricted to primary aligned reads (e.g. secondary alignments flagged with SAMtools view -F 2048 are excluded)
- The BAM input is sorted and indexed by SAMtools v1.15 before Mana/pysamstats run
- The Mana --mrna docker run completes in roughly the expected 10 minutes and writes output to the specified -o directory

## Limitations

- Direct RNA sequencing shows dRNA-specific error biases, including deletion of low-quality poly(A) tail nucleotides, so error statistics are only interpretable when the poly(A) region is excluded
- The authors found cDNA sequencing more reliable than direct RNA sequencing, so dRNA error rates should not be treated as baseline-accurate identity calls
- pysamstats pileup-based statistics default to a maximum read depth of 8000, which can truncate counts on deeply covered positions

## Evidence

- [discussion] Minimap2 alignment step: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] SAMtools BAM generation: "The resulting SAM alignment files were processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] pysamstats per-nucleotide error profile: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [readme] Mana purpose from README: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."

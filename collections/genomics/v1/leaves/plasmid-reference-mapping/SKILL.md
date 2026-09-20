---
name: plasmid-reference-mapping
description: Use when ONT FASTQ reads from a linearised pDNA template (e.g. SQK-LSK109)
  need read-fate fractions against a plasmid reference, including E. coli carry-over
  and failed-read fractions.
when_to_use_negative:
- Input reads are Illumina short reads (the article uses TrimGalore Q20 trimming and
  a different short-read workflow, not Minimap2 -ax map-ont)
- The sample is final mRNA product rather than a plasmid DNA template — use the mRNA/cDNA
  sequencing arm of VAX-seq instead of pDNA reference mapping
- Reads have not passed the quality filter (Q>9) so read-fate fractions would be skewed
  by low-quality reads
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns ONT reads to the plasmid reference with -ax map-ont, and aligns extracted
    unmapped reads to the E. coli reference
- name: SAMtools v1.15
  role: Processes SAM alignments into sorted and indexed BAM files
- name: Mana
  role: Analyses the ONT alignment BAMs and reports quality control statistics including
    read-fate fractions
  repo: https://github.com/scchess/Mana
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform producing the FASTQ reads of the linearised
    pDNA template (SQK-LSK109)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:a3c82fa5cf3186acc27e01b8be7b7808dcd44a3b27b86651b6545bf69b8f016f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2 (Release 2.20-r1064)
  - Oxford Nanopore Technologies (ONT)
  - SAMtools v1.15
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# plasmid-reference-mapping

## Summary

Quantifies the fate of ONT long reads from a linearised plasmid DNA (pDNA) template by aligning Nanopore reads to the plasmid reference with Minimap2, sorting/indexing BAMs with SAMtools, and computing read-fate fractions (plasmid-aligned, E. coli contamination, failed reads) via Mana.

## When to use

Use when ONT FASTQ reads from a linearised pDNA template (e.g. SQK-LSK109) need read-fate fractions against a plasmid reference, including E. coli carry-over and failed-read fractions.

## When NOT to use

- Input reads are Illumina short reads (the article uses TrimGalore Q20 trimming and a different short-read workflow, not Minimap2 -ax map-ont)
- The sample is final mRNA product rather than a plasmid DNA template — use the mRNA/cDNA sequencing arm of VAX-seq instead of pDNA reference mapping
- Reads have not passed the quality filter (Q>9) so read-fate fractions would be skewed by low-quality reads

## Inputs

- Concatenated ONT FASTQ reads from the linearised pDNA run (reads with quality scores >9)
- Plasmid reference sequence (FASTA)
- E. coli reference sequence (FASTA)

## Outputs

- Sorted and indexed BAM files of plasmid-aligned reads
- BAM/FASTA of unmapped reads aligned to the E. coli reference
- Read-fate fractions from Mana: fraction aligned to plasmid reference, fraction matching E. coli, fraction of failed reads

## How to apply

Concatenate nanopore reads (quality >9) into a single FASTQ, then align to the plasmid reference with Minimap2 (Release 2.20-r1064) using the -ax map-ont preset for Nanopore reads. Process the SAM output with SAMtools v1.15 to generate sorted, indexed BAM files. Extract the unmapped reads and align this FASTA/FASTQ to an E. coli reference with Minimap2 to quantify bacterial contamination. Finally run Mana (docker image scchess/mana) on the alignments to compute the read-fate fractions, which for the eGFP pDNA run should reproduce 86.8% plasmid-aligned, 6.8% E. coli, and 6.6% failed reads (summing to ~100%).

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns ONT reads to the plasmid reference with -ax map-ont, and aligns extracted unmapped reads to the E. coli reference)
- **SAMtools v1.15** (Processes SAM alignments into sorted and indexed BAM files)
- **Mana** (Analyses the ONT alignment BAMs and reports quality control statistics including read-fate fractions) — https://github.com/scchess/Mana
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform producing the FASTQ reads of the linearised pDNA template (SQK-LSK109))

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Reported fractions sum to approximately 100% (plasmid-aligned + E. coli + failed reads)
- Fractions reproduce the published values: 86.8% plasmid-aligned, 6.8% E. coli, 6.6% failed reads for the eGFP pDNA run
- BAM files are sorted and indexed successfully by SAMtools v1.15
- Mana completes its ~10-minute BAM analysis without errors and emits QC statistics

## Limitations

- Unmapped read contamination analysis depends on the completeness of the chosen E. coli reference; contamination from other sources would be missed
- Only reads with quality scores >9 are used, so fractions exclude low-quality reads
- Mana analysis takes about 10 minutes per BAM and requires SAMtools/BCFtools/pysam/bedtools dependencies, practically handled via the pre-built docker image

## Evidence

- [discussion] Alignment preset and tool version: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] BAM processing: "The resulting SAM alignment files were processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] Unmapped read contamination analysis: "This FASTA file was aligned to an E.coli reference sequence utilising Minimap2"
- [discussion] Read quality filter: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"
- [readme] Mana QC reporting role: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."

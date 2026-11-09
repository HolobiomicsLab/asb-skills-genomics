---
name: ont-read-fate-fraction-reporting
description: Use when ONT long reads (e.g. SQK-LSK109 of a linearised pDNA template,
  concatenated FASTQ of Q>9 reads) must be partitioned into plasmid-mapped, host-contamination
  and failed-read fractions as a QC metric.
when_to_use_negative:
- Input reads are short Illumina reads quality-trimmed to Q20 with TrimGalore — this
  read-fate accounting is defined for ONT map-ont alignments
- The goal is mRNA product QC (sequence identity, poly(A) tail length, integrity)
  rather than plasmid template identity and contamination quantification
- Only per-nucleotide error profiles or variant calling are needed — use pysamstats
  or bcftools v1.15 instead of the read-fate fractioning
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: Minimap2 (Release 2.20-r1064)
  role: Aligns the ONT reads to the plasmid reference with the -ax map-ont preset,
    and aligns extracted unmapped reads to the E. coli reference
- name: SAMtools v1.15
  role: Processes the SAM alignment files to generate sorted and indexed BAM files
    used for fate counting
- name: Mana
  role: Command-line tool that analyses the ONT alignments (.BAM) to compute and report
    the read-fate QC fractions (plasmid-mapped, E. coli, failed)
  repo: https://github.com/scchess/Mana
- name: Oxford Nanopore Technologies (ONT)
  role: Long-read sequencing platform producing the FASTQ reads of the linearised
    pDNA template analysed by this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:5e7c28f078cc5540732ebe622e6198a88b61cc18665b18619e82ba0668a5d3f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Minimap2 (Release 2.20-r1064)
  - Oxford Nanopore Technologies (ONT)
  - SAMtools v1.15
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ont-read-fate-fraction-reporting

## Summary

Quantifies the fate fractions of Oxford Nanopore reads from a linearised plasmid DNA template run: the percentage mapping to the plasmid reference, the percentage attributable to E. coli carry-over contamination, and the percentage of failed reads, using Minimap2, SAMtools and Mana.

## When to use

Use when ONT long reads (e.g. SQK-LSK109 of a linearised pDNA template, concatenated FASTQ of Q>9 reads) must be partitioned into plasmid-mapped, host-contamination and failed-read fractions as a QC metric.

## When NOT to use

- Input reads are short Illumina reads quality-trimmed to Q20 with TrimGalore — this read-fate accounting is defined for ONT map-ont alignments
- The goal is mRNA product QC (sequence identity, poly(A) tail length, integrity) rather than plasmid template identity and contamination quantification
- Only per-nucleotide error profiles or variant calling are needed — use pysamstats or bcftools v1.15 instead of the read-fate fractioning

## Inputs

- Concatenated ONT FASTQ of nanopore reads from linearised eGFP pDNA (SQK-LSK109), filtered to quality scores >9
- Plasmid reference sequence (FASTA)
- E. coli reference sequence (FASTA)

## Outputs

- Sorted and indexed BAM file of reads aligned to the plasmid reference
- BAM/alignment of unmapped reads aligned to the E. coli reference
- Mana read-fate report giving fraction aligned to plasmid (expected 86.8%), fraction matching E. coli (expected 6.8%), and fraction of failed reads (expected 6.6%)

## How to apply

Align the concatenated ONT FASTQ to the plasmid reference with Minimap2 (Release 2.20-r1064) using the -ax map-ont preset, then process the SAM via SAMtools v1.15 to produce sorted, indexed BAM files. Extract the unmapped reads and align this FASTA/FASTQ file to an E. coli reference sequence with Minimap2 to quantify bacterial contamination. Run Mana on the ONT BAM alignments to compute the read-fate fractions — fraction aligned to the plasmid reference, fraction matching E. coli, and fraction of failed reads. Validate by checking that the fractions reproduce the reported values (86.8% plasmid, 6.8% E. coli, 6.6% failed) and sum to approximately 100%. The rationale is that plasmid identity and carry-over contamination of the template are captured in a single read-classification accounting before in vitro transcription.

## Related tools

- **Minimap2 (Release 2.20-r1064)** (Aligns the ONT reads to the plasmid reference with the -ax map-ont preset, and aligns extracted unmapped reads to the E. coli reference)
- **SAMtools v1.15** (Processes the SAM alignment files to generate sorted and indexed BAM files used for fate counting)
- **Mana** (Command-line tool that analyses the ONT alignments (.BAM) to compute and report the read-fate QC fractions (plasmid-mapped, E. coli, failed)) — https://github.com/scchess/Mana
- **Oxford Nanopore Technologies (ONT)** (Long-read sequencing platform producing the FASTQ reads of the linearised pDNA template analysed by this skill)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- The three fractions (plasmid-aligned, E. coli, failed) sum to approximately 100%
- Reproduced plasmid-aligned fraction ≈ 86.8%, E. coli contamination ≈ 6.8%, and failed reads ≈ 6.6% for the linearised eGFP pDNA run
- The plasmid BAM is sorted and indexable (indexed via SAMtools v1.15 without error)
- Mana run completes on the ONT .BAM input and emits the QC statistics report (runtime about 10 minutes per the README)

## Limitations

- Data are proof of concept, ranging from n = 1 to n = 4, so fractions may not generalise across templates or runs
- Only nanopore reads with quality scores >9 were used to form the concatenated FASTQ, so the fractions exclude lower-quality reads
- Secondary alignments should be handled carefully — SAMtools view -F 2048 was used to extract primary aligned reads in the broader workflow
- The mapped reads were analysed with a custom pipeline (Mana); the analysis depends on correct choice of plasmid and E. coli reference sequences

## Evidence

- [discussion] ONT plasmid QC read-fate fractions: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] SAM processing: "processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] E. coli contamination quantification: "This FASTA file was aligned to an E.coli reference sequence utilising Minimap2"
- [intro] Mana role: "To support VAX-seq, we also develop a software toolkit, Mana, that provides detailed and automated reports of mRNA quality."
- [readme] Mana purpose per README: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [discussion] Read quality filter: "All nanopore reads with quality scores >9 were used to form a concatenated FASTQ file"

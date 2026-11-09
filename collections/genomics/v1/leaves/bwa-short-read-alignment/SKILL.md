---
name: bwa-short-read-alignment
description: Use when Illumina FASTQ reads must be aligned to a reference genome to
  compute read-level deletion fractions at specific homopolymer loci, to test whether
  observed length variation is biological rather than a sequencing-error artifact.
when_to_use_negative:
- The analysis goal is single-molecule methylation status of SMRT reads — use SMALR
  (SMsn/SMp protocols on aligned_reads.cmph5 input) instead; short-read pileups lack
  the circular-consensus repeated observations SMALR depends on.
- Only population-level consensus sequence identity is needed (e.g., variant calling
  or genome assembly) rather than per-read deletion fractions at defined homopolymer
  loci.
- The input reads are long SMRT subreads in .cmph5 format; 'bwa mem' here targets
  short Illumina MiSeq reads against FASTA references.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: bwa
  role: Indexes the reference genome FASTA ('bwa index') and aligns Illumina MiSeq
    FASTQ reads to the matching reference with 'bwa mem'
- name: samtools
  role: Sorts and indexes the BAM alignments produced by 'bwa mem', then generates
    pileups at homopolymer loci from which read-level deletion fractions are counted
- name: SMALR
  role: Complementary framework in the same study for single-molecule interrogation
    of SMRT read methylation status; the bwa/samtools short-read pipeline serves the
    read-level homopolymer deletion analysis
  repo: https://github.com/fanglab/smalr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:d6a1538d96ebe56b1f12e0fb6c11dec0091b42e8ea54f2e97f2bf3d97f532b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - SMALR
  - bwa
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bwa-short-read-alignment

## Summary

A short-read alignment workflow that uses 'bwa mem' to align Illumina MiSeq FASTQ reads to indexed reference genomes (e.g., GenBank CP011330/CP011331), then samtools sort/index and pileups to quantify per-locus read-level deletion rates in homopolymer tracts — distinguishing true biological length variation from homopolymer sequencing error.

## When to use

Use when Illumina FASTQ reads must be aligned to a reference genome to compute read-level deletion fractions at specific homopolymer loci, to test whether observed length variation is biological rather than a sequencing-error artifact.

## When NOT to use

- The analysis goal is single-molecule methylation status of SMRT reads — use SMALR (SMsn/SMp protocols on aligned_reads.cmph5 input) instead; short-read pileups lack the circular-consensus repeated observations SMALR depends on.
- Only population-level consensus sequence identity is needed (e.g., variant calling or genome assembly) rather than per-read deletion fractions at defined homopolymer loci.
- The input reads are long SMRT subreads in .cmph5 format; 'bwa mem' here targets short Illumina MiSeq reads against FASTA references.

## Inputs

- Reference genome FASTA file(s) (e.g., H. pylori J99 GenBank CP011330, CP011331; E. coli K12 control reference)
- Illumina MiSeq FASTQ reads (paired or single-end) for H. pylori J99, the E. coli K12 control, and single-colony isolates HPXZ1383-1387
- A list of genomic coordinates for homopolymer loci of interest and unrelated C/G homopolymer control loci

## Outputs

- Sorted, indexed BAM alignment files per sample/reference pair
- Per-locus, per-isolate read-level deletion rates (e.g., compiled as homopolymer_deletion_rates.csv)
- samtools pileups at each homopolymer locus showing per-read deletion calls

## How to apply

First index each reference genome FASTA (e.g., H. pylori J99 GenBank CP011330 and CP011331, plus the E. coli K12 control reference) with 'bwa index'. Align the matching Illumina MiSeq FASTQ reads with 'bwa mem', pipe through 'samtools sort', and index the resulting BAM files with 'samtools index' so that random-access pileups are possible. Generate samtools pileups at the homopolymer loci of interest (e.g., the M.Hpy99XXI and S.Hpy99XXII homopolymers) and at unrelated C/G homopolymer control loci. From each pileup, count the fraction of reads carrying a deletion within the homopolymer tract to obtain a per-locus read-level deletion rate; the rationale is that true length variation yields elevated deletion fractions across many reads, whereas homopolymer sequencing error produces uniformly low background rates. Repeat per isolate (e.g., the five single-colony isolates HPXZ1383-1387) and compile all per-locus, per-isolate rates into a table such as homopolymer_deletion_rates.csv for comparison.

## Related tools

- **bwa** (Indexes the reference genome FASTA ('bwa index') and aligns Illumina MiSeq FASTQ reads to the matching reference with 'bwa mem')
- **samtools** (Sorts and indexes the BAM alignments produced by 'bwa mem', then generates pileups at homopolymer loci from which read-level deletion fractions are counted)
- **SMALR** (Complementary framework in the same study for single-molecule interrogation of SMRT read methylation status; the bwa/samtools short-read pipeline serves the read-level homopolymer deletion analysis) — https://github.com/fanglab/smalr

## Examples

```
bwa index CP011330.fa && bwa mem CP011330.fa J99_R1.fastq.gz J99_R2.fastq.gz | samtools sort -o J99.sorted.bam && samtools index J99.sorted.bam && samtools mpileup -f CP011330.fa J99.sorted.bam
```

## Evaluation signals

- BAM files are coordinate-sorted and successfully indexed with samtools (index file present, samtools view/quickcheck passes)
- Read-level deletion fraction at the M.Hpy99XXI and S.Hpy99XXII homopolymers exceeds 15%
- Unrelated C/G homopolymer control loci remain below 3% deletion fraction
- The >15% vs <3% contrast is reproduced across all five single-colony isolates HPXZ1383-1387
- All per-locus, per-isolate deletion rates are compiled into homopolymer_deletion_rates.csv with entries for every analyzed locus and isolate

## Limitations

- Short-read pileups provide population-level, consensus-like observations and lack the single-molecule resolution needed to survey complex MTase activity, unlike SMALR's SMsn/SMp protocols.
- Homopolymeric regions are intrinsically error-prone in short reads; low-level deletions in controls (<3%) represent the sequencing-error background that must be measured to interpret target loci.
- Reliable methylation detection from SMRT reads depends on the number of repeated observations per single molecule — a tradeoff between library size and per-molecule accuracy — which this Illumina-based pipeline does not address.

## Evidence

- [other] Align the Illumina MiSeq FASTQ reads for H. pylori J99 and the E. coli K12 control to their matching references with 'bwa mem', then sort and index the resulting BAM files with samtools.: [evidence span withheld — anchor status: not_found]
- [other] accept the output when the read-level deletion fraction at the M.Hpy99XXI and S.Hpy99XXII homopolymers is >15% while unrelated C/G homopolymer controls remain <3%: [evidence span withheld — anchor status: not_found]
- [readme] SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads.: "SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads."
- [abstract] existing SMRT sequencing-based methods for studying bacterial methylomes rely on a population-level consensus that lacks the single-cell resolution required to observe epigenetic heterogeneity: "existing SMRT sequencing-based methods for studying bacterial methylomes rely on a population-level consensus that lacks the single-cell resolution required to observe epigenetic heterogeneity"

---
name: homopolymer-indel-quantification
description: Use when read-level length variation is observed in homopolymeric repeats
  and you must decide whether it is biological (e.
when_to_use_negative:
- The goal is estimating methylated vs non-methylated motif site fractions from SMRT
  kinetics — use the SMALR SMsn/SMp protocols instead of indel counting
- Only population-level consensus methylation status is needed, not single-molecule/read-level
  resolution
- No unrelated homopolymer control loci are available, since the >15% vs <3% contrast
  cannot be validated without error-matched C/G controls
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bwa
  role: Indexes the reference FASTA (bwa index) and aligns Illumina MiSeq FASTQ reads
    to matching references (bwa mem) before pileup generation
- name: samtools
  role: Sorts and indexes the BAM alignments and generates pileups at homopolymer
    and control loci from which per-read deletions are counted
- name: SMALR
  role: Framework from the same study providing the single-molecule-level rationale
    for assessing read-level (not consensus) variation at homopolymeric motif sites
  repo: https://github.com/fanglab/smalr
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms8438
    title: Single molecule-level detection and long read-based phasing of epigenetic
      variations in bacterial methylomes
  source_content_hash: sha256:3c2f7f7c28781774737044edca5b7320bb5a98910e23fb617ba9133ff0e0a9ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fanglab/smalr
  tools:
  - SMALR
  - bwa
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# homopolymer-indel-quantification

## Summary

Quantifies read-level deletion rates within homopolymeric tracts (e.g., the M.Hpy99XXI and S.Hpy99XXII homopolymers of H. pylori J99) from samtools pileups to distinguish true biological length variation from sequencing-error artifacts, using unrelated C/G homopolymer loci as error controls.

## When to use

Use when read-level length variation is observed in homopolymeric repeats and you must decide whether it is biological (e.g., MTase-associated phase variation) versus an artifact of homopolymer sequencing error, by comparing per-read deletion fractions at target tracts against control C/G homopolymers.

## When NOT to use

- The goal is estimating methylated vs non-methylated motif site fractions from SMRT kinetics — use the SMALR SMsn/SMp protocols instead of indel counting
- Only population-level consensus methylation status is needed, not single-molecule/read-level resolution
- No unrelated homopolymer control loci are available, since the >15% vs <3% contrast cannot be validated without error-matched C/G controls

## Inputs

- Reference genome FASTA files (GenBank CP011330 for H. pylori J99; CP011331 for E. coli K12 control)
- Illumina MiSeq FASTQ read files for H. pylori J99 and E. coli K12
- Genomic coordinates of M.Hpy99XXI and S.Hpy99XXII homopolymer loci
- Genomic coordinates of unrelated C/G homopolymer control loci
- Reads for five single-colony isolates HPXZ1383-1387

## Outputs

- Sorted and indexed BAM alignment files
- samtools pileups at homopolymer and control loci
- Per-locus, per-isolate read-level homopolymer deletion rates compiled in homopolymer_deletion_rates.csv

## How to apply

Index the reference FASTA files (GenBank CP011330 for H. pylori J99, CP011331 for the E. coli K12 control) with 'bwa index', align the Illumina MiSeq FASTQ reads to their matching references with 'bwa mem', and sort/index the BAMs with samtools (samtools >= 0.1.18 per the article's dependency list). Generate samtools pileups at the M.Hpy99XXI and S.Hpy99XXII homopolymer loci and at a set of unrelated C/G homopolymer control loci. From each pileup, count the fraction of reads carrying deletions within each homopolymer tract to obtain per-locus read-level deletion rates. Accept the analysis when target homopolymer deletion fractions exceed 15% while C/G controls remain below 3%; the SMALR framework from the same study motivates this single-molecule/read-level logic rather than population-level consensus. Repeat the alignment and counting for the five single-colony isolates HPXZ1383-1387 and compile all per-locus, per-isolate deletion rates into homopolymer_deletion_rates.csv.

## Related tools

- **bwa** (Indexes the reference FASTA (bwa index) and aligns Illumina MiSeq FASTQ reads to matching references (bwa mem) before pileup generation)
- **samtools** (Sorts and indexes the BAM alignments and generates pileups at homopolymer and control loci from which per-read deletions are counted)
- **SMALR** (Framework from the same study providing the single-molecule-level rationale for assessing read-level (not consensus) variation at homopolymeric motif sites) — https://github.com/fanglab/smalr

## Examples

```
bwa mem CP011330.fa J99_reads.fastq | samtools sort -o J99.sorted.bam; samtools index J99.sorted.bam; samtools pileup -f CP011330.fa J99.sorted.bam > homopolymer.pileup.txt
```

## Evaluation signals

- Read-level deletion fraction at the M.Hpy99XXI and S.Hpy99XXII homopolymers is >15%
- Unrelated C/G homopolymer control loci remain <3% deletion fraction
- The >15% vs <3% contrast is reproduced across all five isolates HPXZ1383-1387
- homopolymer_deletion_rates.csv contains per-locus, per-isolate deletion rates for both target and control loci
- BAM files are sorted and indexed and pileups exist for every analyzed locus

## Limitations

- Detection at single-molecule level fundamentally depends on the number of repeated observations per molecule; for a given read length there is a tradeoff between library size and per-molecule accuracy
- Homopolymeric regions are intrinsically error-prone for sequencing, so control loci are essential to interpret deletion rates as biological rather than artifactual
- Read-level indel counts are not themselves methylation calls; the method only establishes that tract length variation is genuine

## Evidence

- [intro] SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads.: "SMALR: a framework for single-molecule level interrogation of the methylation status of SMRT reads."
- [results] Read-level deletions markedly higher than controls: [evidence span withheld — anchor status: not_found]
- [methods] Pileups at target and control loci: [evidence span withheld — anchor status: not_found]
- [methods] samtools dependency version: "* samtools >= 0.1.18"
- [readme] SMsn design rationale: "This is designed for use with short (~250bp) sequencing library preps, where the long read lengths of SMRT reads enables multiple passes over each motif site."

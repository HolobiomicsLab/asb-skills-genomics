---
name: mrna-read-fraction-quantification
description: Use when you have a sorted, indexed BAM of reads aligned to an mRNA reference
  and need to classify and report the fraction of reads on the target mRNA versus
  off-target or contaminant RNA classes.
when_to_use_negative:
- The goal is sequence identity or variant calling of the mRNA itself rather than
  on-target/off-target read fraction quantification
- The input is raw FASTQ rather than an aligned, sorted, indexed BAM — alignment (Minimap2
  -ax map-ont) must be done first
- The sample is direct RNA sequencing chemistry being assessed for reliability rather
  than cDNA-based quality measurement, since the authors found cDNA sequencing more
  reliable than direct RNA sequencing
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Mana
  role: Command-line toolkit that analyses ONT alignments (.BAM) via its --mrna mode
    to report mRNA quality statistics including read fraction/contamination classes
  repo: https://github.com/scchess/Mana
- name: SAMtools
  role: Extracts primary aligned reads (SAMtools view -F 2048) from the sorted BAM
    before fraction classification
- name: bedtools
  role: Strand-aware intersect (-s) against reference mRNA feature boundaries to assign
    reads overlapping the eGFP mRNA interval as on-target
- name: Pysam
  role: Python library used to load the BAM and reference FASTA for read classification
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:5b9d91c10bf59c9f3e4954203550410ab78e78c3a56217a92309c358930223e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Pysam
  - SAMtools
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# mrna-read-fraction-quantification

## Summary

Quantifies the fraction of sequencing reads in VAX-seq mRNA libraries that map on-target versus off-target (upstream, downstream, antisense, cryptic TSS-derived, and contaminant RNAs), giving an integrated purity metric for a manufactured mRNA product.

## When to use

Use when you have a sorted, indexed BAM of reads aligned to an mRNA reference and need to classify and report the fraction of reads on the target mRNA versus off-target or contaminant RNA classes.

## When NOT to use

- The goal is sequence identity or variant calling of the mRNA itself rather than on-target/off-target read fraction quantification
- The input is raw FASTQ rather than an aligned, sorted, indexed BAM — alignment (Minimap2 -ax map-ont) must be done first
- The sample is direct RNA sequencing chemistry being assessed for reliability rather than cDNA-based quality measurement, since the authors found cDNA sequencing more reliable than direct RNA sequencing

## Inputs

- Sorted, indexed alignment BAM file of mRNA (VAX-seq) reads aligned to the reference
- Reference FASTA file containing the mRNA (plasmid/target) sequence
- Feature boundaries of the target mRNA interval (e.g. the 1153 nt eGFP transcript)
- Optional contaminant reference sequences (e.g. E. coli) for classification of unmapped reads

## Outputs

- Classification table of read fractions per class (on-target, upstream, downstream, antisense, cryptic TSS, E. coli contaminant)
- Mana mRNA quality report with fractional abundance of each read class relative to total reads

## How to apply

Load the sorted, indexed BAM and reference FASTA with pysam/SAMtools from the mounted analysis directory, then extract primary aligned reads with SAMtools view -F 2048 to exclude secondary alignments before classification. Use bedtools intersect with strand-aware (-s) coordinate filtering against the reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target. Classify remaining reads by position and orientation relative to the target (upstream, downstream, antisense, cryptic transcription start site-derived) plus reads mapping to contaminant sequences such as E. coli, and compute each class's fractional abundance relative to total reads. For nanopore cDNA libraries of the eGFP mRNA, validate against published fractions: 92.7% on-target, 7.3% off-target, 0.3% cryptic TSS, 0.01% E. coli; for the Illumina short-read library, 1.4% upstream, 5.7% downstream, and ~0.6% antisense. The Mana toolkit wraps these steps and reports the fractions in an automated quality report.

## Related tools

- **Mana** (Command-line toolkit that analyses ONT alignments (.BAM) via its --mrna mode to report mRNA quality statistics including read fraction/contamination classes) — https://github.com/scchess/Mana
- **SAMtools** (Extracts primary aligned reads (SAMtools view -F 2048) from the sorted BAM before fraction classification)
- **bedtools** (Strand-aware intersect (-s) against reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target)
- **Pysam** (Python library used to load the BAM and reference FASTA for read classification)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Nanopore cDNA library fractions match published values: 92.7% on-target, 7.3% off-target, 0.3% cryptic TSS, 0.01% E. coli
- Illumina short-read library fractions match published values: 1.4% upstream, 5.7% downstream, ~0.6% antisense
- Class fractions sum to 1.0 (or 100%) relative to total primary reads after excluding secondary alignments with -F 2048
- Mana run completes within the expected ~10 minutes analysis time and emits a report to the output directory
- Fractional classifications are strand-aware: antisense reads are only assigned using orientation-aware filtering (bedtools -s)

## Limitations

- Reported fractions are strand- and interval-definition dependent; mis-specified reference mRNA feature boundaries will misassign upstream/downstream/antisense classes
- Secondary alignments must be excluded first (SAMtools view -F 2048), otherwise read fractions are double-counted
- Contaminant detection is limited to sequences represented in the references used (e.g. E. coli); unmapped reads require separate BLAST-class searches such as against the NCBI nr/nt nucleotide collection

## Evidence

- [discussion] SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [discussion] Minimap2 ONT alignment to plasmid reference: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] Unmapped read contamination analysis against E. coli reference: "This FASTA file was aligned to an E.coli reference sequence utilising Minimap2"
- [readme] Mana analyses ONT alignments to report QC statistics: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] Pre-built docker image for Mana: "docker pull scchess/mana && docker tag scchess/mana mana"

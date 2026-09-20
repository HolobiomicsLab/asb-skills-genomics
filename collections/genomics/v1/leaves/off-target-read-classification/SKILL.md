---
name: off-target-read-classification
description: Use when a sorted, indexed BAM alignment of nanopore cDNA or Illumina
  TruSeq reads against an mRNA vaccine reference must be fractionated into on-target
  versus off-target RNA contaminant classes with fractional abundances.
when_to_use_negative:
- Input is raw FASTQ without a sorted, indexed BAM alignment to the mRNA/plasmid reference
  — barcode removal (Porechop), Q>9 filtering, and Minimap2 alignment must happen
  first
- The goal is per-nucleotide error profiles or consensus calling (use pysamstats/bcftools)
  rather than read-level on/off-target fractionation
- Analysis targets plasmid template DNA quality attributes rather than RNA contaminant
  fractions in cDNA libraries
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: SAMtools
  role: Extracts primary aligned reads (SAMtools view -F 2048) from the sorted, indexed
    BAM prior to classification
- name: bedtools
  role: Strand-aware coordinate intersection (-s) of reads against reference mRNA
    feature boundaries to assign on-target reads
- name: Pysam
  role: Python library for loading and manipulating the sorted, indexed BAM and reference
    FASTA during classification
  repo: https://github.com/scchess/Mana
- name: Mana
  role: Command-line toolkit that analyses ONT alignments (.BAM) to report mRNA quality
    control statistics including purity/contamination fractions
  repo: https://github.com/scchess/Mana
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:3e9976f53729c7767ce2c6e5470bdf93d5be9b455ee8718e6c6963ab1d4e6690
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - Mana
  - Pysam
  - SAMtools
  - bedtools
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# off-target-read-classification

## Summary

Classifies aligned sequencing reads from mRNA vaccine (VAX-seq) cDNA libraries into on-target mRNA product versus off-target classes (upstream, downstream, antisense, cryptic TSS-derived, and contaminant reads such as E. coli), and reports the fractional abundance of each class. This quantifies mRNA purity, one of the key quality attributes VAX-seq measures.

## When to use

Use when a sorted, indexed BAM alignment of nanopore cDNA or Illumina TruSeq reads against an mRNA vaccine reference must be fractionated into on-target versus off-target RNA contaminant classes with fractional abundances.

## When NOT to use

- Input is raw FASTQ without a sorted, indexed BAM alignment to the mRNA/plasmid reference — barcode removal (Porechop), Q>9 filtering, and Minimap2 alignment must happen first
- The goal is per-nucleotide error profiles or consensus calling (use pysamstats/bcftools) rather than read-level on/off-target fractionation
- Analysis targets plasmid template DNA quality attributes rather than RNA contaminant fractions in cDNA libraries

## Inputs

- Sorted, indexed BAM file of reads aligned to the plasmid/mRNA reference (e.g. via Minimap2 -ax map-ont for Nanopore)
- Reference FASTA file containing the mRNA feature boundaries (e.g. eGFP mRNA interval)
- Contaminant reference sequences (e.g. E. coli reference) for unmapped-read contamination analysis

## Outputs

- Classification table of fractional abundance per read class (on-target, upstream, downstream, antisense, cryptic TSS, E. coli) written to the output directory
- Per-class read counts relative to total primary aligned reads

## How to apply

Load the sorted, indexed alignment BAM and reference FASTA with pysam/SAMtools from the mounted analysis directory. Extract primary aligned reads with 'SAMtools view -F 2048' to exclude secondary alignments before classification. Use bedtools intersect with strand-aware (-s) coordinate filtering against the reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target, then classify remaining reads by position and orientation relative to the target: upstream, downstream, antisense, and cryptic transcription start site (TSS) reads, plus reads mapping to contaminant sequences such as E. coli. Compute the fractional abundance of each class relative to total reads and write the classification table to the output directory. Validate against published fractions: 92.7% on-target, 7.3% off-target, 0.3% cryptic TSS, and 0.01% E. coli for the nanopore cDNA library, and 1.4% upstream, 5.7% downstream, ~0.6% antisense for the Illumina short-read library.

## Related tools

- **SAMtools** (Extracts primary aligned reads (SAMtools view -F 2048) from the sorted, indexed BAM prior to classification)
- **bedtools** (Strand-aware coordinate intersection (-s) of reads against reference mRNA feature boundaries to assign on-target reads)
- **Pysam** (Python library for loading and manipulating the sorted, indexed BAM and reference FASTA during classification) — https://github.com/scchess/Mana
- **Mana** (Command-line toolkit that analyses ONT alignments (.BAM) to report mRNA quality control statistics including purity/contamination fractions) — https://github.com/scchess/Mana

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Nanopore cDNA library recovers ~92.7% of reads aligned to the on-target mRNA product and ~7.3% off-target
- E. coli contamination fraction is ~0.01% and cryptic TSS-derived reads ~0.3% for the nanopore cDNA library
- Illumina short-read library recovers ~1.4% upstream, ~5.7% downstream, and ~0.6% antisense reads
- Read counts classified per class sum to the total number of primary aligned reads (SAMtools view -F 2048 output)
- On-target assignments agree with strand-aware bedtools intersect results over the reference mRNA interval

## Limitations

- Published fractions are proof of concept from small samples (n = 1 to n = 4); no statistical method was used to predetermine sample size
- Nanopore cDNA fractions are specific to the eGFP mRNA VAX-seq libraries; other constructs will yield different class abundances
- Contamination analysis depends on the choice of contaminant reference (e.g. the E. coli reference sequence used via Minimap2)

## Evidence

- [methods] primary aligned read extraction: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [results] fractional abundances: [evidence span withheld — anchor status: not_found]
- [readme] Mana purpose: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [abstract] purity as a quality attribute: "VAX-seq can comprehensively measure key mRNA vaccine quality attributes, including sequence, length, integrity, and purity"

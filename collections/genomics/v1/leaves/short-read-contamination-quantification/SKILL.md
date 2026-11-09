---
name: short-read-contamination-quantification
description: Use when sequencing cDNA libraries of an in vitro-transcribed mRNA product
  (Illumina TruSeq short reads or ONT long reads) and you need to compute the fraction
  of reads attributable to the intended mRNA versus off-target RNAs and bacterial
  contamination.
when_to_use_negative:
- Input is still raw ONT FASTQ from plasmid DNA template sequencing — use the plasmid
  mapping/fate-fraction workflow (Minimap2 -ax map-ont) instead, not mRNA read classification.
- The goal is per-nucleotide error profiles or consensus generation (pysamstats/bcftools),
  not read-class fraction quantification.
- Reads have not been adapter/barcode processed (Porechop for ONT, TrimGalore Q20
  for Illumina), so fractions would be biased by technical artefacts.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: SAMtools
  role: Generates sorted/indexed BAMs and extracts primary aligned reads via 'samtools
    view -F 2048' before classification
- name: Minimap2
  role: Aligns reads (-ax map-ont for Nanopore) and realigns unmapped reads to an
    E. coli reference to quantify bacterial contamination
- name: bedtools
  role: Strand-aware coordinate filtering ('bedtools intersect -s') against mRNA feature
    boundaries to assign on-target reads
- name: Pysam
  role: Python library used by Mana to load BAM/FASTA and compute read classifications
    and fractions
  repo: https://github.com/scchess/Mana
- name: Mana
  role: Analyses alignments (.BAM) to produce automated mRNA quality reports including
    contamination fractions
  repo: https://github.com/scchess/Mana
- name: TrimGalore
  role: Quality trims Illumina reads to the Q20 threshold prior to alignment and classification
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:d288e5d277f4369826d8350ba53f90c30d715c39e0b61d7d5ddaac393ce601e2
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
  - TrimGalore
  - bedtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# short-read-contamination-quantification

## Summary

Quantifies the fractional abundance of on-target versus off-target RNA in VAX-seq cDNA libraries of synthetic mRNA (e.g. eGFP mRNA), classifying reads as on-target mRNA, upstream, downstream, antisense, cryptic TSS-derived, or contaminant (e.g. E. coli) using SAMtools/bedtools/Pysam-based alignment classification and Mana.

## When to use

Use when sequencing cDNA libraries of an in vitro-transcribed mRNA product (Illumina TruSeq short reads or ONT long reads) and you need to compute the fraction of reads attributable to the intended mRNA versus off-target RNAs and bacterial contamination.

## When NOT to use

- Input is still raw ONT FASTQ from plasmid DNA template sequencing — use the plasmid mapping/fate-fraction workflow (Minimap2 -ax map-ont) instead, not mRNA read classification.
- The goal is per-nucleotide error profiles or consensus generation (pysamstats/bcftools), not read-class fraction quantification.
- Reads have not been adapter/barcode processed (Porechop for ONT, TrimGalore Q20 for Illumina), so fractions would be biased by technical artefacts.

## Inputs

- Sorted, indexed BAM alignment file of trimmed cDNA reads against the mRNA/reference sequence
- Reference FASTA of the mRNA product (and contaminant reference, e.g. E. coli genome)
- BED file of reference mRNA feature boundaries
- Illumina FASTQ.gz reads trimmed with TrimGalore to the Q20 threshold (upstream of alignment)

## Outputs

- Classification table of read-fate fractions (on-target, upstream, downstream, antisense, cryptic TSS, E. coli) per library
- Mana automated mRNA quality report over the input BAM

## How to apply

Load the sorted, indexed alignment BAM and reference FASTA (via pysam/SAMtools) from the mounted analysis directory, first excluding secondary alignments with 'SAMtools view -F 2048' to keep only primary aligned reads. Use strand-aware 'bedtools intersect -s' against the reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target. Classify the remaining reads by position and orientation relative to the target as upstream, downstream, antisense, or cryptic transcription start site (TSS)-derived, plus reads mapping to contaminant sequences such as E. coli (identified by realigning unmapped reads to an E. coli reference with Minimap2). Compute each class's fractional abundance relative to total reads and write the classification table with Mana, which analyses alignments (.BAM) into automated mRNA quality reports.

## Related tools

- **SAMtools** (Generates sorted/indexed BAMs and extracts primary aligned reads via 'samtools view -F 2048' before classification)
- **Minimap2** (Aligns reads (-ax map-ont for Nanopore) and realigns unmapped reads to an E. coli reference to quantify bacterial contamination)
- **bedtools** (Strand-aware coordinate filtering ('bedtools intersect -s') against mRNA feature boundaries to assign on-target reads)
- **Pysam** (Python library used by Mana to load BAM/FASTA and compute read classifications and fractions) — https://github.com/scchess/Mana
- **Mana** (Analyses alignments (.BAM) to produce automated mRNA quality reports including contamination fractions) — https://github.com/scchess/Mana
- **TrimGalore** (Quality trims Illumina reads to the Q20 threshold prior to alignment and classification)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Nanopore cDNA library fractions should reproduce 92.7% on-target, 7.3% off-target, 0.3% cryptic TSS, and 0.01% E. coli.
- Illumina short-read library fractions should reproduce ~1.4% upstream, 5.7% downstream, and ~0.6% antisense.
- Class fractions should sum to approximately 100% of total (primary-aligned plus classified) reads.
- Read length distributions computed from 'samtools view -F 2048' primary reads should match the source BAM read counts (no secondary alignments double-counted).
- Mana report completes over the mounted BAM (analysis time about 10 minutes) and emits the classification table to the output directory.

## Limitations

- Analysis depended on a custom pipeline (Mana); small sample sizes (n = 1 to n = 4) mean results are proof of concept.
- Experiments were not randomised and investigators were not blinded, as template quality is the most significant determinant of sequencing quality.
- cDNA sequencing was found more reliable than direct RNA sequencing; direct RNA results should be treated accordingly.

## Evidence

- [results] In VAX-seq cDNA libraries of the eGFP mRNA, the majority of sequences (92.7%) aligned to the on-target mRNA product, with few reads (0.01%) detecting E. coli contamination: [evidence span withheld — anchor status: not_found]
- [discussion] SAMtools primary-read extraction: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [discussion] Illumina Q20 trimming: "quality trimmed utilising TrimGalore (v0.6.8dev) to create FASTQ.gz files with reads passing the Q20 threshold"
- [discussion] E. coli contamination realignment: "This FASTA file was aligned to an E.coli reference sequence utilising Minimap2"
- [readme] Mana purpose: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."

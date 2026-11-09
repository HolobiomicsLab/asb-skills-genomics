---
name: bam-coordinate-strand-filtering
description: Use when you have a sorted, indexed BAM of reads aligned to a plasmid
  or mRNA reference and need to partition reads into on-target mRNA product versus
  off-target classes (upstream, downstream, antisense, cryptic TSS, E. coli contaminants)
  with fractional abundances.
when_to_use_negative:
- Input is raw FASTQ reads not yet aligned and sorted/indexed to a reference BAM —
  alignment (e.g. Minimap2 -ax map-ont, SAMtools sort/index) must come first
- The analysis goal is per-nucleotide error or variation profiling rather than read-level
  on/off-target classification — use pysamstats variation/coverage statistics instead
- Only aggregate QC metrics (read length, poly(A) tail, sequence identity) are needed
  and read-coordinate classification relative to feature boundaries is not required
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3512
tools:
- name: SAMtools
  role: Extracts primary aligned reads with 'samtools view -F 2048' and generates
    sorted, indexed BAM files (v1.15 in the article)
- name: bedtools
  role: Performs strand-aware (-s) intersect of primary reads against reference mRNA
    feature boundaries to assign on-target/off-target classes
  repo: https://bedtools.readthedocs.io/en/latest
- name: Pysam
  role: Python library for loading and manipulating the sorted BAM and reference FASTA
    during classification
  repo: https://pysam.readthedocs.io/en/latest/api.html
- name: Mana
  role: Automated command-line toolkit that analyses ONT alignments (.BAM) to report
    mRNA quality control statistics including these read classifications
  repo: https://github.com/scchess/Mana
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:c6290aa6cda475b8ecc18db4b34bb0cb894952ff67b8c582f6d959b50f2ab950
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mana
  - Pysam
  - SAMtools
  - bedtools
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# bam-coordinate-strand-filtering

## Summary

Classifies primary-aligned sequencing reads from mRNA vaccine (VAX-seq) cDNA libraries as on-target versus off-target RNAs by intersecting sorted BAM alignments with reference mRNA feature boundaries in a strand-aware manner, quantifying cryptic TSS, antisense, upstream/downstream, and contaminant (e.g. E. coli) read fractions.

## When to use

Use when you have a sorted, indexed BAM of reads aligned to a plasmid or mRNA reference and need to partition reads into on-target mRNA product versus off-target classes (upstream, downstream, antisense, cryptic TSS, E. coli contaminants) with fractional abundances.

## When NOT to use

- Input is raw FASTQ reads not yet aligned and sorted/indexed to a reference BAM — alignment (e.g. Minimap2 -ax map-ont, SAMtools sort/index) must come first
- The analysis goal is per-nucleotide error or variation profiling rather than read-level on/off-target classification — use pysamstats variation/coverage statistics instead
- Only aggregate QC metrics (read length, poly(A) tail, sequence identity) are needed and read-coordinate classification relative to feature boundaries is not required

## Inputs

- Sorted, indexed alignment BAM file (e.g. ONT cDNA reads aligned to plasmid reference via Minimap2 -ax map-ont)
- Reference FASTA file containing the mRNA/plasmid target sequence
- Reference mRNA feature boundaries (target interval coordinates for strand-aware intersection)
- Optionally, unmapped-read FASTA for contaminant alignment (e.g. against an E. coli reference with Minimap2)

## Outputs

- Classification table of read fractions: on-target, upstream, downstream, antisense, cryptic TSS, and contaminant (E. coli) percentages
- Primary-aligned read set (SAMtools view -F 2048) for downstream read-length distribution and QC statistics

## How to apply

Load the sorted, indexed alignment BAM and reference FASTA (e.g. with pysam/SAMtools) from the mounted analysis directory. Extract primary aligned reads with 'samtools view -F 2048' to exclude secondary alignments before classification, matching the article's use of SAMtools v1.15 for primary-read extraction. Use bedtools intersect with strand-aware (-s) coordinate filtering against the reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target. Classify the remaining reads by position and orientation relative to the target — upstream, downstream, antisense, and cryptic transcription start site (TSS) reads — plus reads mapping to contaminant sequences such as E. coli. Compute each class's fractional abundance relative to total reads and write the classification table to the output directory; the Mana toolkit (SAMtools 1.5.1, BCFtools 1.16, Pysam 0.20.0, bedtools 2.30.0) automates these QC statistics.

## Related tools

- **SAMtools** (Extracts primary aligned reads with 'samtools view -F 2048' and generates sorted, indexed BAM files (v1.15 in the article))
- **bedtools** (Performs strand-aware (-s) intersect of primary reads against reference mRNA feature boundaries to assign on-target/off-target classes) — https://bedtools.readthedocs.io/en/latest
- **Pysam** (Python library for loading and manipulating the sorted BAM and reference FASTA during classification) — https://pysam.readthedocs.io/en/latest/api.html
- **Mana** (Automated command-line toolkit that analyses ONT alignments (.BAM) to report mRNA quality control statistics including these read classifications) — https://github.com/scchess/Mana

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- Reproduce the nanopore cDNA library fractions: 92.7% on-target mRNA, 7.3% off-target, 0.3% cryptic TSS, and 0.01% E. coli contamination
- Recover the Illumina short-read library fractions: 1.4% upstream, 5.7% downstream, and ~0.6% antisense
- Confirm that on-target + off-target fractions sum to ~100% of primary aligned reads (no reads lost by the -F 2048 filter)
- Verify strand-aware classification: antisense reads are only detected when intersect uses -s; without it the antisense fraction collapses into on-target counts
- Check the output classification table schema contains fractional abundances relative to total reads for every class

## Limitations

- Classifications depend on correct mRNA feature boundaries in the reference; mis-specified intervals misassign upstream/downstream/cryptic TSS reads
- Filtering to primary alignments (-F 2048) discards secondary alignments, so multi-mapping reads contribute only their primary record to class fractions
- Fractional abundances are relative to total reads, so low-level contaminants (e.g. 0.01% E. coli) are near detection limits and sensitive to sequencing depth

## Evidence

- [methods] SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [other] Use bedtools intersect with strand-aware (-s) coordinate filtering against the reference mRNA feature boundaries to assign reads overlapping the eGFP mRNA interval as on-target.: [evidence span withheld — anchor status: not_found]
- [results] the majority of sequences (92.7%) aligned to the on-target mRNA product, with few reads (0.01%) detecting E. coli contamination and the remaining 7.3% comprising off-target RNAs: [evidence span withheld — anchor status: not_found]
- [readme] Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics.: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] The program requires SAMtools, BCFtools, Pysam and bedtools as dependencies: "* [bedtools 2.30.0](https://bedtools.readthedocs.io/en/latest)"

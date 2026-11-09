---
name: cryptic-transcription-start-site-detection
description: Use when analysing ONT cDNA (or Illumina TruSeq) alignments of an IVT
  mRNA product and you need to split reads into on-target mRNA versus off-target RNA
  classes, including reads derived from cryptic transcription start sites.
when_to_use_negative:
- Input is raw FASTQ that has not yet been barcode-trimmed (Porechop) and aligned
  to a reference to produce a sorted, indexed BAM
- The question is poly(A) tail length, sequence identity, or nucleoside-modification
  chemistry rather than off-target RNA classification
- No reference mRNA feature boundaries exist to define the on-target interval for
  strand-aware intersect
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: SAMtools
  role: Extracts primary aligned reads with `samtools view -F 2048` (excluding secondary
    alignments) and generates the sorted, indexed BAM consumed for classification
  repo: http://www.htslib.org
- name: bedtools
  role: Strand-aware intersect (-s) of primary reads against the reference mRNA feature
    boundaries to assign on-target versus positional off-target classes
  repo: https://bedtools.readthedocs.io/en/latest
- name: Pysam
  role: Python/BAM access layer used to load the alignment and reference and compute
    fractional abundances of each read class
  repo: https://pysam.readthedocs.io/en/latest/api.html
- name: Mana
  role: Command-line toolkit that analyses the ONT alignment BAM to produce automated
    mRNA quality-control reports, including off-target RNA statistics
  repo: https://github.com/scchess/Mana
- name: Minimap2
  role: Aligns extracted off-target FASTA to contaminant references (e.g. E. coli)
    with -ax map-ont to quantify contamination
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:90f4e124c546e12ad9a14c8a3663d10088590d802784dec313c4d5e273a8dd54
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Mana
  - Minimap2
  - Pysam
  - SAMtools
  - bedtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cryptic-transcription-start-site-detection

## Summary

A VAX-seq analysis skill that classifies reads in sorted, indexed BAM alignments of in-vitro-transcribed mRNA into on-target versus off-target classes — upstream, downstream, antisense, cryptic transcription start site (TSS)-derived, and contaminant reads such as E. coli — to quantify mRNA product purity.

## When to use

Use when analysing ONT cDNA (or Illumina TruSeq) alignments of an IVT mRNA product and you need to split reads into on-target mRNA versus off-target RNA classes, including reads derived from cryptic transcription start sites.

## When NOT to use

- Input is raw FASTQ that has not yet been barcode-trimmed (Porechop) and aligned to a reference to produce a sorted, indexed BAM
- The question is poly(A) tail length, sequence identity, or nucleoside-modification chemistry rather than off-target RNA classification
- No reference mRNA feature boundaries exist to define the on-target interval for strand-aware intersect

## Inputs

- Sorted, indexed BAM file of ONT cDNA reads (or Illumina TruSeq reads) aligned to the plasmid/mRNA reference
- Reference FASTA of the mRNA/plasmid construct
- Reference mRNA feature boundaries (interval BED) for strand-aware intersect
- Contaminant reference sequences (e.g. E. coli reference genome) for off-target mapping

## Outputs

- Per-class read classification (on-target, upstream, downstream, antisense, cryptic TSS, E. coli contaminant)
- Fractional abundance table of each class relative to total primary reads

## How to apply

Load the sorted, indexed alignment BAM and reference FASTA with pysam/SAMtools, then extract primary aligned reads with `samtools view -F 2048` so secondary alignments do not inflate class fractions. Use strand-aware bedtools intersect (-s) against the reference mRNA feature boundaries to assign reads overlapping the mRNA interval as on-target; classify the remainder by position and orientation relative to the target as upstream, downstream, antisense, or cryptic TSS-derived, and align leftover unmapped reads to contaminant references (e.g. E. coli) with Minimap2. Compute each class as a fraction of total primary reads and write the classification table to the output directory; Mana automates reporting of these quality-control statistics from the ONT BAM.

## Related tools

- **SAMtools** (Extracts primary aligned reads with `samtools view -F 2048` (excluding secondary alignments) and generates the sorted, indexed BAM consumed for classification) — http://www.htslib.org
- **bedtools** (Strand-aware intersect (-s) of primary reads against the reference mRNA feature boundaries to assign on-target versus positional off-target classes) — https://bedtools.readthedocs.io/en/latest
- **Pysam** (Python/BAM access layer used to load the alignment and reference and compute fractional abundances of each read class) — https://pysam.readthedocs.io/en/latest/api.html
- **Mana** (Command-line toolkit that analyses the ONT alignment BAM to produce automated mRNA quality-control reports, including off-target RNA statistics) — https://github.com/scchess/Mana
- **Minimap2** (Aligns extracted off-target FASTA to contaminant references (e.g. E. coli) with -ax map-ont to quantify contamination)

## Examples

```
docker run -v ${PWD}:/src -i -t mana python3 mana.py --mrna -b /src/alignment.bam -f /src/file.fasta -o /src/out
```

## Evaluation signals

- For the eGFP mRNA nanopore cDNA library, the reported fractions are recovered: 92.7% on-target mRNA, 7.3% off-target, 0.3% cryptic TSS, and 0.01% E. coli
- For the Illumina short-read library, the off-target split is recovered as ~1.4% upstream, ~5.7% downstream, and ~0.6% antisense
- Class fractions sum to approximately 1 when normalised against total primary reads (after -F 2048 filtering)
- No secondary alignments (flag 2048) remain in the reads entering classification
- Each classified read overlaps the mRNA interval on the correct strand (bedtools -s) or is assigned to exactly one off-target class

## Limitations

- Reported fractions are proof-of-concept values from small samples (n = 1 to n = 4), without sample-size statistics, randomisation, or blinding
- Direct RNA sequencing was found less reliable than cDNA sequencing by the authors, so cryptic TSS quantification is best anchored on cDNA libraries
- A pre-built Mana docker image is the recommended install; building from source takes >= 20 minutes and is not recommended

## Evidence

- [discussion] SAMtools view -F 2048 filters primary alignments: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"
- [discussion] Sorted indexed BAM generation: "The resulting SAM alignment files were processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [other] Recovered cryptic TSS fraction: "0.3% were likely derived from cryptic transcription start sites"
- [readme] Mana analyses ONT BAM for QC: "Command-line tool that analyses ONT alignments (.BAM) to report quality control statistics."
- [readme] Bedtools dependency for Mana: "* [bedtools 2.30.0](https://bedtools.readthedocs.io/en/latest)"

---
name: bam-sorting-and-indexing
description: Use when raw SAM alignments from Nanopore/cDNA/dRNA reads must be converted
  to sorted, indexed BAM before read-fate QC, per-nucleotide error profiling, or genome-viewer
  inspection.
when_to_use_negative:
- Input is already a sorted, indexed BAM ready for Mana or pysamstats — re-sorting
  adds no value
- The analysis goal is raw read QC (read length/N50) that operates directly on FASTQ
  and does not require alignment, e.g. NanoPlot v1.38.1 summaries
- Alignments are short-read Illumina mappings from a different aligner workflow where
  the map-ont preset and this SAMtools step were not part of the validated chain
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3270
tools:
- name: SAMtools v1.15
  role: Converts Minimap2 SAM output into sorted and indexed BAM files and extracts
    primary aligned reads via 'samtools view -F 2048'
- name: Minimap2 (Release 2.20-r1064)
  role: Upstream aligner producing the SAM files that this skill sorts and indexes
- name: Mana
  role: Downstream consumer of the sorted BAM (docker image scchess/mana, --mrna mode,
    -b alignment.bam)
  repo: https://github.com/scchess/Mana
- name: IGV v2.12.3
  role: Visualises the generated BAM files for manual inspection
- name: Pysam
  role: Dependency (0.20.0) of Mana used for per-nucleotide statistics on the indexed
    BAM
  repo: https://github.com/alimanfoo/pysamstats
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-41354-y
    title: mRNA vaccine quality analysis using RNA sequencing
  source_content_hash: sha256:6b5f641283684f44eb62b3907844960a948bff8d93d2d79504bd026ff5bacd12
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scchess/Mana
  tools:
  - IGV v2.12.3
  - Mana
  - Minimap2
  - Minimap2 (Release 2.20-r1064)
  - Pysam
  - SAMtools
  - SAMtools v1.15
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3270
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# bam-sorting-and-indexing

## Summary

Convert Minimap2 SAM alignments into sorted, indexed BAM files with SAMtools v1.15 so downstream QC tools (Mana, pysamstats, IGV) can consume them. This is a prerequisite step in the VAX-seq mRNA vaccine quality pipeline.

## When to use

Use when raw SAM alignments from Nanopore/cDNA/dRNA reads must be converted to sorted, indexed BAM before read-fate QC, per-nucleotide error profiling, or genome-viewer inspection.

## When NOT to use

- Input is already a sorted, indexed BAM ready for Mana or pysamstats — re-sorting adds no value
- The analysis goal is raw read QC (read length/N50) that operates directly on FASTQ and does not require alignment, e.g. NanoPlot v1.38.1 summaries
- Alignments are short-read Illumina mappings from a different aligner workflow where the map-ont preset and this SAMtools step were not part of the validated chain

## Inputs

- SAM alignment file produced by Minimap2 '-ax map-ont' (Nanopore reads vs plasmid or mRNA reference)
- Concatenated FASTQ reads from ONT runs (SQK-LSK109 pDNA or SQK-RNA002 dRNA), previously barcode-trimmed with Porechop v0.2.4

## Outputs

- Coordinate-sorted BAM file
- BAM index (.bai) enabling random access by region
- Downstream-ready alignment for Mana QC and pysamstats variation statistics

## How to apply

After aligning Nanopore reads (e.g. eGFP plasmid or SQK-RNA002 direct RNA reads) with Minimap2 Release 2.20-r1064 using the '-ax map-ont' preset, process the resulting SAM file with SAMtools v1.15 to generate a sorted BAM and an index. The sorted/indexed BAM is the required input format for Mana (passed as -b /src/alignment.bam to the dockerised scchess/mana image) and for pysamstats per-nucleotide error profiles. Sorting by coordinate and indexing enable random access needed by pileup-based statistics and IGV v2.12.3 visualisation. Use 'samtools view -F 2048' downstream when read-length distributions of primary aligned reads are needed, excluding secondary alignments.

## Related tools

- **SAMtools v1.15** (Converts Minimap2 SAM output into sorted and indexed BAM files and extracts primary aligned reads via 'samtools view -F 2048')
- **Minimap2 (Release 2.20-r1064)** (Upstream aligner producing the SAM files that this skill sorts and indexes)
- **Mana** (Downstream consumer of the sorted BAM (docker image scchess/mana, --mrna mode, -b alignment.bam)) — https://github.com/scchess/Mana
- **IGV v2.12.3** (Visualises the generated BAM files for manual inspection)
- **Pysam** (Dependency (0.20.0) of Mana used for per-nucleotide statistics on the indexed BAM) — https://github.com/alimanfoo/pysamstats

## Examples

```
minimap2 -ax map-ont ref.fasta reads.fastq | samtools sort -o alignment.bam && samtools index alignment.bam
```

## Evaluation signals

- samtools quickcheck passes on the output BAM and a .bai index file exists alongside it
- Sortedness invariant: 'samtools index' succeeds without 'chromosome blocks not continuous' errors
- Read count in the BAM matches the input SAM primary reads (secondary alignments excludable via view -F 2048)
- Downstream Mana run accepts the BAM (docker run -b /src/alignment.bam) and completes in about 10 minutes without format errors
- For task_001 validation, read-fate fractions computed from the BAM reproduce 86.8% plasmid-aligned, 6.8% E. coli, 6.6% failed reads

## Limitations

- SAMtools sorting/indexing alone quantifies nothing — quality fractions (plasmid/E. coli/failed) require a downstream Mana or pysamstats step
- Mismapped or contaminated reads are carried through silently; the pipeline quantifies contamination only after unmapped reads are re-aligned to an E. coli reference
- Mana's docker image requires correct volume mounting (-v ${PWD}:/src) or relative BAM paths will fail

## Evidence

- [discussion] sorted and indexed BAM: "The resulting SAM alignment files were processed via SAMtools v1.15 to generate sorted and indexed BAM files"
- [discussion] minimap2 alignment: "aligned to the plasmid reference via Minimap2 (Release 2.20-r1064) with -ax map-ont for Nanopore"
- [discussion] pysamstats error profile: "The per-nucleotide error profile of mapped reads was determined relative to the reference index sequence using pysamstats"
- [discussion] IGV inspection: "The generated BAM files were viewed and analysed utilising Integrative Genomics Viewer (IGV v2.12.3)"
- [readme] Mana docker invocation: "docker run -v ${PWD}:/src -i -t mana \
               python3 mana.py --mrna \
               -b /src/alignment.bam \
               -f /src/file.fasta"
- [discussion] primary alignment extraction: "SAMtools (v1.15) utilising SAMtools view -F 2048 was used to extract read length distributions of the primary aligned reads"

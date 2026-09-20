---
name: long-read-alignment-processing
description: Use when raw long-read sequencing data (PacBio or ONT) must be turned
  into a sorted, indexed BAM aligned to hg19/mm10 so that breakpoint detection and
  rearrangement annotation can be performed in AgileStructure.
when_to_use_negative:
- The input is short paired-end Illumina read data — minimap2's 'sr' preset or a short-read
  aligner is required, and AgileStructure targets large-scale rearrangements that
  short reads fail to resolve.
- The analysis goal is SNV or small-indel calling, which the paper notes short-read
  sequencing already handles; AgileStructure is aimed at large-scale structural rearrangements.
- A splice-aware alignment of RNA/cDNA reads is needed — minimap2's 'splice' presets,
  not map-ont, apply.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0102
tools:
- name: minimap2
  role: Long-read-specific aligner used to map ONT/PacBio reads to hg19/mm10 with
    the -ax map-ont preset and --secondary=no before SAM output.
  repo: https://github.com/lh3/minimap2
- name: lra
  role: Alternative long-read alignment tool mentioned by the paper for aligning long-read
    sequencing data to the appropriate genome.
  repo: https://github.com/ChaissonLab/LRA
- name: Samtools
  role: Converts the minimap2 SAM output to BAM, sorts it by genomic position, and
    indexes it so AgileStructure can load and screen the alignments.
- name: AgileStructure
  role: Downstream desktop application that consumes the sorted, indexed BAM to locate
    and annotate structural rearrangements; run under WINE on POSIX systems.
  repo: https://github.com/msjimc/AgileStructure
- name: WINE
  role: Compatibility layer enabling AgileStructure, a Windows program, to run on
    Linux/BSD/macOS hosts.
  repo: https://github.com/msjimc/RunningWindowsProgramsOnLinux
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btag294
    title: Utilization of long-read sequencing for the detection of structural rearrangements
      with AgileStructure
  source_content_hash: sha256:fc05af87c0d8a1d48c13fda63092cacf96eff36c11aa7ff24ce5b22c889fb720
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - AgileStructure
  - MakeSVGenome
  - Samtools
  - WINE
  - lra
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# long-read-alignment-processing

## Summary

Align ONT/PacBio long reads to a reference genome with a long-read-specific aligner (minimap2 with the map-ont preset) and convert/sort/index the SAM output to BAM with Samtools, producing the sorted indexed BAM that AgileStructure requires for structural rearrangement detection.

## When to use

Use when raw long-read sequencing data (PacBio or ONT) must be turned into a sorted, indexed BAM aligned to hg19/mm10 so that breakpoint detection and rearrangement annotation can be performed in AgileStructure.

## When NOT to use

- The input is short paired-end Illumina read data — minimap2's 'sr' preset or a short-read aligner is required, and AgileStructure targets large-scale rearrangements that short reads fail to resolve.
- The analysis goal is SNV or small-indel calling, which the paper notes short-read sequencing already handles; AgileStructure is aimed at large-scale structural rearrangements.
- A splice-aware alignment of RNA/cDNA reads is needed — minimap2's 'splice' presets, not map-ont, apply.

## Inputs

- Long-read sequencing reads (FASTQ/FASTA) from ONT or PacBio sequencers, e.g. SRA sample SRR12783028 or SRR9982132
- Reference genome sequence (FASTA) obtained from the UCSC Genome Browser, e.g. hg19 or mm10
- Optionally a minimap2 index (.mmi) built with 'minimap2 -d ref.mmi ref.fa'

## Outputs

- SAM alignment file (CIGAR-containing) from minimap2 -a
- Coordinate-sorted BAM indexed with Samtools (.bam + .bai), ready to load into AgileStructure

## How to apply

Align the reads to the appropriate reference genome (e.g. hg19 from UCSC for human samples, mm10 for mouse) using minimap2 with the '-ax map-ont' preset and the '--secondary=no' parameter — suppressing secondary alignments is essential because AgileStructure clusters primary and secondary junction positions when calling breakpoints. Use the map-pb or map-hifi presets instead for PacBio CLR or HiFi reads per the minimap2 guide. Convert the resulting SAM to BAM, sort by genomic position, and index with Samtools so AgileStructure can screen loci in blocks of up to 10 Mb and call a putative breakpoint where 2 or more split reads have primary/secondary junction positions within 250 nucleotides of each other. On Linux/BSD/macOS systems, run AgileStructure itself via WINE, as it is a Windows desktop application.

## Related tools

- **minimap2** (Long-read-specific aligner used to map ONT/PacBio reads to hg19/mm10 with the -ax map-ont preset and --secondary=no before SAM output.) — https://github.com/lh3/minimap2
- **lra** (Alternative long-read alignment tool mentioned by the paper for aligning long-read sequencing data to the appropriate genome.) — https://github.com/ChaissonLab/LRA
- **Samtools** (Converts the minimap2 SAM output to BAM, sorts it by genomic position, and indexes it so AgileStructure can load and screen the alignments.)
- **AgileStructure** (Downstream desktop application that consumes the sorted, indexed BAM to locate and annotate structural rearrangements; run under WINE on POSIX systems.) — https://github.com/msjimc/AgileStructure
- **WINE** (Compatibility layer enabling AgileStructure, a Windows program, to run on Linux/BSD/macOS hosts.) — https://github.com/msjimc/RunningWindowsProgramsOnLinux

## Examples

```
minimap2 -ax map-ont --secondary=no hg19.fa reads.fq > aln.sam; samtools view -b aln.sam | samtools sort -o sorted.bam; samtools index sorted.bam
```

## Evaluation signals

- The '--secondary=no' parameter was present in the minimap2 command line, matching the paper's workflow on all three exemplar datasets (synthetic, SRR12783028, SRR9982132).
- The SAM output was converted to BAM, coordinate-sorted, and indexed (.bam + .bai pair exists) — AgileStructure requires the indexed BAM to screen loci in blocks of up to 10 Mb.
- Alignment was run with a long-read preset ('-ax map-ont' for ONT) against the correct reference build (hg19 for SRR9982132, mm10 for SRR12783028).
- Downstream, split-read junction positions at true breakpoints cluster within 250 nucleotides with 2 or more reads, indicating the alignment preserved junction-level information AgileStructure needs.
- Reproduced breakpoint coordinates land close to published values (e.g. chr6 breakpoint within -1/+2 bp of g.167,281,717; chr8 within +9/+11 bp of g.113,696,089).

## Limitations

- Indexing parameters such as -k, -w, -H and -I cannot be changed once a minimap2 index is built, so separate indexes are needed for different data types.
- AgileStructure is a Windows desktop application; on Linux/BSD/macOS it must be run via WINE.
- Inverting-read or duplicated regions can produce ambiguous annotation even after correct alignment (ring chromosome vs tandem duplication; offset inversion vs inverted insertion-deletion).
- The MakeSVGenome synthetic data contain no mutations other than the structural rearrangements, so real experimental data (with read noise) may behave differently.

## Evidence

- [intro] Once generated, the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2: "Once generated, the data is typically aligned to the appropriate genome using a long-read-specific aligner such as minimap2"
- [readme] minimap2 map-ont usage: [evidence span withheld — anchor status: external_doc]
- [readme] minimap2 index saving: [evidence span withheld — anchor status: external_doc]
- [intro] AgileStructure runs on POSIX via WINE: "AgileStructure can be run on most POSIX like systems using Intel or AMD CPUs via WINE"
- [abstract] AgileStructure purpose: [evidence span withheld — anchor status: not_found]

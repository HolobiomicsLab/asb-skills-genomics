---
name: short-read-and-long-read-genome-mapping
description: Use when a draft de novo assembly (FASTA/contigs) needs quality assessment
  at single-nucleotide resolution, and matched short-read (Illumina) plus long-read
  (PacBio HiFi, PacBio CLR, or ONT) sequencing data are available to remap onto it.
when_to_use_negative:
- Only one read type is available (NGS-only or SMS-only), since CRAQ integrates coverage
  and clipping signals from both NGS and SMS mapping to classify errors versus heterozygous
  variants
- The genome has ploidy above diploid, where CRAQ's accuracy has not been benchmarked
  and heterozygous/error classification may be unreliable
- The goal is assembly completeness (gene content) assessment rather than error detection
  — use BUSCO instead
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Minimap2
  role: Reads mapper; '-ax sr' preset for NGS short reads and '-ax map-hifi'/'-ax
    map-pb'/'-ax map-ont' presets for SMS long reads mapped against the draft assembly
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Alignment filtering and indexing; 'samtools view -F 1796 -q 20' removes unmapped,
    secondary, QC-failed, PCR-duplicate and low-MAPQ alignments before CRAQ
- name: CRAQ
  role: Downstream consumer of the filtered BAMs; extracts clipped alignments (CIGAR
    S/H) to identify CREs, CSEs, CRHs, CSHs and compute R-AQI/S-AQI
  repo: https://github.com/JiaoLaboratory/CRAQ
- name: Integrative Genomics Viewer (IGV)
  role: Visual inspection of the filtered alignments at candidate error or heterozygous
    loci
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:460d2092b349860f5ccf513ecc1ac886c6f99b8d049e97875fbf1a57bee1aa43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - CRAQ
  - Integrative Genomics Viewer (IGV)
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# short-read-and-long-read-genome-mapping

## Summary

Map NGS short reads and SMS long reads back to a draft genome assembly with Minimap2 ('-ax sr' for Illumina, '-ax map-hifi/map-pb/map-ont' for PacBio HiFi/CLR/ONT), then filter alignments with SAMtools to feed CRAQ's clipping-based error detection. Dual-read-type mapping is the prerequisite step for single-nucleotide-resolution assembly error and heterozygous locus identification.

## When to use

Use when a draft de novo assembly (FASTA/contigs) needs quality assessment at single-nucleotide resolution, and matched short-read (Illumina) plus long-read (PacBio HiFi, PacBio CLR, or ONT) sequencing data are available to remap onto it.

## When NOT to use

- Only one read type is available (NGS-only or SMS-only), since CRAQ integrates coverage and clipping signals from both NGS and SMS mapping to classify errors versus heterozygous variants
- The genome has ploidy above diploid, where CRAQ's accuracy has not been benchmarked and heterozygous/error classification may be unreliable
- The goal is assembly completeness (gene content) assessment rather than error detection — use BUSCO instead

## Inputs

- Draft genome assembly (FASTA contigs/scaffolds, e.g. HiCanu D. melanogaster F1 assembly)
- Illumina NGS short reads (FASTQ)
- SMS long reads (PacBio HiFi, PacBio CLR, or ONT, FASTQ)
- Minimap2 index or reference assembly for mapping

## Outputs

- Filtered NGS short-read BAM (MAPQ >= 20, flags -F 1796)
- Filtered SMS long-read BAM (MAPQ >= 20, flags -F 1796)
- Sorted and indexed alignment files (SAM/BAM) ready for CRAQ clipped-alignment extraction

## How to apply

Map the Illumina short reads to the assembly with Minimap2 using the '-ax sr' preset, and the long reads with '-ax map-hifi' (PacBio HiFi), '-ax map-pb' (PacBio CLR), or '-ax map-ont' (ONT); then sort and index the alignments with SAMtools. Filter both BAM files with 'samtools view -F 1796 -q 20' to remove unmapped, secondary, QC-failed, and PCR-duplicate alignments with MAPQ below 20. The filtered NGS and SMS BAMs are the dual inputs CRAQ requires: it extracts clipped alignments (CIGAR 'S'/'H'), and distinguishes error breakpoints from heterozygous loci using clipping-ratio cutoffs (h = 0.4-0.6 for heterozygous, f = 0.75 for mapping breakpoints) plus coverage filters (m = 2 minimum, M = 5x average coverage maximum). The rationale for using both read types is that NGS clipping reveals small-scale regional errors (CREs/CRHs) while SMS clipping reveals large-scale structural errors (CSEs/CSHs), enabling integration at different error scales.

## Related tools

- **Minimap2** (Reads mapper; '-ax sr' preset for NGS short reads and '-ax map-hifi'/'-ax map-pb'/'-ax map-ont' presets for SMS long reads mapped against the draft assembly) — https://github.com/lh3/minimap2
- **SAMtools** (Alignment filtering and indexing; 'samtools view -F 1796 -q 20' removes unmapped, secondary, QC-failed, PCR-duplicate and low-MAPQ alignments before CRAQ)
- **CRAQ** (Downstream consumer of the filtered BAMs; extracts clipped alignments (CIGAR S/H) to identify CREs, CSEs, CRHs, CSHs and compute R-AQI/S-AQI) — https://github.com/JiaoLaboratory/CRAQ
- **Integrative Genomics Viewer (IGV)** (Visual inspection of the filtered alignments at candidate error or heterozygous loci)

## Examples

```
minimap2 -ax sr assembly.fa shortreads.fq.gz | samtools sort -o ngs.bam; minimap2 -ax map-hifi assembly.fa hifi.fq.gz | samtools sort -o sms.bam; samtools index ngs.bam sms.bam; samtools view -F 1796 -q 20 -b ngs.bam > ngs.filtered.bam
```

## Evaluation signals

- Both filtered BAMs retain only alignments with MAPQ >= 20 and without flags 1796 (unmapped, secondary, QC-failed, PCR-duplicate)
- Downstream CRAQ out_final.Report reproduces published benchmark values — e.g. Table 2 HiCanu D. melanogaster F1: R-AQI ~93.1, S-AQI ~91.9, with 127/3060 (4.2%) clipped loci called true errors (102 CREs, 25 CSEs)
- Heterozygous loci classified where clipping ratio falls in the h region (0.4-0.6) and breakpoints where it exceeds f (0.75)
- Candidates excluded where coverage < 2 or > 5x average coverage, confirming the coverage filters were applied
- Whole-genome AQI values exceed 90 for assemblies intended as reference quality

## Limitations

- Read mapping is the most resource-intensive step; users must manually split query sequences into multiple fragments to parallelize
- Error breakpoint ratios can fall below 100% in practice due to sequencing errors or inaccurate read mapping, complicating classification
- Regions with coverage above the h upper bound and below f are reported as ambiguous heterozygous-or-error without definitive classification
- CRAQ accuracy for polyploid genomes is not established

## Evidence

- [abstract] By default, Minimap2 '–ax sr' and '–ax map-hifi'('map-hifi' for PacBio HiFi,'map-pb' for PacBio CLR, 'map-ont' for ONT library) options were used: "By default, Minimap2 ‘–ax sr’ and  ‘–ax map-hifi’(‘map-hifi’ for PacBio HiFi,‘map-pb’ for PacBio CLR, ‘map-ont’ for ONT library) options were used"
- [discussion] Any read alignments with low mapping quality (MAPQ <20) or that were unmapped, secondary, QC-failed, or PCR-duplicated were filtered out: [evidence span withheld — anchor status: not_found]
- [abstract] CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants: "CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants"
- [discussion] a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6): "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [abstract] Genome Browsers as Integrative Genomics Viewer (IGV) can be used for visually inspecting: "Genome Browsers as Integrative Genomics Viewer (IGV) can be used for visually inspecting"

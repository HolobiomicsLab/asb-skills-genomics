---
name: clip-based-assembly-error-detection
description: Use when a draft genome assembly must be pinpointed for regional or structural
  errors from clipping signals of NGS short reads and SMS long reads mapped back to
  the assembly (CRAQ CRE/CSE workflow).
when_to_use_negative:
- 'Polyploid genomes: the article notes evaluation of assemblies for species with
  higher ploidy levels may not be as accurate as the benchmarked diploid cases'
- When the goal is consensus polishing or variant calling from reads (e.g., medaka-style
  neural-network polishing) rather than locating misassemblies in the draft assembly
- When only coverage- or completeness-based assessment is needed (e.g., BUSCO, QUAST)
  and no read-mapping clipping signal is available
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3168
tools:
- name: CRAQ
  role: Extracts clipped reads (CIGAR S/H) from filtered BAMs, computes per-position
    clipping ratios and coverage, classifies CRE/CSE breakpoints and heterozygous
    loci, and computes AQI scores
  repo: github.com/JiaoLaboratory/CRAQ
- name: Minimap2
  role: Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont')
    back to the draft assembly to produce the clipped alignments
  repo: github.com/lh3/minimap2
- name: SAMtools
  role: Filters alignments with 'samtools view -F 1796 -q 20' (MAPQ >=20, no unmapped/secondary/QC-fail/duplicate
    records) before clipping extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-42336-w
    title: Identification of errors in draft genome assemblies at single-nucleotide
      resolution for quality assessment and improvement
  source_content_hash: sha256:2f576e4f993e1d3ce196f86e95c99f05c0a9fd668f5b203b21ac1f5ea0b56791
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CRAQ
  - Minimap2
  - Perl
  - SAMtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# clip-based-assembly-error-detection

## Summary

Detects assembly errors in a draft genome by mining soft/hard-clipped (CIGAR S/H) alignments of NGS short reads and SMS long reads, converting per-position clipping ratios and coverage into CRE/CSE breakpoints and R-AQI/S-AQI quality scores at single-nucleotide resolution, as implemented in CRAQ.

## When to use

Use when a draft genome assembly must be pinpointed for regional or structural errors from clipping signals of NGS short reads and SMS long reads mapped back to the assembly (CRAQ CRE/CSE workflow).

## When NOT to use

- Polyploid genomes: the article notes evaluation of assemblies for species with higher ploidy levels may not be as accurate as the benchmarked diploid cases
- When the goal is consensus polishing or variant calling from reads (e.g., medaka-style neural-network polishing) rather than locating misassemblies in the draft assembly
- When only coverage- or completeness-based assessment is needed (e.g., BUSCO, QUAST) and no read-mapping clipping signal is available

## Inputs

- Raw NGS short-read FASTQ and SMS long-read FASTQ (PacBio HiFi/CLR or ONT)
- Draft assembly FASTA used as the mapping reference
- Filtered NGS alignment BAM (SR_sort.bam, after 'samtools view -F 1796 -q 20')
- Filtered SMS alignment BAM (LR_sort.bam, after 'samtools view -F 1796 -q 20')

## Outputs

- Per-position clipping tables (SR_clip.coverRate and LR_clip.coverRate equivalents) with columns chr, position, strand, number of clipped-reads, and total coverage at that position
- Coordinates of Clip-based Regional Errors (CREs) and Clip-based Structural Errors (CSEs)
- Coordinates of regional heterozygous variants (CRHs) and structural heterozygous variants (CSHs)
- Normalized R-AQI and S-AQI assembly quality scores (with AQI thresholds: >90 reference quality, 80-90 high quality, 60-80 draft quality, <60 low quality)

## How to apply

Map raw NGS short reads and SMS long reads (PacBio HiFi/CLR or ONT) to the draft assembly with Minimap2 using '-ax sr' and '-ax map-hifi/map-pb/map-ont', then remove poor alignments with 'samtools view -F 1796 -q 20', which drops MAPQ <20, unmapped, secondary, QC-failed, and PCR-duplicate records. Iterate the filtered BAMs (SR_sort.bam, LR_sort.bam) and select every read whose CIGAR string contains an S (soft-clip) or H (hard-clip) operation, recording the mapped reference coordinate of the clip and its orientation as strand '+' for left-clipped or '-' for right-clipped. At each genomic position tally the number of clipped reads against total mapped coverage and write the SR_clip.coverRate and LR_clip.coverRate tables with columns chr, position, strand, number of clipped-reads, and total coverage. Classify loci by clipping ratio: a ratio within h (default 0.4-0.6) marks the locus as heterozygous (CRH/CSH), while a ratio exceeding f (default 0.75) calls a mapping breakpoint; exclude candidates with coverage below m (default 2) or above M (default 5x average coverage), and filter multi-mapped reads when identifying CSE breakpoints. Finally classify breakpoints as CREs (small-scale) or CSEs (structural), break conflicting contigs at CSE positions if correcting, and normalize CRE/CSE counts into R-AQI and S-AQI scores.

## Related tools

- **CRAQ** (Extracts clipped reads (CIGAR S/H) from filtered BAMs, computes per-position clipping ratios and coverage, classifies CRE/CSE breakpoints and heterozygous loci, and computes AQI scores) — github.com/JiaoLaboratory/CRAQ
- **Minimap2** (Maps NGS short reads ('-ax sr') and SMS long reads ('-ax map-hifi/map-pb/map-ont') back to the draft assembly to produce the clipped alignments) — github.com/lh3/minimap2
- **SAMtools** (Filters alignments with 'samtools view -F 1796 -q 20' (MAPQ >=20, no unmapped/secondary/QC-fail/duplicate records) before clipping extraction)

## Examples

```
samtools view -F 1796 -q 20 SR_sort.bam > SR_sort.filtered.bam
```

## Evaluation signals

- Both output clipping tables exist and contain exactly the five required columns: chr, position, strand, number of clipped-reads, and total coverage
- At every position, the reported clipped-read count never exceeds the total mapped-read coverage
- Input BAMs contain no records with MAPQ <20, unmapped, secondary, QC-failed, or PCR-duplicate flags (i.e., '-F 1796 -q 20' was applied)
- Heterozygous calls (CRHs/CSHs) fall within the clipping-ratio window h = 0.4-0.6, while breakpoint calls exceed f = 0.75
- No candidate is called at coverage below m (2) or above M (5x the average coverage), and multi-mapped reads are excluded from CSE breakpoint identification

## Limitations

- CSE/CRE classification depends on a user-tunable coverage discrepancy parameter (-d) comparing SMS coverage, with no stated universal value
- The ratio for assembly errors can be lower than 100% in practice due to sequencing errors or inaccurate read mapping
- Regions with coverage above the upper level of h and below the f value are reported as ambiguous heterozygous-or-error regions without definitive classification
- Read mapping is the most resource-intensive step; users must manually split query sequences into multiple fragments to parallelize

## Evidence

- [discussion] CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string: "CRAQ first extracts all clipped reads, coded as “S” or “H” in the Compact Idiosyncratic Gapped Alignment Report (CIGAR) string"
- [discussion] filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35: "filtered out using the ‘-F 1796 -q 20’ parameters in ‘samtools view’35"
- [discussion] a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6): "a locus is classified as heterozygous when the clipping ratio of NGS/SMS mapping is within a cutoff region h (default = 0.4–0.6)"
- [discussion] A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75): "A region is classified as a mapping breakpoint when the ratio exceeds a stringent cutoff value f (default = 0.75)"
- [abstract] CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants: "CRAQ integrates the reads-mapping status (reads coverage & clipping signals) of NGS short-reads and SMS long-reads to detect types of assembly errors and heterozygous variants"
- [readme] CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator that can assess the accuracy of assembled genomic sequences: "CRAQ (Clipping Reveals Assembly Quality) is a reference-free genome assembly evaluator that can assess the accuracy of assembled genomic sequences"

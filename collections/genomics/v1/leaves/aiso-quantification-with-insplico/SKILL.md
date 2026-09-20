---
name: aiso-quantification-with-insplico
description: Use when RNA-seq data (short- or long-read) must be converted into per-exon
  AISO classes and Fupfi values — e.g. to compare splicing order across replicates,
  cell types, tissues, species or spliceosomal disruption conditions.
when_to_use_negative:
- You need splicing-order information beyond the introns directly flanking the input
  exon — short reads can barely provide this; long reads are required
- The exon of interest is longer than what read length/paired-end fragment size can
  support — read length limits the maximum exon length quantifiable by Insplico with
  short reads
- The question is differential transcript isoform usage rather than adjacent-intron
  splicing order — use a standard PSI-based tool instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Insplico
  role: Standalone software that quantifies AISO (upfi, dofi, bus, bos counts and
    Fupfi) from short- and long-read RNA-seq; includes an optional module correcting
    5'/3' long-read biases
  repo: gitlab.com/aghr/insplico
- name: STAR
  role: 2-pass splice-aware aligner producing BAMs feeding Insplico; run with --chimSegmentMin
    20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax
    100
- name: R
  role: Environment for annotation import (rtracklayer), simulation of UPFI/DOFI transcripts,
    and downstream consistency statistics with dplyr and ggplot2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:f2d3c1a54b52eb4065fcdabff1347c395dca8d058ab2992a250930d95459c488
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  - STAR
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# aiso-quantification-with-insplico

## Summary

Quantifies the splicing order of adjacent introns (AISO) genome-wide around user-specified exons using the standalone tool Insplico on short- and long-read RNA-seq data, classifying exons as upfi (upstream intron first), dofi (downstream intron first), bus or bos and estimating Fupfi.

## When to use

Use when RNA-seq data (short- or long-read) must be converted into per-exon AISO classes and Fupfi values — e.g. to compare splicing order across replicates, cell types, tissues, species or spliceosomal disruption conditions.

## When NOT to use

- You need splicing-order information beyond the introns directly flanking the input exon — short reads can barely provide this; long reads are required
- The exon of interest is longer than what read length/paired-end fragment size can support — read length limits the maximum exon length quantifiable by Insplico with short reads
- The question is differential transcript isoform usage rather than adjacent-intron splicing order — use a standard PSI-based tool instead

## Inputs

- BAM files of RNA-seq reads aligned with STAR to hg38 or mm10 (2-pass, --chimSegmentMin 20)
- Ensembl v88 genome annotation (GTF, e.g. Hsa38_only_transcript_support_level_1.gtf filtered to transcript_support_level 1)
- VastDB exon tables for hg38/mm10
- User-specified list of exons of interest (BED/GTF coordinates)
- Chromosome FASTA files (chr1-chrY) in a chroms/ folder

## Outputs

- Per-exon upfi, dofi, bus and bos read counts
- Fupfi (fraction of UPFI), PSI and PIR estimates around each user-specified exon
- Per-pair AISO class maintenance fractions and Pearson/Spearman correlations of Fupfi (aiso_consistency_statistics.tsv)

## How to apply

Align reads to hg38/mm10 with STAR in 2-pass mode (e.g. --chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100), then run Insplico with hg38/mm10 Ensembl v88 and VastDB exon tables on the chromatin-associated RNA-seq samples (GEO GSE207459). Insplico returns upfi, dofi, bus and bos counts plus Fupfi (fraction of UPFI) around each input exon; for long-read data, apply Insplico's optional module to correct 5' and 3' biases in sequencing and library preparation so long-read results harmonize with short reads. Classify exons as upfi when Fupfi >= 0.8 and dofi when Fupfi <= 0.2, retaining only exons with at least 10 informative reads (N(upfi+dofi) >= 10). For each sample pair (e.g. K562 replicates, HeLa vs K562, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO), compute the fraction of exons maintaining the same AISO class and the Pearson and Spearman correlations of Fupfi values, writing per-pair statistics to aiso_consistency_statistics.tsv.

## Related tools

- **Insplico** (Standalone software that quantifies AISO (upfi, dofi, bus, bos counts and Fupfi) from short- and long-read RNA-seq; includes an optional module correcting 5'/3' long-read biases) — gitlab.com/aghr/insplico
- **STAR** (2-pass splice-aware aligner producing BAMs feeding Insplico; run with --chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100)
- **R** (Environment for annotation import (rtracklayer), simulation of UPFI/DOFI transcripts, and downstream consistency statistics with dplyr and ggplot2)

## Examples

```
Rscript -e 'my_obj <- import("Hsa38_only_transcript_support_level_1.gtf")'  # then run Insplico on STAR-aligned BAMs with hg38 Ensembl v88 + VastDB exon tables
```

## Evaluation signals

- Cross-sample AISO maintenance fractions recapitulate the reported statistics, in particular 64% of mouse upfi exons conserved as upfi in human brain and 75% of dofi exons conserved (one-sided Binomial P = 3e-22)
- Per-exon Fupfi values are highly correlated (Pearson and Spearman) between replicates and across cell/tissue types
- Classified exons retain only N(upfi+dofi) >= 10 informative reads; upfi exons have Fupfi >= 0.8, dofi exons Fupfi <= 0.2
- On simulated reads with uniform UPFI/DOFI distribution, Insplico estimates recover the simulated Fupfi without long-read 5'/3' bias artifacts (after applying the correction module)
- Output file aiso_consistency_statistics.tsv contains one row per sample pair with maintenance fraction plus Pearson and Spearman correlation columns

## Limitations

- Read length and paired-end fragment size limit the maximum exon length quantifiable with short reads
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads
- Long-read sequencing and library preparation carry 5' and 3' biases that must be corrected to harmonize with short-read results
- No software version number is stated for Insplico in the article

## Evidence

- [abstract] Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies: "we present Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [discussion] upfi, dofi, bus and bos counts; Fupfi; PSI; PIR measurements around user-specified exons: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [discussion] Optional module correcting 5'/3' long-read biases: "The optional module to perform these corrections is a unique feature of Insplico"
- [methods] gtf_file<-'Hsa38_only_transcript_support_level_1.gtf'; my_obj <- import(gtf_file): "gtf_file<-"Hsa38_only_transcript_support_level_1.gtf""
- [readme] Transcript_Exon_Count.tab contains gene info and number of exons per transcript; random middle exon selected from transcripts with Nexo>2: "numexo_rand<-read.table("Transcript_Exon_Count.tab", col.names = c("Transcript","GeneName","GeneID","Nexo")) %>%
      as_tibble()  %>% filter(Nexo>2)"
- [supplementary] RNA-seq data deposited to GEO under GSE207459: "RNA seq data have been deposited to Gene Expression Omnibus (GEO) and are accessible under the ID GSE207459."

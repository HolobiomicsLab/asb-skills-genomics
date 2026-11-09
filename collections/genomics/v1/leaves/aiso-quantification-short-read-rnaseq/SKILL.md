---
name: aiso-quantification-short-read-rnaseq
description: Use when you have short-read nuclear/chromatin RNA-seq (e.g. FASTQ or
  BAM) plus genome FASTA and GTF annotation for a species, and need per-exon adjacent-intron
  splicing order counts (upfi/dofi, Fupfi) for feature association or cross-species
  comparison.
when_to_use_negative:
- You need splicing-order information beyond the introns directly flanking the input
  exon — short reads can barely provide this; use long-read AISO quantification with
  Insplico's long-read module instead
- The input exon is a very long exon whose length exceeds what paired-end fragment
  size/read length permits — the maximum quantifiable exon length with short reads
  is limited by read length and fragment size
- You want absolute transcript quantification (expression levels) rather than adjacent-intron
  splicing order measurements
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0204
tools:
- name: Insplico
  role: Standalone software that quantifies AISO from short-read RNA-seq, returning
    per-exon upfi/dofi counts and Fupfi
  repo: gitlab.com/aghr/insplico
- name: Matt
  role: Provides cmpr_features to extract the 55 intron-exon features (splice-site
    strengths, intron/exon lengths, branch point-to-AG distances) associated with
    the AISO measurements
- name: R (dplyr, tidyr, ggplot2, plyranges, rtracklayer, Biostrings, polyester)
  role: GTF import/exon selection, simulation validation with polyester, and feature-binned
    aggregation/plotting of mean Fupfi
  repo: https://github.com/liniguez/Insplico_simulations
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:9dbe967d33de33ce955ee145d8eb460486446c7035bbe1c02ceac7f53d922aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - Matt
  - R
  - R (dplyr, tidyr, ggplot2, plyranges, rtracklayer, Biostrings, polyester)
  - dplyr
  - ggplot2
  - tidyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# aiso-quantification-short-read-rnaseq

## Summary

Quantify the splicing order of adjacent introns (AISO) around user-specified exons from short-read RNA-seq using Insplico, yielding per-exon upfi/dofi counts and the fraction of transcripts with the upstream intron spliced first (Fupfi). This is the core measurement skill behind Figure 6 of the Insplico paper, which maps AISO to intron-exon genomic features across species.

## When to use

Use when you have short-read nuclear/chromatin RNA-seq (e.g. FASTQ or BAM) plus genome FASTA and GTF annotation for a species, and need per-exon adjacent-intron splicing order counts (upfi/dofi, Fupfi) for feature association or cross-species comparison.

## When NOT to use

- You need splicing-order information beyond the introns directly flanking the input exon — short reads can barely provide this; use long-read AISO quantification with Insplico's long-read module instead
- The input exon is a very long exon whose length exceeds what paired-end fragment size/read length permits — the maximum quantifiable exon length with short reads is limited by read length and fragment size
- You want absolute transcript quantification (expression levels) rather than adjacent-intron splicing order measurements

## Inputs

- Short-read nuclear or chromatin RNA-seq datasets (e.g. GEO GSE207459)
- Species-specific genome FASTA files (chromosome FASTA, chr1–chrY, decompressed to chroms/chr*.fa)
- GTF gene annotation, optionally filtered to transcript_support_level 1 (e.g. Hsa38_only_transcript_support_level_1.gtf)
- Transcript-exon count table (Transcript_Exon_Count.tab) with Transcript, GeneName, GeneID, Nexo columns

## Outputs

- Per-exon upfi, dofi, bus and bos counts around user-specified exons
- Fupfi — fraction of transcripts with the upstream intron spliced first (plus PSI and PIR estimates)
- Feature-binned mean Fupfi tables (five equal subsets per intron-exon feature) and ggplot2 plots for the 55 features

## How to apply

Prepare per-species annotation by renaming chromosome tokens in the GTF (e.g. prefixing 'chr' with the provided perl one-liner) and optionally filtering to transcript_support_level 1 exons, then download chromosome FASTA files (chr1–chrY) into a chroms/ folder. Import the GTF with rtracklayer's import() and select middle exons of transcripts with more than 2 exons (Nexo>2). Run Insplico on each short-read dataset with the reference genome and annotation to obtain per-exon upfi, dofi, bus and bos counts and Fupfi; note that short reads only quantify splicing order for the introns directly flanking the input exon. Downstream, extract intron-exon features (e.g. with Matt's cmpr_features) covering splice-site strengths, intron/exon lengths and branch point-to-AG distances, bin exons into five equal subsets per feature, and compute mean Fupfi per bin in R (dplyr/tidyr/ggplot2). AISO around individual exons is expected to be constant across cell and tissue types, so bulk short-read data are sufficient for stable estimates.

## Related tools

- **Insplico** (Standalone software that quantifies AISO from short-read RNA-seq, returning per-exon upfi/dofi counts and Fupfi) — gitlab.com/aghr/insplico
- **Matt** (Provides cmpr_features to extract the 55 intron-exon features (splice-site strengths, intron/exon lengths, branch point-to-AG distances) associated with the AISO measurements)
- **R (dplyr, tidyr, ggplot2, plyranges, rtracklayer, Biostrings, polyester)** (GTF import/exon selection, simulation validation with polyester, and feature-binned aggregation/plotting of mean Fupfi) — https://github.com/liniguez/Insplico_simulations

## Examples

```
numexo_rand<-read.table("Transcript_Exon_Count.tab", col.names = c("Transcript","GeneName","GeneID","Nexo")) %>% as_tibble() %>% filter(Nexo>2)
```

## Evaluation signals

- Insplico output contains upfi, dofi, bus and bos counts per exon and Fupfi estimates for every user-specified exon
- Validation against simulated reads with known UPFI/DOFI copy numbers (uniform distribution, Ncopies 10–20, 1000 genes per simulated sample, nsamples=20, seed=20221107) recovers the assigned splicing order
- Recapitulation of previously reported AISO patterns, e.g. mean Fupfi rising with upstream 5'/3' splice-site strength, falling with upstream intron length, rising with downstream intron length
- AISO around individual exons is consistent across cell and tissue types and conserved between human and mouse, so repeated estimates within a species should agree
- GTF preprocessing check: chromosome tokens prefixed with 'chr' and only transcript_support_level 1 entries retained for the high-confidence analysis

## Limitations

- The length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon
- Long-read AISO estimates carry 5' and 3' biases that must be corrected via Insplico's optional module to harmonize with short-read results
- Simulations used GRCh38 release-85 GTF; genome/annotation release differences may change exon selection

## Evidence

- [abstract] Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies: "Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [discussion] Per-exon measurements returned by Insplico: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [methods] Chromosome FASTA preparation step: "wget ${FAPATH}chr5.fna.gz && gunzip chr5.fna.gz && mv chr5.fna chroms/chr5.fa"
- [readme] Simulation strategy emulating UPFI/DOFI transcripts: "Random selected exons are substituted by new exons, which are defined by merging the up and down stream intron + exon of the exon of interest, emulating UPFI and DOFI"
- [readme] Exon selection rule for simulated exons: "Select one random middle exon from transcripts with more than 2 exons."

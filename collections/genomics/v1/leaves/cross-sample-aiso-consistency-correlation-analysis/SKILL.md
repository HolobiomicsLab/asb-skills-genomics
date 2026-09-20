---
name: cross-sample-aiso-consistency-correlation-analysis
description: Use when you have RNA-seq (short or long reads) for at least two comparable
  samples of the same species or orthologous tissues and need to determine whether
  each exon's AISO class (upfi vs dofi) and Fupfi value is stable across conditions.
when_to_use_negative:
- Your goal is differential gene or transcript expression rather than the relative
  order in which adjacent introns are spliced — Insplico's upfi/dofi/Fupfi metrics
  address splicing order, not abundance
- The samples are not comparable pairs of the same or orthologous tissues/conditions;
  AISO maintenance fractions are only meaningful between matched biological contexts
  such as brain vs brain across species
- 'Exons have too few informative reads: with N(upfi+dofi)<10 the upfi/dofi classification
  is not supported, and short reads provide almost no splicing-order information beyond
  the introns directly flanking the exon'
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Insplico
  role: 'Core quantification tool: computes upfi, dofi, bus and bos counts and estimates
    Fupfi around user-specified exons from short and long RNA-seq reads'
  repo: gitlab.com/aghr/insplico
- name: STAR
  role: 2-pass splice-aware aligner producing BAM alignments and splice junctions
    (with --chimSegmentMin 20, --alignSJoverhangMin 8) that Insplico consumes to quantify
    splicing order
- name: R
  role: 'Downstream analysis environment: dplyr for per-pair AISO consistency statistics
    and ggplot2 for visualizing maintenance fractions and Fupfi correlations'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:c241c03174b4012790bf8e8d6b9d02e74dd5c655e9a5d20d752a66c72cdf9b76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  - STAR
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cross-sample-aiso-consistency-correlation-analysis

## Summary

Quantify the splicing order of adjacent introns (AISO) around each exon genome-wide with Insplico, then measure how consistently upfi/dofi classes and Fupfi values are maintained across sample pairs (replicates, cell types, tissues, species, spliceosome-disrupted conditions) using maintenance fractions and Pearson/Spearman correlations.

## When to use

Use when you have RNA-seq (short or long reads) for at least two comparable samples of the same species or orthologous tissues and need to determine whether each exon's AISO class (upfi vs dofi) and Fupfi value is stable across conditions.

## When NOT to use

- Your goal is differential gene or transcript expression rather than the relative order in which adjacent introns are spliced — Insplico's upfi/dofi/Fupfi metrics address splicing order, not abundance
- The samples are not comparable pairs of the same or orthologous tissues/conditions; AISO maintenance fractions are only meaningful between matched biological contexts such as brain vs brain across species
- Exons have too few informative reads: with N(upfi+dofi)<10 the upfi/dofi classification is not supported, and short reads provide almost no splicing-order information beyond the introns directly flanking the exon

## Inputs

- Chromatin-associated RNA-seq FASTQ files for sample pairs (K562 replicates, HeLa, K562 vs HeLa, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO; GEO accession GSE207459)
- hg38 or mm10 reference genome for STAR alignment
- Ensembl v88 GTF annotation (e.g. Hsa38_only_transcript_support_level_1.gtf, transcript support level 1) and VastDB exon tables for Insplico
- List of user-specified exons of interest (or genome-wide exon table) around which to measure AISO

## Outputs

- Per-exon Insplico quantification table of upfi, dofi, bus and bos counts and Fupfi values for each sample
- Exon-level AISO class assignments (upfi when Fupfi>=0.8, dofi when Fupfi<=0.2) with N(upfi+dofi)>=10 read filter applied
- aiso_consistency_statistics.tsv with per-sample-pair AISO maintenance fractions and Pearson/Spearman correlations of Fupfi

## How to apply

Align reads to hg38 or mm10 with STAR in 2-pass mode using the article's parameters (--chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100). Run Insplico with hg38/mm10 Ensembl v88 annotations and VastDB exon tables to obtain upfi, dofi, bus and bos counts and Fupfi around user-specified exons genome-wide. Classify each exon as upfi when Fupfi>=0.8 and dofi when Fupfi<=0.2, retaining only exons with N(upfi+dofi)>=10 informative reads, since AISO quantification is unreliable at low coverage. For each sample pair (e.g. K562 replicates, HeLa vs K562, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO), compute the fraction of exons maintaining the same AISO class and the Pearson and Spearman correlations of Fupfi, writing per-pair statistics to aiso_consistency_statistics.tsv. The article's key validation signal is the cross-species comparison, where 64% of mouse upfi exons and 75% of dofi exons were conserved in human brain versus 41% expected by chance (P = 3e-22, one-sided binomial test), so a successful run should recapitulate maintenance fractions in this range rather than chance-level agreement.

## Related tools

- **Insplico** (Core quantification tool: computes upfi, dofi, bus and bos counts and estimates Fupfi around user-specified exons from short and long RNA-seq reads) — gitlab.com/aghr/insplico
- **STAR** (2-pass splice-aware aligner producing BAM alignments and splice junctions (with --chimSegmentMin 20, --alignSJoverhangMin 8) that Insplico consumes to quantify splicing order)
- **R** (Downstream analysis environment: dplyr for per-pair AISO consistency statistics and ggplot2 for visualizing maintenance fractions and Fupfi correlations)

## Examples

```
STAR --runThreadN 8 --runMode genomeGenerate --genomeDir hg38_reference --readFilesIn R1.fq.gz R2.fq.gz --readFilesCommand zcat --outSAMstrandField intronMotif --alignSJoverhangMin 8 --alignIntronMin 20 --alignIntronMax 200000 --chimSegmentMin 20 --outFilterMultimapNmax 100
```

## Evaluation signals

- Human vs mouse brain conservation should recapitulate the paper's reported fractions: 64% of upfi exons and 75% of dofi exons maintained the same AISO class, versus 41% expected by chance (P = 3e-22, one-sided binomial test)
- Every classified exon must satisfy the informative-read filter N(upfi+dofi)>=10 and fall into upfi (Fupfi>=0.8) or dofi (Fupfi<=0.2); exons in between remain unclassified
- Replicate pairs (e.g. K562 replicates) should show near-complete AISO maintenance and high Pearson/Spearman Fupfi correlations; large cross-sample drops signal alignment or annotation problems rather than biological variability
- aiso_consistency_statistics.tsv must contain one row per sample pair with both a maintenance fraction and both correlation coefficients (Pearson and Spearman)
- Cross-condition pairs (SF3B1-drug vs DMSO) should still show high maintenance fractions, since the article reports AISO is constant even upon major spliceosomal disruption

## Limitations

- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads
- The maximum length of exons quantifiable by Insplico with short reads is limited by read length and the fragment size of paired-end reads
- Long read sequencing and library preparation carry 5' and 3' biases that must be corrected with Insplico's optional correction module to harmonize patterns with short-read results
- Quantification requires sufficient coverage: exons without at least 10 informative upfi+dofi reads cannot be reliably classified

## Evidence

- [abstract] Insplico quantifies AISO from short and long reads: "we present Insplico, the first standalone software for quantifying AISO that works with both short and long read sequencing technologies"
- [abstract] AISO constancy and conservation finding: "AISO around individual exons is remarkably constant across cell and tissue types and even upon major spliceosomal disruption, and it is evolutionarily conserved between human and mouse brains"
- [discussion] Insplico output metrics: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [methods] STAR alignment parameters used in the workflow: "--readFilesCommand zcat \
 --outSAMstrandField intronMotif --genomeDir hg38_reference"
- [readme] Exon filtering logic from the simulation/analysis scripts: "# Select one random middle exon from transcripts with more than 2 exons."

---
name: cross-species-aiso-conservation-assessment
description: Use when you have RNA-seq FASTQ pairs from replicate, cell-type, tissue,
  species, or spliceosomal-disruption contrasts (e.g. K562/HeLa, mouse brain vs liver,
  human vs mouse brain, SF3B1-drug vs DMSO, GSE207459) and need to test whether AISO
  patterns around exons remain stable across the pairs.
when_to_use_negative:
- You are studying alternative splicing levels (PSI/PIR) rather than splicing ORDER
  of adjacent introns — Insplico's upfi/dofi quantification addresses AISO, not isoform
  abundance.
- 'The exons of interest are microexons or very long exons: read length and paired-end
  fragment size limit the maximum quantifiable exon length with short reads, and long-read
  biases require Insplico''s 5''/3'' correction module.'
- Only a single sample is available with no comparison pair — AISO consistency cannot
  be assessed without at least two samples/conditions to compare.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: Insplico
  role: Standalone software to quantify AISO (upfi, dofi, bus, bos counts and Fupfi)
    from short-read RNA-seq around user-specified exons; used to classify each exon's
    splicing order per sample
  repo: gitlab.com/aghr/insplico
- name: STAR
  role: 2-pass splice-aware aligner mapping RNA-seq reads to hg38/mm10 with junction-sensitive
    parameters (--chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20,
    --alignIntronMax 200000, --outFilterMultimapNmax 100) prior to AISO quantification
- name: R
  role: Environment for computing per-pair consistency statistics (maintenance fractions,
    Pearson/Spearman correlations, one-sided Binomial test) and writing aiso_consistency_statistics.tsv
- name: dplyr
  role: R package for filtering exons by Fupfi/informative-read thresholds and summarising
    per-pair AISO statistics
- name: ggplot2
  role: R package for visualising Fupfi correlations and AISO class-maintenance fractions
    across sample pairs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:f04345a93b86c325c51be29fae3ecede7fd1147efda9283b3ec4c2be1ed167b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - R
  - STAR
  - dplyr
  - ggplot2
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-species-aiso-conservation-assessment

## Summary

Quantify whether adjacent introns' splicing order (AISO) around individual exons is maintained across samples, cell/tissue types, and species (e.g. human vs mouse brain), by running Insplico on STAR-aligned RNA-seq pairs, classifying exons into upfi/dofi classes, and measuring class-maintenance fractions and Fupfi correlations.

## When to use

Use when you have RNA-seq FASTQ pairs from replicate, cell-type, tissue, species, or spliceosomal-disruption contrasts (e.g. K562/HeLa, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO, GSE207459) and need to test whether AISO patterns around exons remain stable across the pairs.

## When NOT to use

- You are studying alternative splicing levels (PSI/PIR) rather than splicing ORDER of adjacent introns — Insplico's upfi/dofi quantification addresses AISO, not isoform abundance.
- The exons of interest are microexons or very long exons: read length and paired-end fragment size limit the maximum quantifiable exon length with short reads, and long-read biases require Insplico's 5'/3' correction module.
- Only a single sample is available with no comparison pair — AISO consistency cannot be assessed without at least two samples/conditions to compare.

## Inputs

- RNA-seq FASTQ files for sample pairs (K562 replicates, HeLa, K562 vs HeLa, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO; GEO accession GSE207459)
- hg38 and mm10 genome references for STAR alignment
- hg38/mm10 Ensembl v88 GTF annotation and VastDB exon tables for Insplico
- User-specified exon coordinates genome-wide for Fupfi estimation

## Outputs

- aiso_consistency_statistics.tsv with per-pair AISO class-maintenance fractions and Pearson/Spearman Fupfi correlations
- Per-exon upfi/dofi classifications with Fupfi values and informative-read counts per sample

## How to apply

Retrieve the chromatin-associated RNA-seq datasets for the sample pairs and align reads to hg38/mm10 with STAR in 2-pass mode using --chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000 and --outFilterMultimapNmax 100. Run Insplico with hg38/mm10 Ensembl v88 and VastDB exon tables to obtain upfi, dofi, bus and bos counts and estimate Fupfi around user-specified exons genome-wide. Classify each exon as upfi when Fupfi >= 0.8 and dofi when Fupfi <= 0.2, retaining only exons supported by at least 10 informative reads (N(upfi+dofi) >= 10), a stringency needed because low-count exons give unstable splicing-order estimates. For each sample pair (replicates, K562 vs HeLa, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO), compute the fraction of exons that maintain the same AISO class and the Pearson and Spearman correlations of Fupfi values, then compare against the paper's reported statistics — notably 64% of mouse upfi exons also upfi in human brain (41% expected by chance, one-sided Binomial P = 3e-22) and 75% dofi conservation. Write all per-pair statistics to aiso_consistency_statistics.tsv.

## Related tools

- **Insplico** (Standalone software to quantify AISO (upfi, dofi, bus, bos counts and Fupfi) from short-read RNA-seq around user-specified exons; used to classify each exon's splicing order per sample) — gitlab.com/aghr/insplico
- **STAR** (2-pass splice-aware aligner mapping RNA-seq reads to hg38/mm10 with junction-sensitive parameters (--chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100) prior to AISO quantification)
- **R** (Environment for computing per-pair consistency statistics (maintenance fractions, Pearson/Spearman correlations, one-sided Binomial test) and writing aiso_consistency_statistics.tsv)
- **dplyr** (R package for filtering exons by Fupfi/informative-read thresholds and summarising per-pair AISO statistics)
- **ggplot2** (R package for visualising Fupfi correlations and AISO class-maintenance fractions across sample pairs)

## Examples

```
STAR --runThreadN 8 --genomeDir hg38_reference --readFilesIn sample_R1.fq.gz sample_R2.fq.gz --readFilesCommand zcat --outSAMstrandField intronMotif --twopassMode Basic --chimSegmentMin 20 --alignSJoverhangMin 8 --alignIntronMin 20 --alignIntronMax 200000 --outFilterMultimapNmax 100; then Insplico on the BAMs and compute per-pair Fupfi maintenance fractions and Pearson/Spearman correlations into aiso_consistency_statistics.tsv.
```

## Evaluation signals

- Cross-sample AISO maintenance fractions recapitulate the paper's reported consistency statistics, in particular 64% upfi and 75% dofi conservation between human and mouse brains.
- Human-mouse upfi conservation significantly exceeds the 41% chance expectation with a one-sided Binomial test P ≈ 3e-22.
- Replicate pairs (e.g. K562 replicates) show high maintenance fractions and strong Pearson/Spearman Fupfi correlations, as AISO is 'remarkably constant across cell and tissue types'.
- Every retained exon passes the filter Fupfi >= 0.8 (upfi) or Fupfi <= 0.2 (dofi) with N(upfi+dofi) >= 10 informative reads.
- All per-pair statistics are present in aiso_consistency_statistics.tsv with matching row counts per sample pair.

## Limitations

- The length of the reads (and paired-end fragment size) limits the maximum length of exons quantifiable by Insplico with short reads.
- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads.
- Long read sequencing and library preparation carry 5' and 3' biases; Insplico's optional correction module must be applied to harmonize with short-read results.
- Whether AISO regulation varies across tissues/conditions was itself an open question in this work; strong conclusions about regulatory shifts require the conservation/constancy framing used here, not causal claims.

## Evidence

- [abstract] AISO around individual exons is constant across cell and tissue types and upon major spliceosomal disruption, and is evolutionarily conserved between human and mouse brains: [evidence span withheld — anchor status: not_found]
- [other] Classify each exon as upfi when Fupfi>=0.8, dofi when Fupfi<=0.2, retaining only exons supported by N(upfi+dofi)>=10 informative reads.: [evidence span withheld — anchor status: not_found]
- [other] 64% of mouse upfi exons were also upfi in human (versus 41% expected by chance; P = 3e-22, one-sided Binomial test): [evidence span withheld — anchor status: not_found]
- [discussion] the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
- [discussion] could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns: "could be due to 5′ and 3′ biases in long read sequencing and library preparation that, when corrected for, harmonize the patterns"

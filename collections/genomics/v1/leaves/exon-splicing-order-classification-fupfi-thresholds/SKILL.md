---
name: exon-splicing-order-classification-fupfi-thresholds
description: Use when you have aligned RNA-seq (short or long reads) around user-specified
  exons and need to label each exon's AISO class (upfi/dofi) or quantify Fupfi for
  cross-sample consistency, conservation, or spliceosomal-disruption analyses.
when_to_use_negative:
- The goal is differential transcript-level splicing (PSI/PIR of alternative exons)
  rather than the relative splicing order of the two introns flanking a single exon
  — Insplico targets AISO specifically.
- Exons lack a clear adjacent upstream/downstream intron pair (first/last exons, single-exon
  transcripts) or the flanking introns exceed quantifiable limits, since informative
  upfi/dofi reads cannot be defined.
- Only very short reads or short paired-end fragments are available for long exons
  — read length and fragment size limit the maximum exon length quantifiable with
  short reads.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_3320
tools:
- name: Insplico
  role: Standalone software that quantifies upfi/dofi/bus/bos counts and estimates
    Fupfi around user-specified exons from short- or long-read RNA-seq, enabling AISO
    classification
  repo: gitlab.com/aghr/insplico
- name: STAR
  role: 2-pass splice-aware aligner producing the BAM/SJ inputs Insplico consumes,
    with parameters tuned for splicing-order detection (--chimSegmentMin 20, --alignSJoverhangMin
    8)
- name: R
  role: Environment for classification scripting, cross-sample correlation, and binomial
    tests on AISO maintenance fractions
- name: dplyr
  role: R data wrangling of per-exon count tables and filtering to informative-read
    thresholds
- name: ggplot2
  role: Visualization of Fupfi distributions and cross-sample AISO consistency
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:bd3ecb2027f4965f9dca6fe4e1a2b96178a1f591c4a6c31f41987f578c9f02b9
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
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# exon-splicing-order-classification-fupfi-thresholds

## Summary

Classifies individual exons into adjacent-intron splicing-order (AISO) categories — upfi (upstream intron first), dofi (downstream intron first), or unclassified — using Insplico-quantified upfi/dofi/bus/bos read counts and the fraction of upstream-first informative reads (Fupfi) with fixed thresholds.

## When to use

Use when you have aligned RNA-seq (short or long reads) around user-specified exons and need to label each exon's AISO class (upfi/dofi) or quantify Fupfi for cross-sample consistency, conservation, or spliceosomal-disruption analyses.

## When NOT to use

- The goal is differential transcript-level splicing (PSI/PIR of alternative exons) rather than the relative splicing order of the two introns flanking a single exon — Insplico targets AISO specifically.
- Exons lack a clear adjacent upstream/downstream intron pair (first/last exons, single-exon transcripts) or the flanking introns exceed quantifiable limits, since informative upfi/dofi reads cannot be defined.
- Only very short reads or short paired-end fragments are available for long exons — read length and fragment size limit the maximum exon length quantifiable with short reads.

## Inputs

- BAM alignments of chromatin-associated RNA-seq from GEO accession GSE207459 sample pairs (K562 replicates, HeLa, K562 vs HeLa, mouse brain vs liver, human vs mouse brain, SF3B1-drug vs DMSO), aligned to hg38/mm10
- hg38/mm10 Ensembl v88 annotation GTF (e.g. Hsa38_only_transcript_support_level_1.gtf) and VastDB exon tables
- User-specified list of exons of interest (genomic coordinates with flanking adjacent introns)

## Outputs

- Per-exon upfi, dofi, bus and bos informative read counts and Fupfi estimates
- Per-exon AISO class labels (upfi at Fupfi>=0.8; dofi at Fupfi<=0.2; unclassified otherwise), filtered to N(upfi+dofi)>=10 informative reads
- Per-pair cross-sample consistency statistics (AISO class maintenance fractions and Pearson/Spearman correlations of Fupfi), e.g. written to aiso_consistency_statistics.tsv

## How to apply

Run Insplico (gitlab.com/aghr/insplico) on reads aligned with STAR 2-pass mode (--chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100) against hg38 or mm10 with Ensembl v88 / VastDB exon tables, so it quantifies upfi, dofi, bus and bos counts and estimates Fupfi around each user-specified exon. Classify an exon as upfi when Fupfi >= 0.8 and as dofi when Fupfi <= 0.2, retaining only exons supported by at least N(upfi+dofi) >= 10 informative reads; exons in the 0.2–0.8 Fupfi band or below the read-depth floor stay unclassified. Once classes are assigned, compare across sample pairs (replicates, cell types, tissues, species, SF3B1-drug vs DMSO) by computing the fraction of exons that maintain the same class and the Pearson/Spearman correlation of Fupfi values. This threshold scheme is what underpins the paper's validation targets such as 64%/75% AISO conservation between human and mouse brains (P = 3e-22, one-sided binomial test versus 41% expected by chance).

## Related tools

- **Insplico** (Standalone software that quantifies upfi/dofi/bus/bos counts and estimates Fupfi around user-specified exons from short- or long-read RNA-seq, enabling AISO classification) — gitlab.com/aghr/insplico
- **STAR** (2-pass splice-aware aligner producing the BAM/SJ inputs Insplico consumes, with parameters tuned for splicing-order detection (--chimSegmentMin 20, --alignSJoverhangMin 8))
- **R** (Environment for classification scripting, cross-sample correlation, and binomial tests on AISO maintenance fractions)
- **dplyr** (R data wrangling of per-exon count tables and filtering to informative-read thresholds)
- **ggplot2** (Visualization of Fupfi distributions and cross-sample AISO consistency)

## Examples

```
STAR --runThreadN 8 --genomeDir hg38_reference --readFilesIn R1.fq.gz R2.fq.gz --readFilesCommand zcat --twopassMode Basic --chimSegmentMin 20 --alignSJoverhangMin 8 --alignIntronMin 20 --alignIntronMax 200000 --outFilterMultimapNmax 100 --outSAMstrandField intronMotif
```

## Evaluation signals

- Upfi/dofi class assignments reproduce only within the specified Fupfi bands: exons with Fupfi>=0.8 are upfi and Fupfi<=0.2 are dofi, with no class assigned in the 0.2–0.8 interval.
- Every classified exon passes the informative-read floor N(upfi+dofi)>=10; check no exon below this depth appears in the output.
- Cross-sample AISO maintenance fractions and Pearson/Spearman Fupfi correlations between replicates, tissues, and human vs mouse brains recapitulate the paper's reported values (e.g. 64%/75% human-mouse brain conservation; P = 3e-22 one-sided binomial test).
- Fupfi distributions are stable across cell/tissue types and upon SF3B1 spliceosomal disruption, consistent with the finding that AISO is remarkably constant across conditions.
- All per-pair statistics are written to aiso_consistency_statistics.tsv with matching pair identifiers and non-empty correlation values.

## Limitations

- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, unlike long reads.
- Read length and paired-end fragment size limit the maximum exon length that can be quantified with short reads.
- Long-read sequencing and library preparation carry 5' and 3' biases that must be corrected (Insplico's optional correction module) to harmonize with short-read results.
- No Insplico software version number is stated in the article, which may hamper exact reproducibility.

## Evidence

- [other] Classify each exon as upfi when Fupfi>=0.8, dofi when Fupfi<=0.2, retaining only exons supported by N(upfi+dofi)>=10 informative reads.: [evidence span withheld — anchor status: not_found]
- [abstract] AISO around individual exons is remarkably constant across cell and tissue types and even upon major spliceosomal disruption, and it is evolutionarily conserved between human and mouse brains: "AISO around individual exons is remarkably constant across cell and tissue types and even upon major spliceosomal disruption, and it is evolutionarily conserved between human and mouse brains"
- [discussion] It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts): "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts)"
- [discussion] the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads: "the length of the reads (and the fragment size of paired-end reads) limits the maximum length of the exons that are quantifiable by Insplico using short reads"
- [discussion] short reads can barely provide splicing order information beyond them as long reads can do: "short reads can barely provide splicing order information beyond them as long reads can do"
- [readme] numexo_rand <- read.table("Transcript_Exon_Count.tab", ...) %>% as_tibble() %>% filter(Nexo>2): "numexo_rand<-read.table("Transcript_Exon_Count.tab", col.names = c("Transcript","GeneName","GeneID","Nexo")) %>%
      as_tibble()  %>% filter(Nexo>2)"

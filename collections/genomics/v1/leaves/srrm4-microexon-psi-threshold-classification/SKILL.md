---
name: srrm4-microexon-psi-threshold-classification
description: Use when short-read RNA-seq from SRRM4 perturbation (e.g. GSE207459 HEK293
  GFP vs 3xFlag-SRRM4) must be classified into SRRM4-dependent microexons and compared
  for downstream-intron-first vs upstream-intron-first AISO fractions.
when_to_use_negative:
- The analysis targets genome-wide AISO patterns across tissues or species rather
  than classification of microexons around a single trans-acting factor perturbation
- Only long-read RNA-seq is available without correction for 5'/3' biases, which Insplico's
  correction module must harmonize with short-read results
- The exon of interest exceeds the length quantifiable by short reads, which is limited
  by read length and paired-end fragment size
edam_operation: http://edamontology.org/operation_0260
edam_topics:
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: Insplico
  role: Quantifies AISO metrics (upfi, dofi, bus, bos counts; Fupfi, PSI, PIR) around
    user-specified microexons from STAR-aligned short-read RNA-seq
  repo: https://gitlab.com/aghr/insplico
- name: STAR
  role: Aligns HEK293 GFP and 3xFlag-SRRM4 RNA-seq reads to hg38 with splice-junction-sensitive
    parameters (--chimSegmentMin 20, --alignSJoverhangMin 8) feeding Insplico
- name: dplyr
  role: R data manipulation for classifying microexons by DeltaPSI and Fupfi thresholds
    and summarizing dofi/upfi fractions
- name: plyranges
  role: Genomic-range filtering of exon annotations during control-microexon assembly
- name: Matt
  role: Extracts splicing-related features (exon length, 3' splice-site strength,
    UGC RNA map) for comparison between upfi and dofi microexon classes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkad244
    title: '<i>Insplico</i> : effective computational tool for studying splicing order
      of adjacent introns genome-wide with short and long RNA-seq reads'
  source_content_hash: sha256:0af2cd3b3bd1016323b833f6f6668cb1c9d108e48b42351a60a7e65adf16de15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Insplico
  - Matt
  - R
  - STAR
  - dplyr
  - plyranges
  edam_operation: http://edamontology.org/operation_0260
  edam_topics:
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# srrm4-microexon-psi-threshold-classification

## Summary

Classifies microexons as SRRM4-dependent versus matched controls and stratifies them by adjacent-intron splicing order (upfi/dofi) using PSI/PIR and upfi/dofi/bus/bos counts estimated with Insplico from STAR-aligned HEK293 GFP vs 3xFlag-SRRM4 RNA-seq (GSE207459).

## When to use

Use when short-read RNA-seq from SRRM4 perturbation (e.g. GSE207459 HEK293 GFP vs 3xFlag-SRRM4) must be classified into SRRM4-dependent microexons and compared for downstream-intron-first vs upstream-intron-first AISO fractions.

## When NOT to use

- The analysis targets genome-wide AISO patterns across tissues or species rather than classification of microexons around a single trans-acting factor perturbation
- Only long-read RNA-seq is available without correction for 5'/3' biases, which Insplico's correction module must harmonize with short-read results
- The exon of interest exceeds the length quantifiable by short reads, which is limited by read length and paired-end fragment size

## Inputs

- HEK293 GFP and 3xFlag-SRRM4 total RNA-seq FASTQ files (GEO accession GSE207459)
- hg38 genome reference for STAR alignment
- Ensembl v88 + VastDB exon table for Insplico
- List of neural microexons of interest with genomic coordinates

## Outputs

- Insplico per-exon table of upfi, dofi, bus, bos counts with Fupfi, PSI and PIR estimates
- Binary SRRM4-dependent / control classification per microexon (DeltaPSI > 0.15, control PSI <= 0.2)
- upfi (Fupfi >= 0.8) and dofi (Fupfi <= 0.2) class assignments per microexon
- Fisher's exact test statistics for dofi-vs-upfi enrichment
- Feature-comparison summaries (exon length, 3' splice-site strength, UGC RNA map) written to srrm4_microexon_aiso_results.tsv

## How to apply

Download HEK293 GFP and 3xFlag-SRRM4 total RNA-seq from GSE207459 and align with STAR (--chimSegmentMin 20, --alignSJoverhangMin 8, --alignIntronMin 20, --alignIntronMax 200000, --outFilterMultimapNmax 100, --quantMode GeneCounts). Run Insplico with the Ensembl v88+VastDB exon table to obtain upfi, dofi, bus and bos counts plus Fupfi, PSI and PIR around each microexon. Classify a microexon as SRRM4-dependent when DeltaPSI > 0.15 between 3xFlag-SRRM4 and GFP with control PSI <= 0.2, and assemble matched control microexons. Bin classes by Fupfi (upfi if Fupfi >= 0.8, strongly dofi if Fupfi <= 0.2), compute dofi-vs-upfi fractions, and test enrichment with Fisher's exact test (expected 58% dofi vs 26% upfi, P = 0.013). Extract splicing features (exon length, 3' splice-site strength, UGC RNA map) with Matt for the upfi and dofi subsets and compare distributions between classes.

## Related tools

- **Insplico** (Quantifies AISO metrics (upfi, dofi, bus, bos counts; Fupfi, PSI, PIR) around user-specified microexons from STAR-aligned short-read RNA-seq) — https://gitlab.com/aghr/insplico
- **STAR** (Aligns HEK293 GFP and 3xFlag-SRRM4 RNA-seq reads to hg38 with splice-junction-sensitive parameters (--chimSegmentMin 20, --alignSJoverhangMin 8) feeding Insplico)
- **dplyr** (R data manipulation for classifying microexons by DeltaPSI and Fupfi thresholds and summarizing dofi/upfi fractions)
- **plyranges** (Genomic-range filtering of exon annotations during control-microexon assembly)
- **Matt** (Extracts splicing-related features (exon length, 3' splice-site strength, UGC RNA map) for comparison between upfi and dofi microexon classes)

## Examples

```
STAR --genomeDir hg38_reference --readFilesIn SRRM4_R1.fq.gz SRRM4_R2.fq.gz --readFilesCommand zcat --chimSegmentMin 20 --alignSJoverhangMin 8 --alignIntronMin 20 --alignIntronMax 200000 --outFilterMultimapNmax 100 --outSAMtype BAM SortedByCoordinate --quantMode GeneCounts
```

## Evaluation signals

- SRRM4-dependent set is reproducible under the dual threshold: DeltaPSI > 0.15 over GFP AND control PSI <= 0.2
- Class fractions land near published proportions: ~58% of SRRM4-dependent microexons strongly dofi (Fupfi <= 0.2) and ~26% upfi (Fupfi >= 0.8)
- Fisher's exact test for dofi-vs-upfi enrichment in SRRM4-dependent vs control microexons yields a significant result (published P = 0.013)
- Insplico outputs contain all four count columns (upfi, dofi, bus, bos) plus Fupfi, PSI and PIR for every microexon of interest
- Control microexons show roughly equal upfi and dofi percentages, in contrast to the SRRM4-dependent set

## Limitations

- Short reads provide almost no splicing-order information beyond the introns directly flanking the input exon, and read/fragment length limits the maximum quantifiable exon length
- Proposed mechanisms of SRRM4-dependent microexon splicing remain untested hypotheses
- Software versions for the tools in this pipeline are not stated in the article

## Evidence

- [results] In SRRM4-overexpressing cells, the majority of SRRM4-dependent microexons (58%) show strongly dofi splicing (Fupfi ≤ 0.2) and only 26% are upfi (Fupfi ≥ 0.8): [evidence span withheld — anchor status: not_found]
- [discussion] Insplico AISO measurements: "It obtains key measurements to investigate pre-mRNA processing around user-specified exons of interest (upfi, dofi, bus and bos counts) and estimates the fraction of transcripts"
- [discussion] SRRM4 mechanisms untested: "Although those hypotheses remain to be formally tested, our AISO analysis thus suggests two possible mechanisms of splicing of SRRM4-dependent neural microexons"
- [discussion] Short-read limits: "short reads can barely provide splicing order information beyond them as long reads can do"
- [supplementary] GEO accession: "RNA seq data have been deposited to Gene Expression Omnibus (GEO) and are accessible under the ID GSE207459."

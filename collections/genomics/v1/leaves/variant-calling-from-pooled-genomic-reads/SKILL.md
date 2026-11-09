---
name: variant-calling-from-pooled-genomic-reads
description: Use when you have raw pooled whole-genome sequencing reads, a reference-free
  set of predicted exon sequences (e.g. from EXFI's splice graph), and need exonic
  SNPs/indels suitable for downstream genotyping.
when_to_use_negative:
- A reference genome is available and reliable — a splice-aware aligner such as GMAP
  plus standard reference-based variant calling would be the more direct route
- You need exact genomic coordinates of variants; reads are mapped here to predicted
  exon sequences, not to a reference genome, so positions are relative to predicted
  exons
- The goal is exon/splice-graph prediction itself — use the EXFI build_baited_bloom_filter/build_splice_graph
  steps instead; this skill covers only the downstream read mapping and SNP calling
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0769
tools:
- name: EXFI
  role: Predicts the splice graph and exon sequences (GFA1) from a transcriptome assembly
    and raw WGS reads; gfa1_to_fasta extracts exon FASTA used as mapping targets
  repo: https://github.com/jlanga/exfi
- name: Bowtie2
  role: Maps the raw pooled genomic reads to the predicted exon sequences before variant
    calling
- name: BCFTools
  role: Performs SNP/indel calling on the reads mapped to predicted exons
- name: biobloomcategorizer
  role: Pre-filters the WGS reads, discarding reads not sharing at least one k-mer
    with the transcriptome, which improves downstream prediction metrics
  repo: https://github.com/bcgsc/biobloom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:f4384190fa19ae8a96807af62478b1b5c0cf5f19703f9069f67b580193db00c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BCFTools
  - Bowtie2
  - EXFI
  - biobloomcategorizer
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Variant calling from pooled genomic reads

## Summary

Discovery and genotyping of exonic SNPs/indels by mapping raw pooled-sequence (Pool-Seq) genomic reads onto predicted exon sequences (e.g. EXFI output) and calling variants with BCFTools, applying quality and proximity filters to retain genotypable variants.

## When to use

Use when you have raw pooled whole-genome sequencing reads, a reference-free set of predicted exon sequences (e.g. from EXFI's splice graph), and need exonic SNPs/indels suitable for downstream genotyping.

## When NOT to use

- A reference genome is available and reliable — a splice-aware aligner such as GMAP plus standard reference-based variant calling would be the more direct route
- You need exact genomic coordinates of variants; reads are mapped here to predicted exon sequences, not to a reference genome, so positions are relative to predicted exons
- The goal is exon/splice-graph prediction itself — use the EXFI build_baited_bloom_filter/build_splice_graph steps instead; this skill covers only the downstream read mapping and SNP calling

## Inputs

- Predicted exon sequences in FASTA format (e.g. extracted from an EXFI splice graph in GFA1 format with gfa1_to_fasta)
- Raw pooled whole-genome sequencing (WGS) reads (e.g. Pool-Seq FASTQ from two pools of five diploid individuals at 52x coverage)
- Optional: a reference set of previously validated SNPs (e.g. 96 transcriptomic SNPs from Kumar et al., 2019) for recovery-rate benchmarking

## Outputs

- Variant calls (SNPs and indels) from BCFTools in VCF format
- Filtered set of genotypable SNPs/indels (e.g. 228,931 SNPs and 26,169 indels after quality and distance filtering in the tench analysis)
- Recovery statistics versus a reference SNP set (e.g. 92/96 known SNPs genotyped)

## How to apply

First extract the predicted exon sequences in FASTA format from the EXFI splice graph (GFA1) with gfa1_to_fasta, then map the raw pooled genomic reads (e.g. two pools of five diploid tench individuals at 52x overall coverage) to these exons with Bowtie2. Call variants on the mapped reads with BCFTools. Filter out variants with a quality value below 20, then exclude variants located within 35 bp of another variant or of a predicted exon boundary to derive genotypable regions; this proximity filter is needed because variants too close to edges cannot be reliably genotyped. Optionally validate against a reference SNP set by tabulating how many known positions are recovered — in the tench retrospective analysis, 92 of 96 previously validated transcriptomic SNPs were successfully genotyped (100% precision, 99% recall).

## Related tools

- **EXFI** (Predicts the splice graph and exon sequences (GFA1) from a transcriptome assembly and raw WGS reads; gfa1_to_fasta extracts exon FASTA used as mapping targets) — https://github.com/jlanga/exfi
- **Bowtie2** (Maps the raw pooled genomic reads to the predicted exon sequences before variant calling)
- **BCFTools** (Performs SNP/indel calling on the reads mapped to predicted exons)
- **biobloomcategorizer** (Pre-filters the WGS reads, discarding reads not sharing at least one k-mer with the transcriptome, which improves downstream prediction metrics) — https://github.com/bcgsc/biobloom

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- After quality (<20) and 35 bp proximity filtering, the count of surviving SNPs/indels is in a plausible range (tench benchmark: 228,931 SNPs and 26,169 indels suitable for genotyping)
- Against a validated reference SNP set, recovery approaches the published benchmark: 92/96 tench SNPs genotyped (100% precision, 99% recall over the IEB-proximal set)
- All IEBs proximal to known exonic SNPs are detected — EXFI gave 100% precision over the 96-SNP tench reference set
- Read filtering with biobloomcategorizer reduces the Bloom filter false-positive rate by 68%-75% while improving classification metrics
- Filtered variants fall outside 35 bp windows around other variants and around predicted exon boundaries (i.e. inside genotypable regions)

## Limitations

- Read filtering and exon prediction rules strongly affect results; special care is required regarding input transcriptome quality and tissue breadth (single-tissue transcriptomes lose lowly expressed transcripts and other tissues' transcripts)
- Mappability scores are more optimistic than exon precision because they cover only interior exon sections, not full exon sequences — predicted SNPs may sit in imprecise exons
- Mappability of all methods falls when the input transcriptome is de novo-assembled (e.g. the Atlantic herring case)
- Filtering out variants within 35 bp of another variant or exon boundary discards otherwise callable positions; proximity-driven losses are inherent to the genotypable-region definition

## Evidence

- [methods] variant quality filter (quality value below 20 filtered out): "variants with a quality value below 20 were filtered out"
- [methods] proximity filter (variants within 35 bp of another variant or exon boundary): "and then those that were within 35 bp to another variant or a predicted exon boundary"
- [results] Quality and distance filtering of SNP/indel predictions: "after quality and distance filtering, 228,931 SNPs and 26,169 indels were predicted suitable for genotyping"
- [discussion] Read filtering step explains positive results: "These positive results for EXFI are due to the read filtering step and the exon prediction rules used."
- [results] Read filtering reduces BF FPR and improves metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [discussion] Mappability caveat in discussion: "This high mappability result, although more optimistic than the exon precision, means that even if the exon prediction is not precise, it is still usable for downstream analysis."

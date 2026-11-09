---
name: pooled-genomic-read-mapping-to-predicted-exons
description: Use when pooled WGS reads must be mapped to exon sequences predicted
  from a de novo transcriptome (no reference genome) to call genotypable exonic SNPs,
  e.g. retrospective validation against known SNPs.
when_to_use_negative:
- A reference genome and annotation exist and a splice-aware aligner such as GMAP
  can extract IEB coordinates directly — the EXFI route is for organisms where genomic
  references are not available or not reliable
- The input transcriptome is of poor quality or assembled from a single tissue, so
  lowly expressed transcripts are fragmented or missing (the herring case where EXFI
  was less accurate)
- SNPs of interest are non-exonic, since predicted exons cover only transcriptome-overlapping
  genomic k-mers
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: EXFI
  role: Predicts the splice graph and exon sequences from the transcriptome and WGS
    reads; output exons (FASTA via gfa1_to_fasta) are the mapping targets
  repo: https://github.com/jlanga/exfi
- name: abyss-bloom
  role: Builds the cascading Bloom filter used to filter raw genomic reads down to
    error-free transcriptome-overlapping k-mers
  repo: https://github.com/bcgsc/abyss/
- name: biobloomcategorizer
  role: Discards each WGS read that does not share at least one k-mer with the transcriptome,
    reducing the BF FPR by 68%–75%
  repo: https://github.com/bcgsc/biobloom
- name: Bowtie2
  role: Maps the pooled raw genomic reads to the predicted exon sequences before SNP
    calling
- name: BCFTools
  role: Calls SNPs from the reads mapped to predicted exons
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:c5f3f7ae6fcd2837a45b3d9bb4b555d08a3794712613ac3255444c880ad65f2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BCFTools
  - Bowtie2
  - EXFI
  - abyss-bloom
  - biobloomcategorizer
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# Pooled genomic read mapping to predicted exons

## Summary

Maps pooled raw whole-genome sequencing reads onto exon sequences predicted by EXFI (from a de novo transcriptome) with Bowtie2, then calls variants with BCFTools to recover genotypable exonic SNPs when no reference genome is available.

## When to use

Use when pooled WGS reads must be mapped to exon sequences predicted from a de novo transcriptome (no reference genome) to call genotypable exonic SNPs, e.g. retrospective validation against known SNPs.

## When NOT to use

- A reference genome and annotation exist and a splice-aware aligner such as GMAP can extract IEB coordinates directly — the EXFI route is for organisms where genomic references are not available or not reliable
- The input transcriptome is of poor quality or assembled from a single tissue, so lowly expressed transcripts are fragmented or missing (the herring case where EXFI was less accurate)
- SNPs of interest are non-exonic, since predicted exons cover only transcriptome-overlapping genomic k-mers

## Inputs

- Raw pooled whole-genome sequencing reads (FASTQ; e.g. two pools of five diploid individuals at 52x overall coverage)
- De novo assembled transcriptome (FASTA)
- Reference set of previously validated transcriptomic SNP positions (e.g. 96 SNPs from Kumar et al., 2019)

## Outputs

- Predicted exon sequences in FASTA format (via gfa1_to_fasta)
- Splice graph in GFA1 format
- Mapped reads (BAM) from Bowtie2 against predicted exons
- Filtered genotypable SNP calls (BCFTools, quality >= 20, >= 35 bp from other variants and exon boundaries)

## How to apply

First run EXFI (build_baited_bloom_filter followed by build_splice_graph) on the assembled transcriptome and the pooled raw genomic reads, using a cascading Bloom filter built with ABySS to retain error-free transcriptome-overlapping k-mers. Extract the predicted exon sequences in FASTA format with gfa1_to_fasta. Map the pooled genomic reads to the predicted exons with Bowtie2 and call variants on the mapped reads with BCFTools. Filter out variants with a quality value below 20, then exclude variants within 35 bp of another variant or of a predicted exon boundary to define genotypable regions. Validate by comparing the surviving SNP positions against a reference set of known transcriptomic SNPs — in the tench case (Kumar et al., 2019, two pools of five diploid individuals at 52x coverage), EXFI detected all IEBs proximal to the 96 known SNPs and 92/96 were successfully genotyped (100% precision, 99% recall).

## Related tools

- **EXFI** (Predicts the splice graph and exon sequences from the transcriptome and WGS reads; output exons (FASTA via gfa1_to_fasta) are the mapping targets) — https://github.com/jlanga/exfi
- **abyss-bloom** (Builds the cascading Bloom filter used to filter raw genomic reads down to error-free transcriptome-overlapping k-mers) — https://github.com/bcgsc/abyss/
- **biobloomcategorizer** (Discards each WGS read that does not share at least one k-mer with the transcriptome, reducing the BF FPR by 68%–75%) — https://github.com/bcgsc/biobloom
- **Bowtie2** (Maps the pooled raw genomic reads to the predicted exon sequences before SNP calling)
- **BCFTools** (Calls SNPs from the reads mapped to predicted exons)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph transcriptome.fa genomic_reads/*.fa
```

## Evaluation signals

- Recovery rate of the known transcriptomic SNP set after quality (<20) and 35 bp proximity filtering — the tench benchmark reports 92/96 genotyped (~99% recall)
- All intron–exon boundaries proximal to the known SNPs are detected (100% precision over this SNP set)
- The Bloom-filter read filtering step should reduce the BF FPR by roughly 68%–75% while slightly improving classification metrics
- Exon predictions are output as a valid GFA1 splice graph and convert cleanly to FASTA with gfa1_to_fasta
- A large number of genotypable variants should survive filtering (the tench case yielded 228,931 SNPs and 26,169 indels suitable for genotyping)

## Limitations

- Mappability is more optimistic than exon precision: it covers only interior exon sections, not full exon sequences
- Mappability of all methods fell when the input transcriptome was de novo-assembled (e.g. herring from a single tissue), and special care is required regarding input transcriptome quality/tissue breadth
- Bloom filters return no false negatives but can produce false positives with a tunable BF FPR; read filtering mitigates this

## Evidence

- [intro] we applied EXFI to a recently published dataset on tench (Kumar et al., 2019) to evaluate its success in IEB detection for SNP discovery in exonic regions: "we applied EXFI to a recently published dataset on tench (Kumar et al., 2019) to evaluate its success in IEB detection for SNP discovery in exonic regions"
- [methods] variants with a quality value below 20 were filtered out: "variants with a quality value below 20 were filtered out"
- [methods] and then those that were within 35 bp to another variant or a predicted exon boundary: "and then those that were within 35 bp to another variant or a predicted exon boundary"
- [results] after quality and distance filtering, 228,931 SNPs and 26,169 indels were predicted suitable for genotyping: "after quality and distance filtering, 228,931 SNPs and 26,169 indels were predicted suitable for genotyping"
- [readme] Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"

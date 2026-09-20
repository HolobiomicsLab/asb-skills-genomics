---
name: cross-tool-exon-prediction-comparison
description: Use when comparing exon/IEB prediction tools (EXFI, ChopStitch, GMAP)
  on the same transcriptomes and WGS reads, needing precision/recall/F1 against a
  GFF3 reference plus exon mappability via BWA MEM.
when_to_use_negative:
- You need full splice-graph or transcript-path output rather than a tool-vs-tool
  performance benchmark (EXFI's GFA1 splice graph is the deliverable, not the comparison)
- The input transcriptome is de novo-assembled from a single tissue — mappability
  scores fall for all methods in the herring case, so cross-tool ranking may reflect
  transcriptome quality rather than tool performance
- You only need IEB positions for SNP genotyping; use EXFI directly instead of running
  a three-tool benchmark
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: EXFI
  role: k-mer/Bloom-filter-based exon and splice graph predictor run with k-mer length
    25 on transcriptome + WGS reads
  repo: https://github.com/jlanga/exfi
- name: ChopStitch
  role: comparison method using Bloom filters of WGS k-mers, run at version 1.0.0
    with default 50 bp k-mer length
- name: GMAP
  role: splice-aware transcript-to-genome aligner (version 2018.07.04, default parameters)
    used to extract IEB coordinates as comparator
- name: bedtools
  role: intersect subcommand compares predicted vs reference exon coordinates with
    ≥95% mutual overlap
- name: BWA MEM
  role: aligns predicted exons to the genomic reference to compute mappability
- name: SAMTools
  role: stores BWA MEM alignment results in BAM for CIGAR-based mappability scoring
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:2e912826a99c02bb76282b1aa8dee22000751b44518e508b167df5426eb17624
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BWA MEM
  - ChopStitch
  - EXFI
  - GMAP
  - SAMTools
  - bedtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cross-tool-exon-prediction-comparison

## Summary

Benchmark exon-prediction approaches — k-mer-based EXFI and ChopStitch versus splice-aware aligner GMAP — by computing precision, recall, F1, and mappability of predicted exons across species with and without reference annotations.

## When to use

Use when comparing exon/IEB prediction tools (EXFI, ChopStitch, GMAP) on the same transcriptomes and WGS reads, needing precision/recall/F1 against a GFF3 reference plus exon mappability via BWA MEM.

## When NOT to use

- You need full splice-graph or transcript-path output rather than a tool-vs-tool performance benchmark (EXFI's GFA1 splice graph is the deliverable, not the comparison)
- The input transcriptome is de novo-assembled from a single tissue — mappability scores fall for all methods in the herring case, so cross-tool ranking may reflect transcriptome quality rather than tool performance
- You only need IEB positions for SNP genotyping; use EXFI directly instead of running a three-tool benchmark

## Inputs

- Assembled genomes, transcriptomes, and GFF3 annotations (Ensembl release 91 GRCz10/GRCh38; NCBI GCA_000233375.4, GCA_000966335.1)
- Raw whole-genome sequencing (WGS) reads for each species
- Predicted exon sets from EXFI, ChopStitch, and GMAP on the same datasets

## Outputs

- Benchmark comparison table of precision, recall, and F1 per tool per species
- Mappability metrics: fraction of mapped exons and fraction with perfect CIGAR strings (e.g. ~99.5% for EXFI on Atlantic herring)

## How to apply

Download genomes, transcriptomes, and GFF3 annotations for the species under test (e.g. Ensembl release 91 GRCz10/GRCh38 for D. rerio and H. sapiens; NCBI assemblies GCA_000233375.4 for S. salar and GCA_000966335.1 for C. harengus) plus corresponding WGS reads. Run each predictor on identical inputs: EXFI via build_baited_bloom_filter (k-mer length 25) then build_splice_graph; GMAP (version 2018.07.04) with default parameters; ChopStitch (version 1.0.0) with its default 50 bp k-mer length. Transform reference GFF exon coordinates to transcript-based BED, accounting for strand and exon order, then compare predicted versus reference coordinates with bedtools intersect requiring a mutual overlap of at least 95%, computing precision, recall, and F1 via the compare_to_gff3 script. Assess mappability by aligning predicted exons to each genomic reference with BWA MEM, storing BAMs with SAMTools, and computing the fraction of mapped exons and of exons with a perfect CIGAR string (all matches or small indels, no base clipping).

## Related tools

- **EXFI** (k-mer/Bloom-filter-based exon and splice graph predictor run with k-mer length 25 on transcriptome + WGS reads) — https://github.com/jlanga/exfi
- **ChopStitch** (comparison method using Bloom filters of WGS k-mers, run at version 1.0.0 with default 50 bp k-mer length)
- **GMAP** (splice-aware transcript-to-genome aligner (version 2018.07.04, default parameters) used to extract IEB coordinates as comparator)
- **bedtools** (intersect subcommand compares predicted vs reference exon coordinates with ≥95% mutual overlap)
- **BWA MEM** (aligns predicted exons to the genomic reference to compute mappability)
- **SAMTools** (stores BWA MEM alignment results in BAM for CIGAR-based mappability scoring)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_baited_bloom_filter --kmer-length 25 transcriptome.fa reads_1.fq reads_2.fq
```

## Evaluation signals

- Reference and predicted exon coordinates match only under bedtools intersect with a mutual overlap of at least 95% of coordinates
- Mappability is reported as both fraction of mapped exons and fraction with a perfect CIGAR string (all matches or small indels, no base clipping)
- EXFI attains the highest perfect-mapping rate across all datasets (~99.5% on Atlantic herring), even where its exon precision is lower (worse-than-expected F1 for k-mer methods vs GMAP on herring)
- GFF-to-BED conversion must account for strand and exon order per the methods' coordinate-conversion scheme
- Same input datasets (species genomes, transcriptomes, WGS reads) are used for all three tools

## Limitations

- In the Atlantic herring reference transcriptome, the k-mer-based methods obtained worse-than-expected F1 scores than GMAP; EXFI was less accurate predicting exons but achieved the highest mappability across all datasets (99.5%)
- High mappability scores are more optimistic than exon precision and cover only interior exon sections, not full exon sequences
- Mappability of all methods fell when the input transcriptome was de novo-assembled (herring case), because that transcriptome came from a single tissue where lowly expressed transcripts were fragmented and other tissues' transcripts were missing

## Evidence

- [methods] Herring F1 / precision caveat: [evidence span withheld — anchor status: not_found]
- [methods] Mappability definition: [evidence span withheld — anchor status: not_found]
- [discussion] Herring result: "In that case, EXFI was less accurate predicting exons, but in terms of mappability, it achieved the highest results across all datasets (99.5%)."
- [discussion] Mappability interpretation: "This high mappability result, although more optimistic than the exon precision, means that even if the exon prediction is not precise, it is still usable for downstream analysis."
- [methods] Dataset provenance: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38"

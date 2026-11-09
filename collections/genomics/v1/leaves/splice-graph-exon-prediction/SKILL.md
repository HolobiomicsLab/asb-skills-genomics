---
name: splice-graph-exon-prediction
description: Use when only an assembled transcriptome and raw WGS shotgun reads are
  available (no reliable genomic reference) and exons, IEBs, or a splice graph are
  needed, e.g. for gene-targeted SNP genotyping in non-model organisms.
when_to_use_negative:
- A reliable reference genome and GFF3 annotation exist — a splice-aware aligner such
  as GMAP on the transcriptome/reads is the direct alternative and avoids Bloom-filter
  false positives
- The input transcriptome comes from a single tissue and is fragmented (e.g. the Atlantic
  herring case), where exon prediction accuracy degrades and mappability falls
- The goal is full-length transcript assembly rather than exon decomposition for SNP
  genotyping
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: EXFI
  role: 'Main pipeline: build_baited_bloom_filter and build_splice_graph produce the
    exon set and GFA1 splice graph'
  repo: https://github.com/jlanga/exfi
- name: biobloomtools
  role: biobloomcategorizer filters WGS reads that do not share a k-mer with the transcriptome
    before cascading Bloom filter construction
  repo: https://github.com/bcgsc/biobloom
- name: ABySS
  role: abyss-bloom builds the cascading Bloom filter of error-free transcriptome-overlapping
    k-mers from the filtered reads
  repo: https://github.com/bcgsc/abyss
- name: bedtools
  role: Used together with abyss-bloom to obtain exons from a transcriptome and raw
    genomic reads
- name: EXFI-docker
  role: Container image bundling all EXFI dependencies for running build_baited_bloom_filter
    and build_splice_graph
  repo: https://github.com/jlanga/exfi-docker
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:bd17f09ab7711a55215fab5c9868aa1eabbea9107e9948f1daa60b7d414df39b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - ABySS
  - BCFTools
  - BioPython
  - Bowtie2
  - EXFI
  - EXFI-docker
  - Pandas
  - Python
  - bedtools
  - biobloomtools
  - wgsim
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# splice-graph-exon-prediction

## Summary

Predicts exon sequences and the underlying splice graph from a de novo transcriptome assembly plus raw whole-genome sequencing (WGS) reads without a reference genome, using Bloom filters (EXFI) and outputting a GFA1 splice graph where exons are nodes, intron-exon boundaries (IEBs) are edges, transcripts are paths, and genes are connected components.

## When to use

Use when only an assembled transcriptome and raw WGS shotgun reads are available (no reliable genomic reference) and exons, IEBs, or a splice graph are needed, e.g. for gene-targeted SNP genotyping in non-model organisms.

## When NOT to use

- A reliable reference genome and GFF3 annotation exist — a splice-aware aligner such as GMAP on the transcriptome/reads is the direct alternative and avoids Bloom-filter false positives
- The input transcriptome comes from a single tissue and is fragmented (e.g. the Atlantic herring case), where exon prediction accuracy degrades and mappability falls
- The goal is full-length transcript assembly rather than exon decomposition for SNP genotyping

## Inputs

- Assembled transcriptome (FASTA), ideally multi-tissue so lowly expressed transcripts are not fragmented
- Raw whole-genome sequencing reads (FASTQ/FASTA; e.g. pooled genomic reads at ~52x coverage)
- Optional reference GFF3 annotation for benchmarking (e.g. Ensembl release 91, GRCz10)

## Outputs

- Splice graph in GFA1 format encoding predicted exon sequences and their connections into transcripts
- Predicted exon sequences in FASTA (via gfa1_to_fasta) usable for read mapping and SNP calling

## How to apply

Run EXFI's build_baited_bloom_filter on the transcriptome assembly and WGS reads in three steps: optionally discard each WGS read that does not share at least one k-mer with the transcriptome using biobloomcategorizer (this read-filtering step cut the BF FPR by 68%-75% in the zebrafish test and reduced processing time), then build a cascading Bloom filter with ABySS (abyss-bloom) retaining error-free transcriptome-overlapping k-mers with k fixed to 25 bp. Then run build_splice_graph, which in one step predicts exon sequences and the splice graph by marking IEBs wherever a transcript section's k-mers are absent from the WGS Bloom filter. Apply post-processing filters: remove exons shorter than k+q (q default 5), relax-merge exons overlapping by more than 10 bases, and polish donor/acceptor sites (usually GU/AG) by trimming long-overlapping exon pairs. Output is GFA1 encoding exon sequences and their connections; exon sequences can be extracted with gfa1_to_fasta for downstream steps such as Bowtie2 mapping and BCFTools SNP calling.

## Related tools

- **EXFI** (Main pipeline: build_baited_bloom_filter and build_splice_graph produce the exon set and GFA1 splice graph) — https://github.com/jlanga/exfi
- **biobloomtools** (biobloomcategorizer filters WGS reads that do not share a k-mer with the transcriptome before cascading Bloom filter construction) — https://github.com/bcgsc/biobloom
- **ABySS** (abyss-bloom builds the cascading Bloom filter of error-free transcriptome-overlapping k-mers from the filtered reads) — https://github.com/bcgsc/abyss
- **bedtools** (Used together with abyss-bloom to obtain exons from a transcriptome and raw genomic reads)
- **EXFI-docker** (Container image bundling all EXFI dependencies for running build_baited_bloom_filter and build_splice_graph) — https://github.com/jlanga/exfi-docker

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_baited_bloom_filter --help && docker run --rm exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- The BF FPR of the filtered run should be substantially lower than the unfiltered run (reported 68%-75% relative reduction; zebrafish low-memory 4 GB case fell from 32.6% to 8.1%)
- GFA1 output must parse as a valid splice graph: exons as nodes, IEBs as edges, transcripts as paths, genes as connected components
- No exons shorter than k+q (k=25, q default 5) survive the exon length filter
- Against a reference annotation, predicted exons should show high precision/recall with bedtools intersect requiring mutual overlap of at least 95% (compare_to_gff3 F1 up to 95.6% max, 94.8% filtered zebrafish low-memory)
- Retrospective SNP benchmark: IEBs detected proximal to 92 of 96 known tench transcriptomic SNPs (100% precision over this set) after quality <20 and 35 bp proximity filtering

## Limitations

- Bloom filters return no false negatives but produce false positives at a tunable BF FPR; unfiltered WGS reads inflate the FPR (up to 32.6% in low-memory settings) unless biobloomcategorizer read filtering is enabled
- Only 4.24% (human) and 5.68% (zebrafish) of genome bases are exons, so memory sizing matters; low-memory (4 GB) configurations rely on the read-filtering step
- EXFI was less accurate than comparators predicting exons for Atlantic herring, whose single-tissue transcriptome fragmented lowly expressed transcripts and lacked other tissues' transcripts; mappability fell for all methods with de novo-assembled input
- High mappability (up to 99.5%) is more optimistic than exon precision and covers only interior exon sections, not full exon sequences
- Missing version numbers and WGS dataset accession numbers (deferred to Table S1) complicate exact reproducibility

## Evidence

- [abstract] a Python pipeline that predicts the splice graph and exon sequences using an assembled transcriptome and raw whole-genome sequencing reads: "This paper presents EXFI, a Python pipeline that predicts the splice graph and exon sequences using an assembled transcriptome and raw whole-genome sequencing reads."
- [methods] read filtering with biobloomcategorizer: "each read of the WGS dataset that does not share at least one k‐mer with the transcriptome is discarded with biobloomcategorizer"
- [methods] exon length filtering rule: "exons of length less than k + q (q by default five) are filtered out (Figure 2c)"
- [methods] Bloom filter FPR behaviour: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] filtering improves FPR and metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [abstract] GFA1 output format: "The results are returned in GFA1 format, which encodes both the predicted exon sequences and how they are connected to form transcripts"
- [discussion] transcriptome quality caveat: "Another lesson learned is that special care has to be taken regarding the input transcriptome."
- [readme] EXFI repo install: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"

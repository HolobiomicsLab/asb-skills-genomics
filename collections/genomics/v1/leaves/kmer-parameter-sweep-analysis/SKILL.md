---
name: kmer-parameter-sweep-analysis
description: Use when tuning a k-mer-dependent pipeline (e.g., EXFI's build_baited_bloom_filter/build_splice_graph)
  and you must justify a fixed k by showing F1 stability across odd k values on a
  reference-annotated benchmark.
when_to_use_negative:
- The organism lacks a reference GFF3 annotation, so TP/FP/FN exon counts cannot be
  computed to score each k
- The goal is a single exon prediction run rather than parameter selection — run build_splice_graph
  directly with the established default k=25
- The input is an RNA-Seq alignment to a reference genome; splice-aware alignment
  (e.g., GMAP) is the appropriate route, not a Bloom-filter k-mer sweep
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: EXFI
  role: 'Pipeline under test: build_baited_bloom_filter and build_splice_graph are
    rerun per k-mer length; gfa1_to_fasta and compare_to_gff3 extract exons and compute
    metrics'
  repo: https://github.com/jlanga/exfi
- name: exfi-docker
  role: Containerized execution of EXFI commands at each k in the sweep
  repo: https://github.com/jlanga/exfi-docker
- name: abyss-bloom
  role: Builds the cascading Bloom filter from filtered WGS reads at each k-mer length
    and memory setting
  repo: https://github.com/bcgsc/abyss/
- name: bedtools
  role: BEDTools intersect with ≥95% mutual overlap to call true-positive exons between
    reference BED and predictions
- name: BioPython
  role: Sequence parsing underlying compare_to_gff3 metric computation
- name: Pandas
  role: Tabulation of per-k precision/recall/F1 metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:fcb3731cf58f86a0ab512735cc0641e0356b24b1ca655646036f5b618c29147e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - BioPython
  - EXFI
  - Pandas
  - Python
  - abyss-bloom
  - bedtools
  - exfi-docker
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# kmer-parameter-sweep-analysis

## Summary

Sweeps EXFI's k-mer length parameter on a simulated WGS benchmark (zebrafish, Ensembl release 91 GRCz10) and tabulates exon-prediction precision, recall, and F1 per k to identify a stable k window and select the pipeline default (k=25 bp from a stable F1 window of k=23–35).

## When to use

Use when tuning a k-mer-dependent pipeline (e.g., EXFI's build_baited_bloom_filter/build_splice_graph) and you must justify a fixed k by showing F1 stability across odd k values on a reference-annotated benchmark.

## When NOT to use

- The organism lacks a reference GFF3 annotation, so TP/FP/FN exon counts cannot be computed to score each k
- The goal is a single exon prediction run rather than parameter selection — run build_splice_graph directly with the established default k=25
- The input is an RNA-Seq alignment to a reference genome; splice-aware alignment (e.g., GMAP) is the appropriate route, not a Bloom-filter k-mer sweep

## Inputs

- Reference genome, transcriptome, and GFF3 annotation (zebrafish Ensembl release 91, assembly GRCz10)
- Simulated WGS reads generated with wgsim
- EXFI predicted exon FASTA and splice graph (GFA1) per k-mer length
- Reference exon coordinates (GFF3 converted to transcript-based BED)

## Outputs

- Per-k table of precision, recall, and F1 at each Bloom filter memory setting (4 GB and 60 GB)
- Identified stable F1 window of k-mer lengths (k=23–35 on the zebrafish benchmark)
- Selected default k-mer length for EXFI (k=25 bp)

## How to apply

Obtain a genome, transcriptome, and GFF3 annotation with a reference (zebrafish GRCz10, Ensembl release 91) and simulate WGS reads with wgsim. For each odd k in the sweep (paper runs k=21–65; the stable window is k=23–35), run EXFI's build_baited_bloom_filter at low and high Bloom filter memory (4 GB and 60 GB), then build_splice_graph with the matching k and extract predicted exons via gfa1_to_fasta. Convert reference exon coordinates from chromosome-based GFF3 to transcript-based BED, accounting for strand and exon order, and compare predicted vs. reference exons with BEDTools intersect requiring mutual overlap of at least 95%. Compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 (2PR/(P+R)) per k with the compare_to_gff3 script and tabulate per k (e.g., Pandas); select the default k that maximizes recall while keeping precision high within the stable F1 window.

## Related tools

- **EXFI** (Pipeline under test: build_baited_bloom_filter and build_splice_graph are rerun per k-mer length; gfa1_to_fasta and compare_to_gff3 extract exons and compute metrics) — https://github.com/jlanga/exfi
- **exfi-docker** (Containerized execution of EXFI commands at each k in the sweep) — https://github.com/jlanga/exfi-docker
- **abyss-bloom** (Builds the cascading Bloom filter from filtered WGS reads at each k-mer length and memory setting) — https://github.com/bcgsc/abyss/
- **bedtools** (BEDTools intersect with ≥95% mutual overlap to call true-positive exons between reference BED and predictions)
- **BioPython** (Sequence parsing underlying compare_to_gff3 metric computation)
- **Pandas** (Tabulation of per-k precision/recall/F1 metrics)

## Examples

```
docker run --rm --volume $PWD:$PWD --user "$UID:$GID" exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- F1 is near-constant across the k=23–35 window at both the 4 GB and 60 GB Bloom filter settings, matching the paper's justification of k=25 as default
- Precision = TP/(TP+FP) and recall = TP/(TP+FN) are computed from BEDTools intersect calls requiring mutual overlap of at least 95%, with F1 = 2PR/(P+R) per k
- The transcript-based BED conversion preserves strand and exon order relative to the chromosome-based GFF3, since mis-converted coordinates would corrupt TP/FP counting
- Every odd k in the sweep was run with a Bloom filter built at the same k, since mismatched k between build_baited_bloom_filter and build_splice_graph is undefined
- Exons shorter than k + q (q default 5) are filtered out by build_splice_graph, so very small k values are not directly comparable without noting this length filter

## Limitations

- Results were established on the zebrafish (GRCz10) benchmark with simulated WGS reads; the stability window may not transfer to other genome sizes or read profiles
- The k sweep is over odd values only (paper runs k=21–65), so even k values are untested
- Bloom filters can produce false positives with a tunable BF FPR, so k choice interacts with filter memory (4 GB vs 60 GB) and FPR
- The evaluation covers exon-coordinate accuracy only; downstream splice-graph edge quality and transcript path recovery are not directly scored by the sweep

## Evidence

- [methods] The exon and splice graph prediction procedure is carried out by the build_splice_graph script, which predicts in one step the exon sequences: "The exon and splice graph prediction procedure is carried out by the build_splice_graph script, which predicts in one step the exon sequences"
- [methods] Bloom filters return no false negatives but tunable false positives: "It does not return false negatives, but it can produce false positives with a tunable false‐positive rate (BF FPR)"
- [results] Read filtering improves BF FPR and metrics: "Filtering the reads resulted in a 68%–75% reduction of the BF FPR while also slightly improving all the classification metrics"
- [intro] EXFI purpose: exons from transcriptome and WGS reads via abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"
- [readme] Docker invocation pattern for EXFI commands used at each k: [evidence span withheld — anchor status: not_found]
- [other] Stable F1 window finding from the source card justifying default k=25: [evidence span withheld — anchor status: not_found]

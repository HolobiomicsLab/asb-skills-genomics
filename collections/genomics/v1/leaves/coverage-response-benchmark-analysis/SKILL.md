---
name: coverage-response-benchmark-analysis
description: Use when you need to determine the minimum WGS coverage required for
  reliable exon/splice-graph prediction, or to characterize how prediction quality
  degrades at very low or very high depth.
when_to_use_negative:
- You only need a single-shot exon prediction at a fixed coverage rather than a depth-response
  characterization
- The input transcriptome is of unknown quality or single-tissue origin — prediction
  metrics may reflect transcriptome fragmentation rather than coverage effects
- No reference annotation exists to compute precision/recall against (the benchmark
  requires GFF3 ground truth, e.g. Ensembl release 91 GRCz10)
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3308
tools:
- name: EXFI
  role: 'Pipeline under benchmark: build_baited_bloom_filter and build_splice_graph
    run per coverage subsample'
  repo: https://github.com/jlanga/exfi
- name: Seqtk
  role: Subsamples the WGS read set in 10% increments to generate the coverage titration
    series
- name: bedtools
  role: Intersect predicted exons with reference exons requiring mutual overlap of
    at least 95%
- name: Pandas
  role: Tabulates precision and recall versus genome coverage in coverage_metrics.csv
- name: Python
  role: Implementation language of the EXFI pipeline and benchmark analysis
  repo: https://github.com/jlanga/exfi
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.6587
    title: 'EXFI: Exon and splice graph prediction without a reference genome'
  source_content_hash: sha256:6971db2d9cf27490b04bb8913b561e9e82b63a0c789e85ce305d33661a2cc133
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jlanga/exfi
  tools:
  - EXFI
  - Pandas
  - Python
  - Seqtk
  - bedtools
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# coverage-response-benchmark-analysis

## Summary

Benchmark how WGS sequencing depth affects exon-prediction precision and recall by running a coverage titration series (e.g. Seqtk subsampling in 10% increments) through the EXFI pipeline and tabulating precision/recall/F1 versus coverage, as done for the zebrafish simulation (k=25 bp, >90% by 16x, max 96.7% precision / 94.7% recall at 26-37x).

## When to use

Use when you need to determine the minimum WGS coverage required for reliable exon/splice-graph prediction, or to characterize how prediction quality degrades at very low or very high depth.

## When NOT to use

- You only need a single-shot exon prediction at a fixed coverage rather than a depth-response characterization
- The input transcriptome is of unknown quality or single-tissue origin — prediction metrics may reflect transcriptome fragmentation rather than coverage effects
- No reference annotation exists to compute precision/recall against (the benchmark requires GFF3 ground truth, e.g. Ensembl release 91 GRCz10)

## Inputs

- Simulated or real WGS FASTQ/FASTA reads (zebrafish D. rerio simulation in the paper)
- Reference genome, transcriptome, and GFF3 annotation (Ensembl release 91, assembly GRCz10)
- Chosen k-mer length (25 bp) and Bloom filter memory settings (4 GB low, 60 GB high)

## Outputs

- Per-subsample predicted exons and splice graph in GFA1 format
- coverage_metrics.csv: precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 versus genome coverage for all subsamples
- Coverage-response curve identifying the minimum usable coverage and the optimum plateau

## How to apply

Subsample a simulated WGS read set in 10% increments with Seqtk to build a genome-coverage titration series. For each subsample run the EXFI pipeline (build_baited_bloom_filter followed by build_splice_graph) with k-mer length fixed at 25 bp, under both low (4 GB) and high (60 GB) Bloom filter memory settings. Convert reference exon coordinates from chromosome-based GFF/BED to transcript-based BED, accounting for strand and exon order, then compare predictions to the reference with BEDTools intersect requiring mutual overlap of at least 95% and compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 via the compare_to_gff3 script. Tabulate precision and recall versus genome coverage (coverage_metrics.csv) and read the response curve: performance should exceed 90% by ~16x, plateau at maxima (96.7% precision, 94.7% recall at 26-37x), then decline at very high depth because sequencing errors introduce spurious k-mers.

## Related tools

- **EXFI** (Pipeline under benchmark: build_baited_bloom_filter and build_splice_graph run per coverage subsample) — https://github.com/jlanga/exfi
- **Seqtk** (Subsamples the WGS read set in 10% increments to generate the coverage titration series)
- **bedtools** (Intersect predicted exons with reference exons requiring mutual overlap of at least 95%)
- **Pandas** (Tabulates precision and recall versus genome coverage in coverage_metrics.csv)
- **Python** (Implementation language of the EXFI pipeline and benchmark analysis) — https://github.com/jlanga/exfi

## Examples

```
seqtk sample wgs.fq 0.1 | docker run --rm -i --volume $PWD:$PWD --user $UID:$GID exfi:1.5.6 build_splice_graph --help
```

## Evaluation signals

- Precision and recall exceed 90% at approximately 16x sequencing depth
- The curve plateaus at maxima of 96.7% precision and 94.7% recall within 26-37x coverage
- Both metrics decline beyond the plateau due to sequencing errors, confirming the expected non-monotonic response
- All exon matches satisfy the mutual-overlap-at-least-95% criterion under BEDTools intersect
- Bloom filter memory setting (4 GB vs 60 GB) yields comparable metric trajectories, with low-memory settings computationally feasible

## Limitations

- At very high sequencing depth both precision and recall decline because sequencing errors dominate
- Metrics depend on transcriptome quality: single-tissue transcriptomes fragment lowly expressed transcripts and miss other tissues' transcripts (herring case)
- Results are demonstrated on simulated zebrafish (GRCz10) data; transfer to other genomes or real reads is not established by this benchmark

## Evidence

- [results] At 16× sequencing depth, EXFI's zebrafish precision and recall already exceed 90%, with maximum values of 96.7% precision and 94.7% recall reached between 26× and 37× coverage; beyond that, both decline due to sequencing errors.: [evidence span withheld — anchor status: not_found]
- [methods] Subsample the WGS read set in 10% increments using Seqtk to generate a coverage titration series.: [evidence span withheld — anchor status: not_found]
- [methods] Compare predicted exons to the reference with BEDTools intersect requiring a mutual overlap of at least 95%, and compute precision (TP/(TP+FP)), recall (TP/(TP+FN)), and F1 with the compare_to_gff3 script.: [evidence span withheld — anchor status: not_found]
- [methods] Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38: "Genomes, transcriptomes, and GFF3 annotations of D. rerio and H. sapiens were downloaded from Ensembl (release 91, assemblies GRCz10 and GRCh38"
- [readme] Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools: "Get exons from a transcriptome and raw genomic reads using abyss-bloom and bedtools"

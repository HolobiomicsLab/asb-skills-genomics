---
name: overlap-detection-benchmarking-on-simulated-reads
description: Use when you need to quantify how sequencing parameters (read length,
  coverage depth, accuracy) affect overlap graph quality of a long-read overlap detector
  on simulated data with known ground truth.
when_to_use_negative:
- The goal is de novo genome assembly rather than overlap graph quality evaluation
  — use the full assembler integration (Fedrann-Shasta) instead.
- Input is real sequencing data without a ground-truth reference alignment (e.g.,
  ONT R9 SRR10028111 or PacBio HiFi HG002); use the real-data benchmarking workflows
  from kNN-overlap-finder rather than simulation-based error-rate scoring.
- You need end-to-end assembly quality metrics (BUSCO primates_odb10, contiguity)
  rather than overlap-level error rates.
tools:
- name: Fedrann
  role: Overlap detector under evaluation; run per simulated condition with default
    FE/DR/ANN pipeline to produce overlaps.tsv
  repo: https://github.com/jzhang-dev/FEDRANN
- name: PBSIM3
  role: Long-read simulator generating FASTQ datasets across the read length, coverage,
    and accuracy parameter grid
- name: minimap2
  role: Aligns simulated reads back to the reference to define ground-truth overlaps
    before filtering
- name: kNN-overlap-finder
  role: Reproducible benchmarking and evaluation workflows, including PBSIM3-based
    simulation scripts
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:0c79fb624dbca93aa0efcb195975bd68255fa099e8dfb6147e836a08146d3a63
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/FEDRANN
  tools:
  - Fedrann
  - PBSIM3
  - Python
  - kNN-overlap-finder
  - minimap2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# overlap-detection-benchmarking-on-simulated-reads

## Summary

Benchmark overlap-detection pipelines (e.g., Fedrann) on simulated long-read datasets by sweeping read length, coverage, and sequencing accuracy, then scoring predicted overlaps against minimap2-derived ground truth as overlap graph error rates.

## When to use

Use when you need to quantify how sequencing parameters (read length, coverage depth, accuracy) affect overlap graph quality of a long-read overlap detector on simulated data with known ground truth.

## When NOT to use

- The goal is de novo genome assembly rather than overlap graph quality evaluation — use the full assembler integration (Fedrann-Shasta) instead.
- Input is real sequencing data without a ground-truth reference alignment (e.g., ONT R9 SRR10028111 or PacBio HiFi HG002); use the real-data benchmarking workflows from kNN-overlap-finder rather than simulation-based error-rate scoring.
- You need end-to-end assembly quality metrics (BUSCO primates_odb10, contiguity) rather than overlap-level error rates.

## Inputs

- Reference genome FASTA (e.g., chromosome 22 from T2T-CHM13 v2.0)
- Simulated long-read FASTQ files, one per parameter-grid condition (PBSIM3: read lengths 10/20/30 kb; coverages 10/30/50x; accuracies 91/95/99%)
- Fedrann overlap predictions (overlaps.tsv)
- minimap2 alignment (PAF) of simulated reads against the reference

## Outputs

- Ground-truth overlap sets derived from filtered minimap2 alignments (>= 100 matched bases, >= 10% alignment identity; reads >= 5 kb, aligned fraction >= 50%, MAPQ >= 30)
- Per-condition overlap graph error rates (false-positive/false-negative overlap fraction)
- Aggregated table overlap_error_rates.csv

## How to apply

Extract the target reference (e.g., chromosome 22 from T2T-CHM13 v2.0) and simulate long reads per grid point with PBSIM3 — here read lengths 10/20/30 kb, coverages 10/30/50x, accuracies 91/95/99% — yielding one FASTQ per condition. Run the overlap detector (Fedrann) with its default pipeline: Jellyfish k-mer counting with --kmer-min-multiplicity 2 to remove sequencing errors, --kmer-sample-fraction 0.05 k-mer sampling, ICF weighting as a practical approximation of IDF, sparse random projection, and NNDescent with cosine distance under multi-threaded execution. Generate ground truth by aligning simulated reads back to the reference with minimap2, filtering alignments at >= 100 matched bases and >= 10% alignment identity, and keeping reference reads with length >= 5 kb, aligned fraction >= 50%, and mapping quality >= 30. Compare the detector's overlaps.tsv against ground-truth alignments per condition and compute false-positive/false-negative overlap fractions; aggregate per-condition rates into overlap_error_rates.csv.

## Related tools

- **Fedrann** (Overlap detector under evaluation; run per simulated condition with default FE/DR/ANN pipeline to produce overlaps.tsv) — https://github.com/jzhang-dev/FEDRANN
- **PBSIM3** (Long-read simulator generating FASTQ datasets across the read length, coverage, and accuracy parameter grid)
- **minimap2** (Aligns simulated reads back to the reference to define ground-truth overlaps before filtering)
- **kNN-overlap-finder** (Reproducible benchmarking and evaluation workflows, including PBSIM3-based simulation scripts) — https://github.com/jzhang-dev/kNN-overlap-finder

## Examples

```
fedrann -i sim_chr22_30x_20kb_99pct.fastq -o output_30x_20kb_99/ -k 15 --kmer-sample-fraction 0.05 --seed 602 --kmer-min-multiplicity 2 --threads 32
```

## Evaluation signals

- At 30x coverage and 20 kb mean read length, overlap graph error rate falls from ~8.34% at 91% accuracy to ~5.55% at 99% accuracy — verify this monotonic trend in overlap_error_rates.csv.
- overlaps.tsv schema holds: query_name, query_orientation, target_name, target_orientation, neighbor_rank, distance, with smaller distance indicating higher similarity.
- Ground-truth alignments satisfy both filters: >= 100 matched bases and >= 10% alignment identity; reference reads additionally pass length >= 5 kb, aligned fraction >= 50%, MAPQ >= 30.
- Each grid condition produces exactly one simulated FASTQ and one corresponding Fedrann run; error rates are computable for every cell of the 3x3x3 grid.
- Fedrann's per-condition error rates should be comparable to or better than MECAT2, minimap2, and wtdbg2 at competitive runtime.

## Limitations

- Fedrann's memory consumption is higher than most existing overlap detection tools, which can constrain large-scale benchmark runs.
- minimap2 PAF output can exceed 4 TB for human whole-genome read alignments; the identity threshold was raised to 30% for human PAF files, which must be accounted for when reproducing filters.
- Reproducibility depends solely on GitHub repositories (kNN-overlap-finder); no DOI or Zenodo archive is provided.
- PCA- and UMAP-based dimensionality reduction fail to scale to human-genome-sized feature matrices, limiting alternative DR choices in the benchmark.

## Evidence

- [discussion] All alignments were filtered using two criteria: (1) $≥ 100 matched bases and (2) $≥ 10% alignment identity: [evidence span withheld — anchor status: not_found]
- [discussion] read length $≥ 5 kb; (2) aligned fraction $≥ 50%; (3) mapping quality $≥ 30: [evidence span withheld — anchor status: not_found]
- [readme] Ignore k-mers that appear less than 2 times to filter out sequencing errors: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"
- [methods] Applies inverse collection frequency (ICF) as a practical approximation of IDF: "Applies inverse collection frequency (ICF) as a practical approximation of IDF"
- [discussion] Reproducible workflows for benchmarking and evaluation: "Reproducible workflows for benchmarking and evaluation: https://github.com/jzhang-dev/kNN-overlap-finder"

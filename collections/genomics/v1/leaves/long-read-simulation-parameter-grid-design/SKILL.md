---
name: long-read-simulation-parameter-grid-design
description: Use when you need to isolate the effect of long-read data characteristics
  (read length, coverage, accuracy) on a pipeline's performance by simulating reads
  against a reference genome with controlled parameter values.
when_to_use_negative:
- When the goal is to evaluate on real sequencing data — use empirical reads (e.g.,
  ONT R9 SRR10028111, PacBio HiFi HG002) with benchmarking workflows instead of simulation
- When you lack a high-quality reference genome to simulate against, since ground
  truth depends on aligning reads back to the reference
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: PBSIM3
  role: Simulates long reads with controlled depth, accuracy, and read length configurations
    for each grid point
  repo: https://github.com/jzhang-dev/kNN-overlap-finder
- name: minimap2
  role: Aligns simulated reads back to the reference chromosome to produce ground-truth
    overlaps
- name: Fedrann
  role: Tool under test; run per simulated dataset to produce overlaps.tsv for error-rate
    computation
  repo: https://github.com/jzhang-dev/FEDRANN
- name: Python
  role: Aggregates per-condition error rates into overlap_error_rates.csv
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giag048
    title: 'FEDRANN: effective long-read overlap detection based on dimensionality
      reduction and approximate nearest neighbors'
  source_content_hash: sha256:937c4ac85030b9c90b528974495e1fa129d0e76caf6ba7cd04950cbb2cbfad2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jzhang-dev/kNN-overlap-finder
  tools:
  - Fedrann
  - PBSIM3
  - Python
  - minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Long-read simulation parameter grid design

## Summary

Designs controlled PBSIM3 simulation grids over read length, coverage depth, and sequencing accuracy to generate FASTQ datasets with known ground truth, enabling systematic assessment of how read characteristics affect downstream overlap-graph error rates.

## When to use

Use when you need to isolate the effect of long-read data characteristics (read length, coverage, accuracy) on a pipeline's performance by simulating reads against a reference genome with controlled parameter values.

## When NOT to use

- When the goal is to evaluate on real sequencing data — use empirical reads (e.g., ONT R9 SRR10028111, PacBio HiFi HG002) with benchmarking workflows instead of simulation
- When you lack a high-quality reference genome to simulate against, since ground truth depends on aligning reads back to the reference

## Inputs

- Reference genome assembly (e.g., chromosome 22 from T2T-CHM13 v2.0)
- Parameter grid specification: read lengths (10/20/30 kb), coverages (10/30/50x), accuracies (91/95/99%)
- Simulated FASTQ per grid condition (PBSIM3 output)
- Tool overlaps output (overlaps.tsv) per condition

## Outputs

- One PBSIM3-simulated FASTQ file per grid condition
- Ground-truth overlap set per condition from minimap2 alignments with applied filters
- Per-condition overlap graph error rates (false-positive/false-negative overlap fraction)
- Aggregated overlap_error_rates.csv across all conditions

## How to apply

Extract the target reference (e.g., chromosome 22 from the human T2T-CHM13 v2.0 assembly) and simulate long reads with PBSIM3 across a full parameter grid — read lengths 10, 20, and 30 kb; coverages 10, 30, and 50x; accuracies 91%, 95%, and 99% — producing one FASTQ per condition. For each simulated dataset, run the pipeline under test with its default settings (for Fedrann: Jellyfish k-mer counting with --kmer-min-multiplicity 2 filtering, --kmer-sample-fraction 0.05 k-mer sampling, ICF weighting, sparse random projection, and NNDescent with cosine distance, multi-threaded). Generate ground-truth overlaps by aligning simulated reads back to the reference with minimap2 and applying alignment filters (>= 100 matched bases, >= 10% alignment identity; reference reads kept if length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30). Compare predicted overlaps.tsv against ground-truth alignments per condition and compute overlap graph error rates (false-positive/false-negative overlap fraction), then aggregate into a per-condition CSV to reveal parameter effects.

## Related tools

- **PBSIM3** (Simulates long reads with controlled depth, accuracy, and read length configurations for each grid point) — https://github.com/jzhang-dev/kNN-overlap-finder
- **minimap2** (Aligns simulated reads back to the reference chromosome to produce ground-truth overlaps)
- **Fedrann** (Tool under test; run per simulated dataset to produce overlaps.tsv for error-rate computation) — https://github.com/jzhang-dev/FEDRANN
- **Python** (Aggregates per-condition error rates into overlap_error_rates.csv)

## Examples

```
fedrann -i sim_reads_20kb_30x_acc95.fastq -o out_20kb_30x_acc95/ -k 15 --kmer-sample-fraction 0.05 --kmer-min-multiplicity 2 --seed 602 --threads 32
```

## Evaluation signals

- One FASTQ exists per grid condition (3 read lengths × 3 coverages × 3 accuracies = 27 conditions)
- Ground-truth alignments pass both filters: >= 100 matched bases and >= 10% alignment identity; reference reads satisfy length >= 5 kb, aligned fraction >= 50%, mapping quality >= 30
- Per-condition error rates are computed as false-positive/false-negative overlap fractions relative to the minimap2-derived ground truth
- At 30x coverage / 20 kb reads, error rate should fall from 8.34% to 5.55% as accuracy rises from 91% to 99%, confirming the reported robustness pattern

## Limitations

- Simulation-based evaluation depends on the accuracy of PBSIM3's error model; real-data behavior may differ
- Large ground-truth outputs can be problematic — minimap2 PAF files for human whole-genome read alignments exceeded 4TB, requiring a raised 30% identity threshold
- Parameter-grid simulations on a single chromosome may not capture whole-genome repeat complexity

## Evidence

- [readme] PBSIM3-based simulation: Varied depth, accuracy, and read length configurations: [evidence span withheld — anchor status: not_found]
- [discussion] Reference read filter thresholds: [evidence span withheld — anchor status: not_found]
- [discussion] Alignment filter criteria: [evidence span withheld — anchor status: not_found]
- [discussion] minimap2 PAF size limitation: "for minimap2 PAF files (which produced exceptionally large outputs exceeding 4TB for human whole-genome sequencing read alignments), we increased the identity threshold to 30%"
- [methods] k-mer min multiplicity option: "Ignore k-mers that appear less than 2 times to filter out sequencing errors"

---
name: m6a-gold-standard-benchmarking
description: Use when comparing multiple Nanopore dRNA-seq m6A callers (e.g. 14 tools)
  on oligos, yeast, mouse, or human datasets against reference m6A peak sets, or when
  deciding which bin filters or cut-off thresholds improve tool precision/recall.
when_to_use_negative:
- The analysis targets a single tool or needs absolute m6A site quantification rather
  than comparative benchmarking against gold-standard peaks
- No orthogonal reference m6A peak set (bed file) exists for the organism, since the
  biological datasets lack a ground truth and rely on short-read-derived reference
  sets
- Input is raw signal only without basecalled FAST5 data, which the pipeline's --fast5_slot
  handling requires
edam_operation: http://edamontology.org/operation_3946
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3169
tools:
- name: NanOlympicsMod
  role: Containerized Nextflow pipeline that executes and compares the 14 m6A detection
    tools, performs genome binning, filter construction, and PR/AUPRC analysis
  repo: https://github.com/mfurla/NanOlympicsMod
- name: Nextflow
  role: Workflow engine underlying the NanOlympicsMod pipeline for reproducible benchmarking
    runs
  repo: https://github.com/mfurla/NanOlympicsMod
- name: nanoporetech/ont_fast5_api
  role: Interface for reading ONT .fast5 files containing basecalled read data used
    by the pipeline
  repo: https://github.com/nanoporetech/ont_fast5_api
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae001
    title: Benchmarking of computational methods for m6A profiling with Nanopore direct
      RNA sequencing
  source_content_hash: sha256:2f14b6570c73d6a18498ec1a4f15b170563ad9fa30e1d5d798d99e8fbe95a047
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfurla/NanOlympicsMod
  tools:
  - NanOlympicsMod
  - Nextflow
  - nanoporetech/ont_fast5_api
  edam_operation: http://edamontology.org/operation_3946
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# m6a-gold-standard-benchmarking

## Summary

Benchmark dRNA-seq m6A detection tools against gold-standard peak bed files using the NanOlympicsMod Nextflow pipeline, sweeping stringency cut-offs to build precision-recall curves and AUPRC estimates with 95% confidence intervals, and testing whether bin-set restrictions (RRACH motifs, exclusion zones, high coverage) improve performance.

## When to use

Use when comparing multiple Nanopore dRNA-seq m6A callers (e.g. 14 tools) on oligos, yeast, mouse, or human datasets against reference m6A peak sets, or when deciding which bin filters or cut-off thresholds improve tool precision/recall.

## When NOT to use

- The analysis targets a single tool or needs absolute m6A site quantification rather than comparative benchmarking against gold-standard peaks
- No orthogonal reference m6A peak set (bed file) exists for the organism, since the biological datasets lack a ground truth and rely on short-read-derived reference sets
- Input is raw signal only without basecalled FAST5 data, which the pipeline's --fast5_slot handling requires

## Inputs

- Base-called FAST5 folders (with --fast5_slot specifying the slot containing basecalled bases) listed in a tab-separated samples file with sample name, condition, and path
- Reference genome and transcriptome annotation files (GENCODE M23 for mouse, hg38 for human, SK1 reference genome for yeast)
- Gold-standard m6A peaks bed file (--peaksfile) from orthogonal techniques; for yeast, m6A peaks lifted to the SK1 reference genome
- Bin length setting (--binLength) for genome binning
- Per-bin m6A score matrix from upstream NanOlympicsMod genome binning

## Outputs

- Precision-recall curve figures per tool and per dataset (and per filtered bin set)
- AUPRC-with-95%-CI summary table (auprc_confidence_intervals.tsv)
- Per-bin m6A confidence-score matrix labeled as gold-standard positives/negatives
- Filtered bin sets (RRACH-containing bins, bins outside 100-nt exon-end exclusion zones, highly expressed bins >100 read counts) with recomputed precision/recall per tool

## How to apply

Run NanOlympicsMod (git clone https://github.com/mfurla/NanOlympicsMod.git; configure pipeline.conf; `nextflow -c pipeline.conf run pipeline.nf --samples=... --resultsDir=...`) to generate a per-bin confidence-score matrix using genome bins sized by the --binLength parameter. Label each bin positive/negative via the gold-standard m6A peaks bed file (--peaksfile), then sweep each tool's native cut-off (nanom6A probability thresholds, Yanocomp/differr FDR, DRUMMER P-value, EpiNano error thresholds, plus 'default' and 'relaxed' threshold sets) and compute precision/recall at each threshold with the pipeline's R post-processing scripts and PRROC. Derive AUPRC point estimates with 95% CIs per tool-dataset pair, noting that CI width depends on positive-bin count (oligos 1543, yeast 1453, mouse 84060, human 18932). Optionally annotate bins with three binary flags — RRACH motif presence in the bin sequence, distance >100 nt beyond the exon-end exclusion zone, and total read count >100 — and recompute PR curves per filtered bin set to test whether restriction improves performance.

## Related tools

- **NanOlympicsMod** (Containerized Nextflow pipeline that executes and compares the 14 m6A detection tools, performs genome binning, filter construction, and PR/AUPRC analysis) — https://github.com/mfurla/NanOlympicsMod
- **Nextflow** (Workflow engine underlying the NanOlympicsMod pipeline for reproducible benchmarking runs) — https://github.com/mfurla/NanOlympicsMod
- **nanoporetech/ont_fast5_api** (Interface for reading ONT .fast5 files containing basecalled read data used by the pipeline) — https://github.com/nanoporetech/ont_fast5_api

## Examples

```
nextflow -c pipeline.conf run pipeline.nf --samples="/path/to/samples.txt" --resultsDir="/path/to/resultsDir"
```

## Evaluation signals

- auprc_confidence_intervals.tsv contains AUPRC point estimates with finite 95% CI bounds for every tool-dataset pair
- Confidence intervals are wider for low-positive-count datasets (oligos 1543 positives, yeast 1453) than for mouse (84060) and human (18932), matching reported variability
- Validation pattern confirmed: marginal precision gain for the RRACH filter, no significant gain for the exclusion-zone filter, marked precision and recall improvement for the high-expression (>100 read counts) filter
- AUPRC CI width shrinks monotonically as the number of gold-standard positive bins increases
- Cut-off sweeps (default vs relaxed threshold sets) produce distinct PR curves, confirming the precision-recall trade-off is tunable

## Limitations

- The yeast, mouse and human datasets lack a ground truth; reference m6A sets come from orthogonal short-read methods, so possible differences in bias between the orthogonal references and nanopore-based techniques confound performance assessment
- The synthetic oligos dataset is an artificial condition that poorly recapitulates the relative location and frequency of in vivo marks and lacks the confounding effect of proximal RNA modifications
- Several tools could not complete the benchmark runs; for a few, runs were abandoned due to lack of progress or unexpected generation of empty output files
- The full benchmark entails a high computational burden: >22 M files totalling 4 TB of raw data
- m6Anet and DENA were applied outside their training species (H. sapiens, A. thaliana) and m6Anet used the default human-trained neural network without retraining for yeast/mouse

## Evidence

- [abstract] We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data.: "We developed NanOlympicsMod, a Nextflow pipeline exploiting containerized technology for comparing 14 tools for m6A detection on dRNA-seq data."
- [discussion] Only by filtering for highly expressed bins significatively boosted precision and recall.: "Only by filtering for highly expressed bins significatively boosted precision and recall."
- [discussion] Restricting to specific bin sets (RRACH motifs, high coverage, far from exclusion zones) tested for performance improvement.: "We tested whether restricting to specific sets of bins, such as those associated with RRACH motifs, high coverage or far from exclusion zones, could improve the performance"
- [methods] --peaksfile Path to bed file with set of m6A gold-standard peaks: "--peaksfile Path to bed file with set of m6A gold-standard peaks"
- [readme] The source code for the NanOlympicsMod workflow is available at the GitHub repository.: "git clone https://github.com/mfurla/NanOlympicsMod.git"

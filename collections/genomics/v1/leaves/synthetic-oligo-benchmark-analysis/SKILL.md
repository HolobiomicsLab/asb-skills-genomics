---
name: synthetic-oligo-benchmark-analysis
description: Use when evaluating whether a read-level RNA modification caller can
  resolve phase-shifted or patterned modifications hidden in synthetic oligo benchmarks
  whose aggregate site-level signals are uninformative.
when_to_use_negative:
- Data are natural mRNA without per-read ground-truth modification labels — benchmark
  AUC cannot be computed without known even-cycle true positives
- The question is site-level stoichiometry in biological samples (e.g., GLORI correlation),
  not read-level pattern resolution on synthetic oligos
- Reads cover only a single modification status per site (~0% or ~100%), so any competent
  read-level caller separates them and the benchmark loses discriminating power
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0080
tools:
- name: RODAN
  role: Fully-convolutional backbone model (backbone.torch) used to basecall TEST2
    fast5 files and extract hidden signal features for mAFiA
  repo: https://github.com/biodlab/RODAN
- name: mAFiA
  role: Method under test; produces per-read m6A modification probabilities at GGACU
    sites via logistic regression on RODAN features
  repo: https://github.com/dieterich-lab/mAFiA
- name: minimap2
  role: Aligns basecalled TEST2 reads to the reference sequences (splice/uf preset,
    -k14) before filtering
- name: samtools
  role: Filters, sorts, and indexes the BAM with 'samtools view -q50' to remove reads
    with mapping quality below 50
- name: CHEUI
  role: Comparison method for single-molecule m6A detection run on the same TEST2
    dataset
- name: m6Anet
  role: Comparison method for single-molecule m6A detection run on the same TEST2
    dataset
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-47661-2
    title: Detecting m6A at single-molecular resolution via direct RNA sequencing
      and realistic training data
  source_content_hash: sha256:5322349d0fada671ff0b89350290144e23b47ab7af23f3a110885fcb7438f126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biodlab/RODAN
  tools:
  - CHEUI
  - RODAN
  - m6Anet
  - mAFiA
  - minimap2
  - samtools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# synthetic-oligo-benchmark-analysis

## Summary

Benchmark single-read m6A detection tools (mAFiA vs CHEUI and m6Anet) on synthetic TEST2 periodic-repeat dRNA-Seq reads, where GGACU sites are indistinguishable (~50% modification) at the aggregate site level, by computing precision-recall AUC against known even-cycle true positives.

## When to use

Use when evaluating whether a read-level RNA modification caller can resolve phase-shifted or patterned modifications hidden in synthetic oligo benchmarks whose aggregate site-level signals are uninformative.

## When NOT to use

- Data are natural mRNA without per-read ground-truth modification labels — benchmark AUC cannot be computed without known even-cycle true positives
- The question is site-level stoichiometry in biological samples (e.g., GLORI correlation), not read-level pattern resolution on synthetic oligos
- Reads cover only a single modification status per site (~0% or ~100%), so any competent read-level caller separates them and the benchmark loses discriminating power

## Inputs

- TEST2 periodic-repeat synthetic dRNA-Seq fast5 files
- TEST2 reference sequences (FASTA)
- RODAN-based backbone model backbone.torch and pickled classifiers directory
- Known even-cycle true-positive modification labels

## Outputs

- Per-read modification probabilities per tool (mAFiA, CHEUI, m6Anet)
- Precision-recall curves and PR AUC values per tool
- pr_auc_comparison.csv aggregating per-tool PR AUC values

## How to apply

Basecall the TEST2 dRNA-Seq fast5 files with the RODAN-based backbone model (backbone.torch), align reads to the TEST2 reference with minimap2 (splice/uf preset, -k14), and filter with samtools view -q50 to remove reads with mapping quality below 50. Run mAFiA (test_mAFiA) with --min_coverage 50, --max_num_reads 1000, --mod_prob_thresh 0.5 to obtain per-read modification probabilities, and run CHEUI and m6Anet on the same dataset as comparison methods. Classify each read at even-cycle GGACU positions as modified/unmodified using a P threshold of 0.5, compute precision-recall curves against the known even-cycle true positives, and aggregate per-tool PR AUC values into pr_auc_comparison.csv. The rationale is that ensemble-level metrics are blind to the phase-shifted pattern, so read-level PR AUC is the discriminating metric.

## Related tools

- **RODAN** (Fully-convolutional backbone model (backbone.torch) used to basecall TEST2 fast5 files and extract hidden signal features for mAFiA) — https://github.com/biodlab/RODAN
- **mAFiA** (Method under test; produces per-read m6A modification probabilities at GGACU sites via logistic regression on RODAN features) — https://github.com/dieterich-lab/mAFiA
- **minimap2** (Aligns basecalled TEST2 reads to the reference sequences (splice/uf preset, -k14) before filtering)
- **samtools** (Filters, sorts, and indexes the BAM with 'samtools view -q50' to remove reads with mapping quality below 50)
- **CHEUI** (Comparison method for single-molecule m6A detection run on the same TEST2 dataset)
- **m6Anet** (Comparison method for single-molecule m6A detection run on the same TEST2 dataset)

## Examples

```
test_mAFiA --bam_file minimap.q50.bam --fast5dir fast5_TEST2 --backbone_model_path backbone.torch --classifier_model_dir classifiers --min_coverage 50 --max_num_reads 1000 --mod_prob_thresh 0.5
```

## Evaluation signals

- mAFiA's PR AUC at even-cycle GGACU positions substantially exceeds that of CHEUI and m6Anet, which show 'considerable deterioration' (Fig. 2f expectation)
- All reads passing the q50 filter have mapping quality ≥ 50, and every evaluated site has coverage meeting --min_coverage 50
- Per-read classifications use the article's threshold: read-level P(m6A) thresholded at 0.5
- pr_auc_comparison.csv contains one PR AUC value per tool (mAFiA, CHEUI, m6Anet) computed against the same even-cycle ground truth
- Aggregate site-level methylation of GGACU positions is ~50%, confirming the benchmark is unresolvable at the ensemble level

## Limitations

- The synthetic periodic-repeat design means only even-cycle GGACU positions are true positives; results do not transfer directly to natural mRNA without read-level labels
- Reads with mapping quality below 50 are excluded, and sites below the minimum coverage of 50 are not evaluated
- CHEUI and m6Anet must be run with their own recommended settings; cross-tool differences may partly reflect training-data domain mismatch rather than algorithmic capability
- mAFiA covers a specific motif set (e.g., GGACU); sites outside covered DRACH motifs cannot be evaluated

## Evidence

- [results] On the TEST2 benchmark, mAFiA resolves the underlying read-level m6A pattern, whereas analogous calculations with CHEUI and m6Anet show considerable deterioration in performance (Fig. 2f).: [evidence span withheld — anchor status: not_found]
- [abstract] Our approach uncovers distinct modification patterns in single molecules that would appear identical at the ensemble level.: "Our approach uncovers distinct modification patterns in single molecules that would appear identical at the ensemble level."
- [results] Read-level P_n(m6A) is thresholded at 0.5 to classify each nucleotide on a read into modified or unmodified status: [evidence span withheld — anchor status: not_found]
- [readme] samtools view -bST ${ref} -q50 - \: "minimap2 --secondary=no -ax splice -uf -k14 -t 36 --cs ${ref} ${basecall} \
| samtools view -bST ${ref} -q50 - \"
- [results] Reads with mapq below 50 are filtered out.: "Reads with mapq below 50 are filtered out."
- [readme] backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction: "backbone.torch: [RODAN](https://github.com/biodlab/RODAN)-based neural network for basecalling and feature extraction"

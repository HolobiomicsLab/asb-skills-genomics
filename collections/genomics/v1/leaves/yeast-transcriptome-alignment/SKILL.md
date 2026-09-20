---
name: yeast-transcriptome-alignment
description: Use when raw ONT fast5 files from yeast direct RNA sequencing (WT vs
  ime4Δ knockout) must be base-called and anchored to a yeast transcriptome reference
  before signal-level modification detection (e.g., Tombo resquiggle) or site-level
  metric computation.
when_to_use_negative:
- Input is already base-called error feature tables (e.g., EpiNano mismatch/insertion/deletion/quality
  per k-mer) — no signal resquiggling or transcriptome re-alignment step is needed
- The dataset is not yeast direct RNA sequencing or the fixed 61,163-site evaluation
  ground truth does not apply, making the accuracy/recovery baselines meaningless
- Goal is EpiNano-SVM-style prediction from base-called features without squiggle
  realignments, which the paper explicitly designed to avoid manipulating raw current
  intensity data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0102
tools:
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform whose raw fast5 files are the input; ONT basecaller
    (Albacore/Guppy) provides per-read basecalls and signal offsets needed for resquiggle
- name: Tombo
  role: Performs resquiggle — re-aligning each read's raw current-intensity signal
    against the expected signal derived from its basecalls and the yeast reference
    transcriptome — and canonical sample-comparison mode using ime4Δ as the control
- name: EpiNano
  role: Baseline comparator that operates on base-called features from the same aligned
    yeast data; its 87% in-vivo accuracy and 61,163-site evaluation set define the
    metrics the alignment supports
  repo: https://github.com/novoalab/EpiNano
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:7b605b33e8a4fde8bf910f2d940dbed52d3ce718de34b00e61c0a484d805306e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  - Tombo
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# yeast-transcriptome-alignment

## Summary

Aligns ONT direct RNA sequencing reads from yeast wild-type and ime4Δ samples to the yeast transcriptome reference so that raw current-intensity signals can be re-squiggled and m6A site predictions evaluated on the fixed 61,163-site set. It supplies the read-to-transcript coordinate frame required for Tombo resquiggle and baseline comparison against EpiNano.

## When to use

Use when raw ONT fast5 files from yeast direct RNA sequencing (WT vs ime4Δ knockout) must be base-called and anchored to a yeast transcriptome reference before signal-level modification detection (e.g., Tombo resquiggle) or site-level metric computation.

## When NOT to use

- Input is already base-called error feature tables (e.g., EpiNano mismatch/insertion/deletion/quality per k-mer) — no signal resquiggling or transcriptome re-alignment step is needed
- The dataset is not yeast direct RNA sequencing or the fixed 61,163-site evaluation ground truth does not apply, making the accuracy/recovery baselines meaningless
- Goal is EpiNano-SVM-style prediction from base-called features without squiggle realignments, which the paper explicitly designed to avoid manipulating raw current intensity data

## Inputs

- Raw ONT direct RNA fast5 files for yeast wild-type (WT) sample
- Raw ONT direct RNA fast5 files for yeast ime4Δ knockout sample
- Yeast transcriptome reference (FASTA) used for alignment in the published dataset

## Outputs

- Base-called reads with per-read basecalls and signal offsets (FASTQ + base-called fast5)
- Read-to-transcript alignments against the yeast transcriptome reference
- Re-squiggled reads with per-site raw current-intensity features mapped to transcript coordinates
- tombo_canonical_comparison_metrics.csv with recovery/specificity/accuracy on the 61,163-site set

## How to apply

Download the deposited yeast wild-type and ime4Δ direct RNA raw fast5 files for the published dataset together with the yeast transcriptome reference used for alignment. Base-call the fast5 files with the ONT basecaller (e.g., Albacore/Guppy) so that Tombo has per-read basecalls and signal offsets to work from, since resquiggle compares raw current intensity against the expected signal derived from basecalls plus the reference transcriptome. Align base-called reads to the yeast transcriptome and then run Tombo resquiggle to re-align each read's raw current-intensity signal against that expected signal. Restrict downstream site calls to the fixed 61,163-site evaluation set and use the ime4Δ strain — which lacks m6A — as the canonical control. Rationale: signal-level methods need the transcript-anchored coordinate frame that base-called error-based methods like EpiNano do not require.

## Related tools

- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform whose raw fast5 files are the input; ONT basecaller (Albacore/Guppy) provides per-read basecalls and signal offsets needed for resquiggle)
- **Tombo** (Performs resquiggle — re-aligning each read's raw current-intensity signal against the expected signal derived from its basecalls and the yeast reference transcriptome — and canonical sample-comparison mode using ime4Δ as the control)
- **EpiNano** (Baseline comparator that operates on base-called features from the same aligned yeast data; its 87% in-vivo accuracy and 61,163-site evaluation set define the metrics the alignment supports) — https://github.com/novoalab/EpiNano

## Examples

```
tombo resquiggle --overwrite existing ./fast5_wt yeast_transcriptome.fa --num-processes 8
```

## Evaluation signals

- Reproduced Tombo baseline recovers 220 of the 363 known m6A-modified sites (59.6% recovery) on the 61,163-site evaluation set
- Reproduced specificity matches 69.8% and overall accuracy matches 69% (vs 87% for EpiNano on the same site set); deviations flag a failed reproduction
- Resquiggle succeeds for the majority of reads — each read's raw signal must align against expected signal derived from its basecalls and the yeast reference transcriptome
- All site calls are restricted to the fixed 61,163-site evaluation set before computing recovery and classification metrics
- Predicted sites are compared against the ime4Δ canonical control, so signals at sites not shifted relative to the knockout should not be called modified

## Limitations

- Tombo's signal-level comparison achieves lower specificity (69.8%) and accuracy (69%) than EpiNano (87%) on the same 61,163-site set, so it serves as a baseline rather than the preferred detector
- Only 32–60% of known sites are recovered by the tested methods overall (EpiNano recovers 32% of known sites), so low recovery is expected rather than a pipeline error
- Direct RNA base-calling is 'error'-rich even in unmodified sequences, so a matched control (ime4Δ knockout) is required for reliable discrimination
- Results depend on base-caller version; EpiNano models, for example, are only accurate for data base-called with specific versions (e.g., Albacore 2.1.7 or Guppy 3.1.5)

## Evidence

- [other] Run Tombo resquiggle to re-align each read's raw current-intensity signal against the expected signal derived from its basecalls and the yeast reference transcriptome.: [evidence span withheld — anchor status: not_found]
- [discussion] Identify RNA modifications from base-called features without squiggle realignments or manipulation of raw current intensity data: "here we provide a novel strategy to identify RNA modifications from base-called features, without the need of squiggling realignments or manipulation of raw current intensity data sets."
- [discussion] The method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites.: "We find that the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites."
- [readme] direct RNA sequencing base-calling produces a significant amount of 'errors' in unmodified sequences. Therefore, to obtain higher confidence m6A-modified sites, we recommend to sequence both modified and unmodified datasets: [evidence span withheld — anchor status: external_doc]
- [readme] The SVM predictions will be only accurate if your data has been base-called using Albacore 2.1.7.: [evidence span withheld — anchor status: external_doc]

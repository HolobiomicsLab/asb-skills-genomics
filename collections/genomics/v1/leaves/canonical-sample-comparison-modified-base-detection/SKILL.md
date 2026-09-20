---
name: canonical-sample-comparison-modified-base-detection
description: Use when benchmarking a raw-signal (re-squiggle) modification caller
  against the EpiNano baseline on ONT direct RNA fast5 data, and a matched unmodified
  control strain (e.g. ime4Δ knockout) is available for canonical sample comparison.
when_to_use_negative:
- The goal is the base-called-feature method itself (EpiNano-Error/EpiNano-SVM from
  mismatch, deletion, insertion frequencies and per-base quality) rather than the
  signal-level baseline — use the EpiNano pipeline instead.
- No matched unmodified/canonical control (e.g. a knockout or demethylase-treated
  sample) exists, which the sample-comparison mode requires.
- Input is already a scored modification table or per-site metric CSV rather than
  raw fast5 signal data.
edam_operation: http://edamontology.org/operation_3200
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: Tombo
  role: Performs resquiggle re-alignment of raw current intensities and the canonical
    (sample-comparison) mode that calls shifted-signal modification sites using ime4Δ
    as control
- name: Oxford Nanopore Technologies (ONT)
  role: Platform providing the direct RNA raw fast5 files; ONT basecaller generates
    the per-read basecalls and signal offsets Tombo needs
- name: EpiNano
  role: Base-called-feature method that this Tombo baseline is compared against on
    the same 61,163-site set (87% vs 69% accuracy)
  repo: https://github.com/novoalab/EpiNano
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:72343dc065bbde0c48cb463823318b04f3893fda6cb8a78c31d96df37db6d903
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  - Tombo
  edam_operation: http://edamontology.org/operation_3200
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# canonical-sample-comparison-modified-base-detection

## Summary

Reproduces the Tombo baseline for m6A detection on yeast ONT direct RNA sequencing: raw current intensities are re-squiggled and compared between wild-type and the ime4Δ knockout (m6A-lacking) canonical control to call modified sites, which are then scored on the fixed 61,163-site evaluation set. This provides the signal-level comparator (69% accuracy, 59.6% recovery, 69.8% specificity) against which EpiNano's base-called-feature approach (87% accuracy) is judged.

## When to use

Use when benchmarking a raw-signal (re-squiggle) modification caller against the EpiNano baseline on ONT direct RNA fast5 data, and a matched unmodified control strain (e.g. ime4Δ knockout) is available for canonical sample comparison.

## When NOT to use

- The goal is the base-called-feature method itself (EpiNano-Error/EpiNano-SVM from mismatch, deletion, insertion frequencies and per-base quality) rather than the signal-level baseline — use the EpiNano pipeline instead.
- No matched unmodified/canonical control (e.g. a knockout or demethylase-treated sample) exists, which the sample-comparison mode requires.
- Input is already a scored modification table or per-site metric CSV rather than raw fast5 signal data.

## Inputs

- Yeast wild-type direct RNA raw fast5 files (deposited ONT dataset)
- Yeast ime4Δ knockout direct RNA raw fast5 files (canonical control, lacks m6A)
- Yeast transcriptome reference used for alignment
- Per-read ONT basecalls for the raw fast5 files
- Fixed 61,163-site evaluation set with ground truth of 363 known m6A-modified sites

## Outputs

- tombo_canonical_comparison_metrics.csv (recovery, specificity, accuracy against reported baseline: 220/363, 69.8%, 69%)
- Per-read re-squiggled signal alignments of raw current intensities to basecalls/reference
- Set of shifted-signal candidate modification sites restricted to the 61,163-site evaluation set

## How to apply

Download the deposited yeast wild-type and ime4Δ direct RNA raw fast5 files plus the yeast transcriptome reference, then base-call the fast5 with the ONT basecaller so Tombo has per-read basecalls and signal offsets. Run Tombo resquiggle to re-align each read's raw current-intensity signal against the expected signal from its basecalls and the reference transcriptome. Run Tombo's sample-comparison (canonical comparison) mode using the ime4Δ knockout, which lacks m6A, as the canonical control against wild-type to detect shifted-signal sites. Restrict detected sites to the fixed 61,163-site evaluation set used in the paper, then compute recovery and classification metrics against the 363 known m6A sites and write tombo_canonical_comparison_metrics.csv. A correct reproduction matches the reported baseline — 220 of 363 known sites recovered (59.6%), specificity 69.8%, accuracy 69% — with deviations flagging a failed reproduction.

## Related tools

- **Tombo** (Performs resquiggle re-alignment of raw current intensities and the canonical (sample-comparison) mode that calls shifted-signal modification sites using ime4Δ as control)
- **Oxford Nanopore Technologies (ONT)** (Platform providing the direct RNA raw fast5 files; ONT basecaller generates the per-read basecalls and signal offsets Tombo needs)
- **EpiNano** (Base-called-feature method that this Tombo baseline is compared against on the same 61,163-site set (87% vs 69% accuracy)) — https://github.com/novoalab/EpiNano

## Evaluation signals

- Reproduced recovery matches 220 of 363 known m6A sites (59.6%) on the yeast wild-type sample
- Reproduced specificity matches 69.8% on the fixed 61,163-site evaluation set
- Reproduced accuracy matches 69%, sitting below EpiNano's 87% on the same site set (lower accuracy than the base-called-feature method is expected)
- tombo_canonical_comparison_metrics.csv is produced and its metrics fall within tolerance of the reported baseline; deviations flag a failed reproduction
- Detection restricted to the same 61,163-site evaluation set used for the EpiNano comparison, ensuring apples-to-apples metrics

## Limitations

- Tombo's signal-based baseline recovers fewer known sites and has lower specificity (69.8%) and accuracy (69%) than EpiNano's base-called-feature approach (87% in vivo, 32% overall recovery of known sites).
- Both approaches are site-level, not single-molecule: the algorithm 'does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site'.
- The approaches cannot distinguish modification types (e.g., m1A vs m6A), and feature/error contributions vary across k-mers, so a matched unmodified control (here, the ime4Δ strain lacking m6A) is essential given the error-rich nature of direct RNA reads.

## Evidence

- [abstract] in vivo accuracy: "our method can identify m 6 A RNA modifications in vivo with an accuracy of 87%"
- [abstract] ime4 knockout validation: "these ‘errors’ are typically not observed in yeast ime4 -knockout strains, which lack m 6 A modifications"
- [discussion] site-level aggregation limit: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [readme] paired control rationale: [evidence span withheld — anchor status: external_doc]
- [readme] error-rich direct RNA reads: [evidence span withheld — anchor status: external_doc]

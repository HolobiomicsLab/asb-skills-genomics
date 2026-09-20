---
name: raw-fast5-signal-processing
description: Use when you have ONT direct RNA raw FAST5 files and need signal-level
  re-alignment (resquiggling) of current intensities to a reference transcriptome
  to detect m6A sites, e.g., to benchmark against a base-called-feature method like
  EpiNano.
when_to_use_negative:
- Your analysis plan is the EpiNano strategy, which explicitly identifies RNA modifications
  from base-called features 'without the need of squiggling realignments or manipulation
  of raw current intensity data sets'
- Your data is not ONT direct RNA raw signal data (e.g., you already have FASTQ-only
  base-called feature tables and no FAST5 signal)
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0622
tools:
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the raw FAST5 signal files and the
    basecaller used to generate per-read basecalls for resquiggling
- name: Tombo
  role: Performs resquiggle re-alignment of raw current-intensity signal and sample-comparison
    modification calling (ime4Δ canonical control vs WT)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:6c4aebb93c920cd3c75a2acb853aa68151ada9bb2b80ac75f652674af3b053f8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Oxford Nanopore Technologies (ONT)
  - Tombo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# raw-fast5-signal-processing

## Summary

Process ONT direct RNA raw FAST5 files by base-calling them and re-squiggling each read's raw current-intensity signal against an expected signal derived from basecalls and a reference transcriptome (as done with Tombo on yeast WT/ime4Δ data), providing a signal-level baseline for detecting RNA modifications.

## When to use

Use when you have ONT direct RNA raw FAST5 files and need signal-level re-alignment (resquiggling) of current intensities to a reference transcriptome to detect m6A sites, e.g., to benchmark against a base-called-feature method like EpiNano.

## When NOT to use

- Your analysis plan is the EpiNano strategy, which explicitly identifies RNA modifications from base-called features 'without the need of squiggling realignments or manipulation of raw current intensity data sets'
- Your data is not ONT direct RNA raw signal data (e.g., you already have FASTQ-only base-called feature tables and no FAST5 signal)

## Inputs

- Yeast wild-type and ime4Δ direct RNA raw FAST5 files (deposited dataset)
- Yeast transcriptome reference used for alignment
- Base-called FAST5s or FASTQ with per-read basecalls from the ONT basecaller

## Outputs

- Resquiggled per-read signal alignments against the yeast transcriptome
- Tombo shifted-signal site calls from canonical (ime4Δ vs WT) comparison
- Classification metrics on the 61,163-site evaluation set (tombo_canonical_comparison_metrics.csv)

## How to apply

Download the deposited wild-type and ime4Δ direct RNA FAST5 files plus the yeast transcriptome reference, then base-call the FAST5s with the ONT basecaller so per-read basecalls and signal offsets exist for Tombo. Run Tombo resquiggle to re-align each read's raw current-intensity signal against the expected signal derived from its basecalls and the yeast reference transcriptome. Then run Tombo's sample-comparison (canonical comparison) mode using the ime4Δ knockout — which lacks m6A — as the canonical control against the WT sample to detect shifted-signal sites. Restrict detected sites to the fixed 61,163-site evaluation set before computing recovery, specificity, and accuracy against known m6A ground truth (363 known sites). This contrasts with the EpiNano approach, which deliberately avoids squiggle realignment and works only on base-called features (mismatches, deletions, per-base qualities).

## Related tools

- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the raw FAST5 signal files and the basecaller used to generate per-read basecalls for resquiggling)
- **Tombo** (Performs resquiggle re-alignment of raw current-intensity signal and sample-comparison modification calling (ime4Δ canonical control vs WT))

## Examples

```
tombo resquiggle ./wt_fast5 ./yeast_transcriptome.fa
```

## Evaluation signals

- Reproduced recovery matches the baseline: 220 of 363 known m6A sites recovered (59.6% recovery)
- Reproduced specificity is 69.8% and overall accuracy is 69% on the fixed 61,163-site evaluation set
- Deviation from the reported 220/363 recovery, 69.8% specificity, or 69% accuracy values flags a failed reproduction
- All detected sites are restricted to the same 61,163-site set used in the paper before metric computation

## Limitations

- Tombo's re-squiggle comparison achieves lower specificity (69.8%) and lower overall accuracy (69%) than EpiNano's 87% in vivo accuracy on the same evaluation set
- Only 220 of 363 (59.6%) known sites are recovered, so many true m6A sites are missed relative to EpiNano's reported recovery rates
- Requires raw FAST5 signal and basecalls; the article's proposed method avoids 'squiggling realignments or manipulation of raw current intensity data sets' precisely because of this dependency

## Evidence

- [other] resquiggle re-alignment rationale: [evidence span withheld — anchor status: not_found]
- [intro] hypothesis for signal-level effects: [evidence span withheld — anchor status: not_found]
- [discussion] EpiNano avoids raw signal manipulation: "here we provide a novel strategy to identify RNA modifications from base-called features, without the need of squiggling realignments or manipulation of raw current intensity data sets."
- [readme] current intensity extraction from FAST5: [evidence span withheld — anchor status: external_doc]

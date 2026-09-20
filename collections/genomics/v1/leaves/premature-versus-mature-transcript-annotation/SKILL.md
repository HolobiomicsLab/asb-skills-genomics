---
name: premature-versus-mature-transcript-annotation
description: Use when native Nanopore dRNA-seq reads from a subcellular fraction (e.g.
  chromatin RNA) must be labeled per-read as premature or mature, or when a fraction's
  premature-read fraction must be reproduced/validated against a published value (e.g.
  ~83% for nano-COP K562 chromatin).
when_to_use_negative:
- Input is not native-RNA Nanopore data (e.g. short-read or cDNA-amplified libraries),
  since nano-ID modification-probability scores depend on native-RNA signals
- Reads come from metabolically labeled, fractionated Nanodynamo samples where classification
  of nascent reads is handled by the nano-ID rate model rather than standalone premature/mature
  annotation
- The goal is estimating kinetic rates (synthesis/processing/degradation) rather than
  per-read premature/mature annotation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Nanodynamo
  role: Provides the fractionsReadsClassification.R workflow and core R functions
    for classifying nascent and premature read fractions
  repo: https://github.com/mfurla/Nanodynamo
- name: nano-ID model
  role: Machine-learning classifier whose labeled/unlabeled modification-probability
    training distributions define the premature/mature read thresholding
  repo: https://github.com/birdumbrella/nano-ID
- name: R
  role: Language in which the classification script, per-read labeling, and premature-fraction
    aggregation are executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:229dff08230941563a02eb7514c128dda09d8a4ed0d12d5a1d240d58757ee4a6
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  repo_url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - R
  - nano-ID model
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Premature versus mature transcript annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies individual native Nanopore dRNA-seq reads as premature or mature transcripts by thresholding nano-ID modification-probability scores against labeled/unlabeled training distributions (fractionsReadsClassification-style analysis), enabling quantification of premature-read enrichment in subcellular RNA fractions.

## When to use

Use when native Nanopore dRNA-seq reads from a subcellular fraction (e.g. chromatin RNA) must be labeled per-read as premature or mature, or when a fraction's premature-read fraction must be reproduced/validated against a published value (e.g. ~83% for nano-COP K562 chromatin).

## When NOT to use

- Input is not native-RNA Nanopore data (e.g. short-read or cDNA-amplified libraries), since nano-ID modification-probability scores depend on native-RNA signals
- Reads come from metabolically labeled, fractionated Nanodynamo samples where classification of nascent reads is handled by the nano-ID rate model rather than standalone premature/mature annotation
- The goal is estimating kinetic rates (synthesis/processing/degradation) rather than per-read premature/mature annotation

## Inputs

- Native Nanopore dRNA-seq reads from a subcellular fraction (e.g. nano-COP K562 chromatin RNA, GEO GSM4663623)
- Trained nano-ID classifier model
- nano-ID training modification-probability data: nanoID.modification.probabilities.labeled.RData and nanoID.modification.probabilities.unlabeled.RData

## Outputs

- Per-read premature/mature classification labels
- Summary premature-read fraction per sample/fraction (e.g. premature_fraction.csv)

## How to apply

Load the native Nanopore dRNA-seq reads for the target sample (e.g. GEO GSM4663623, nano-COP K562 chromatin RNA) into R. Load the trained nano-ID classifier together with the training modification-probability distributions shipped with Nanodynamo (nanoID.modification.probabilities.labeled.RData and nanoID.modification.probabilities.unlabeled.RData). Run the fractionsReadsClassification-style assignment, labeling each read premature or mature by comparing its modification-probability score against the labeled and unlabeled training distributions. Aggregate per-read labels to compute the fraction of premature reads in the fraction of interest, then write per-read labels plus the summary premature fraction (e.g. to premature_fraction.csv). The rationale is that chromatin-associated RNA should be depleted of mature molecules, so premature enrichment is the expected signature for fraction-quality classification.

## Related tools

- **Nanodynamo** (Provides the fractionsReadsClassification.R workflow and core R functions for classifying nascent and premature read fractions) — https://github.com/mfurla/Nanodynamo
- **nano-ID model** (Machine-learning classifier whose labeled/unlabeled modification-probability training distributions define the premature/mature read thresholding) — https://github.com/birdumbrella/nano-ID
- **R** (Language in which the classification script, per-read labeling, and premature-fraction aggregation are executed)

## Examples

```
Rscript Scripts/fractionsReadsClassification.R
```

## Evaluation signals

- Premature fraction for the nano-COP K562 chromatin dRNA-seq sample (GSM4663623) should match the reported ~83% premature reads
- Every input read receives exactly one premature or mature label (no unclassified reads silently dropped)
- Per-read labels and the summary premature fraction are written to premature_fraction.csv with consistent counts
- Classification is reproducible when re-run with the same labeled/unlabeled nano-ID training probability data
- Chromatin fraction should show higher premature fraction than cytoplasmic/polysomal fractions if comparable samples are processed

## Limitations

- The classifier is trained on nano-ID labeled/unlabeled distributions; reads from platforms or sample preparations outside native-RNA Nanopore dRNA-seq may score unreliably
- The classification is a thresholding of modification-probability scores against training distributions, so boundary-case reads near the labeled/unlabeled distributions' overlap may be mislabeled
- Validation anchors on a single published value (~83% premature for the nano-COP K562 chromatin sample); other samples lack such explicit acceptance thresholds

## Evidence

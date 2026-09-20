---
name: chromatin-rna-fraction-enrichment-quantification
description: Use when a native Nanopore dRNA-seq sample (e.g., chromatin-fraction
  RNA from K562 cells, GEO GSM4663623) must be split into premature vs mature reads
  to verify subcellular-fraction enrichment of unprocessed RNA.
when_to_use_negative:
- Input is short-read RNA-seq or cDNA-based sequencing rather than native RNA Nanopore
  dRNA-seq — the classification depends on modification-probability scores derived
  from native RNA signals.
- The goal is estimating RNA kinetic rates (nanoIDMain.R / nanoIDResults.rds) rather
  than characterizing nascent and premature read fractions; use the modeling scripts
  instead.
- The sample is from a cytoplasmic or polysomal fraction where mature reads, not chromatin-associated
  premature RNA, are the expected dominant class and the ~83% enrichment benchmark
  does not apply.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Nanodynamo
  role: Provides the fractionsReadsClassification.R-style procedure that assigns each
    dRNA-seq read a premature or mature label by thresholding modification-probability
    scores
  repo: github.com/mfurla/Nanodynamo
- name: R
  role: Environment in which the classification of chromatin-fraction reads and aggregation
    to a premature fraction are run
- name: nano-ID model
  role: Trained machine-learning classifier whose labeled/unlabeled modification-probability
    distributions define the premature vs mature decision boundary
  repo: github.com/birdumbrella/nano-ID
- name: Nanopore sequencing
  role: Data-acquisition modality producing the native RNA reads (nano-COP dRNA-seq
    of K562 chromatin RNA) that are classified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51917-2
    title: Nanodynamo quantifies subcellular RNA dynamics revealing extensive coupling
      between steps of the RNA life cycle
  source_content_hash: sha256:2dea39500700c43f36fb32a4d8e318430f8e0b4e533930ab4bc598469c5ab0f1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mfurla/Nanodynamo
  tools:
  - Nanodynamo
  - Nanopore sequencing
  - R
  - nano-ID model
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# chromatin-rna-fraction-enrichment-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies premature vs mature read fractions in a chromatin-fraction native Nanopore dRNA-seq sample by applying Nanodynamo's fractionsReadsClassification-style labeling, validating chromatin premature-read enrichment against the reported ~83% for nano-COP K562.

## When to use

Use when a native Nanopore dRNA-seq sample (e.g., chromatin-fraction RNA from K562 cells, GEO GSM4663623) must be split into premature vs mature reads to verify subcellular-fraction enrichment of unprocessed RNA.

## When NOT to use

- Input is short-read RNA-seq or cDNA-based sequencing rather than native RNA Nanopore dRNA-seq — the classification depends on modification-probability scores derived from native RNA signals.
- The goal is estimating RNA kinetic rates (nanoIDMain.R / nanoIDResults.rds) rather than characterizing nascent and premature read fractions; use the modeling scripts instead.
- The sample is from a cytoplasmic or polysomal fraction where mature reads, not chromatin-associated premature RNA, are the expected dominant class and the ~83% enrichment benchmark does not apply.

## Inputs

- nano-COP native Nanopore dRNA-seq reads for the K562 chromatin RNA sample (GEO GSM4663623)
- nanoID.modification.probabilities.labeled.RData (nano-ID training modification-probability data)
- nanoID.modification.probabilities.unlabeled.RData (nano-ID training modification-probability data)
- trained nano-ID classifier (task_001 output)

## Outputs

- Per-read premature/mature classification labels
- premature_fraction.csv containing per-read labels and the summary premature read fraction (~83% expected for the chromatin sample)

## How to apply

Load the nano-COP native Nanopore dRNA-seq reads for the K562 chromatin RNA sample (GEO GSM4663623) into R, together with the trained nano-ID classifier and the nanoID.modification.probabilities.labeled.RData / nanoID.modification.probabilities.unlabeled.RData distributions used during training. Run the fractionsReadsClassification-style procedure, assigning each read a premature or mature label by thresholding its modification-probability scores against the labeled and unlabeled training distributions. Aggregate the per-read labels to compute the fraction of premature reads in the chromatin fraction and write per-read labels plus the summary premature fraction to premature_fraction.csv. The key decision point is the threshold placement between the two modification-probability distributions, which determines each read's class; the chromatin fraction should be dominated by premature reads.

## Related tools

- **Nanodynamo** (Provides the fractionsReadsClassification.R-style procedure that assigns each dRNA-seq read a premature or mature label by thresholding modification-probability scores) — github.com/mfurla/Nanodynamo
- **R** (Environment in which the classification of chromatin-fraction reads and aggregation to a premature fraction are run)
- **nano-ID model** (Trained machine-learning classifier whose labeled/unlabeled modification-probability distributions define the premature vs mature decision boundary) — github.com/birdumbrella/nano-ID
- **Nanopore sequencing** (Data-acquisition modality producing the native RNA reads (nano-COP dRNA-seq of K562 chromatin RNA) that are classified)

## Examples

```
Rscript fractionsReadsClassification.R
```

## Evaluation signals

- The computed premature fraction for the nano-COP K562 chromatin sample should match the paper's reported value of approximately 83% premature reads.
- Every read in the dRNA-seq input receives exactly one label (premature or mature) after thresholding against the labeled and unlabeled modification-probability distributions.
- premature_fraction.csv contains both per-read labels and a single summary premature-fraction value consistent with the aggregate of those labels.
- The chromatin fraction shows clear premature-read enrichment relative to fractions expected to contain mature RNA, confirming the expected biological enrichment.

## Limitations

- The validation benchmark (~83% premature) is specific to the nano-COP K562 chromatin sample; other fractions or cell types will yield different expected fractions.
- Classification accuracy depends on the nano-ID training distributions (nanoID.modification.probabilities.labeled/unlabeled.RData); the same article separately provides nanoIDAccuracy.R to characterize nano-ID accuracy.
- The classification requires the modification-probability scores from native RNA Nanopore data; reads lacking these scores cannot be assigned a class.

## Evidence

- [intro] Nanodynamo relies on RNA metabolic labelling and Nanopore sequencing of native RNA for the profiling of transcripts from cellular fractions and polysomes: "Nanodynamo relies on RNA metabolic labelling and Nanopore sequencing of native RNA for the profiling of transcripts from cellular fractions and polysomes"
- [methods] nano-ID training modification-probability data: "Data produced during nano-ID training (nanoID.modification.probabilities.labeled.RData,nanoID.modification.probabilities.unlabeled.RData"
- [readme] fractionsReadsClassification.R script: "An R script to characterize nascent and premature reads fractions (fractionsReadsClassification.R),"

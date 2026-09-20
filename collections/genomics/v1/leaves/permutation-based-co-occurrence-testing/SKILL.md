---
name: permutation-based-co-occurrence-testing
description: Use when per-read modification calls (e.g. CHEUI-solo Model 1 m6A/m5C
  probabilities) must be tested for co-occurrence within single reads against a permuted
  null expectation.
when_to_use_negative:
- Input is site-level stoichiometry or differential methylation output only, with
  no per-read (single-molecule) modification state available — read-level co-occurrence
  cannot be computed from aggregated site calls
- Testing co-occurrence within rRNA or other densely modified RNAs, where the article
  notes signal interference at distances <5 nt is exacerbated
- Data produced by differential-comparison tools (Nanocompore, Xpore, DRUMMER, ELIGOS,
  Epinano) that compare conditions rather than calling modifications in individual
  reads from a single sample
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0099
tools:
- name: CHEUI
  role: Provides the per-read (Model 1) m6A/m5C modification states and the Model
    2 site probabilities (>0.9999) that define the modified site pairs tested for
    co-occurrence
  repo: https://github.com/comprna/CHEUI
- name: R2Dtool
  role: Annotates transcript-specific methylation calls and computes relative distances
    between sites on transcripts (cheui_to_bed.sh, R2_annotate, R2_lift), enabling
    the 1–15 nt pair selection
  repo: https://github.com/comprna/R2Dtool
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47953-7
    title: Prediction of m6A and m5C at single-molecule resolution reveals a transcriptome-wide
      co-occurrence of RNA modifications
  source_content_hash: sha256:7c25108aeeb86eea75aa7d891894060eb1ba4c3e3d5d5798dd8f673d41fd158a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/comprna/CHEUI
  tools:
  - CHEUI
  - R2Dtool
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# permutation-based-co-occurrence-testing

## Summary

Tests whether two RNA modifications (m6A and m5C) co-occur within individual nanopore reads more often than chance, by comparing observed per-read co-occurrence of same-state site pairs (1–15 nt apart) against an expected rate built from 1000 random inter-transcript site pairs, plus a Fisher's exact test on transcript-level contingency classes.

## When to use

Use when per-read modification calls (e.g. CHEUI-solo Model 1 m6A/m5C probabilities) must be tested for co-occurrence within single reads against a permuted null expectation.

## When NOT to use

- Input is site-level stoichiometry or differential methylation output only, with no per-read (single-molecule) modification state available — read-level co-occurrence cannot be computed from aggregated site calls
- Testing co-occurrence within rRNA or other densely modified RNAs, where the article notes signal interference at distances <5 nt is exacerbated
- Data produced by differential-comparison tools (Nanocompore, Xpore, DRUMMER, ELIGOS, Epinano) that compare conditions rather than calling modifications in individual reads from a single sample

## Inputs

- CHEUI-solo Model 1 per-read m6A prediction tables (HEK293 WT pooled replicates; mouse cortex E12/E15/E18)
- CHEUI-solo Model 1 per-read m5C prediction tables for the same samples
- CHEUI-solo Model 2 transcriptomic-site probabilities (threshold >0.9999) for m6A and m5C
- Transcriptome annotation (Gencode v38 for human GRCh38; Ensembl v104 for mouse GRCm39) for protein-coding filtering and site distances

## Outputs

- Distance-resolved observed-vs-expected co-occurrence values for 1–15 nt site pairs
- Two-tailed Fisher's exact test p-value for transcript-level co-occurrence enrichment (expected ~1.76E-06)
- CCDC102A single-transcript co-occurrence statistic (~78%)
- Combined results written to co_occurrence_stats.tsv

## How to apply

Load CHEUI-solo Model 1 per-read m6A and m5C predictions for the pooled HEK293 WT replicates and mouse cortex E12/E15/E18 samples, keeping only protein-coding transcripts with at least two tested sites covered by >=20 reads and excluding rRNA reads/sites. Select pairs of significantly modified sites (CHEUI Model 2 probability >0.9999) separated by 1–15 nt within the same transcript. Compute observed read-level co-occurrence per pair as reads carrying the same modification state at both sites divided by reads covering both sites. Estimate the expected background by sampling 1000 random pairs of modified sites located in different transcripts and computing the same per-read statistic, which yields distance-resolved observed-vs-expected values. In parallel, partition all tested mRNAs into four contingency classes (both m6A and m5C sites, only m6A, only m5C, neither) and run a two-tailed Fisher's exact test to assess transcript-level enrichment. Acceptance requires observed co-occurrence to exceed the permuted expectation at 5–15 nt distances, a significant Fisher result (paper reports p = 1.76E-06), and a CCDC102A pair co-occurrence near ~78%.

## Related tools

- **CHEUI** (Provides the per-read (Model 1) m6A/m5C modification states and the Model 2 site probabilities (>0.9999) that define the modified site pairs tested for co-occurrence) — https://github.com/comprna/CHEUI
- **R2Dtool** (Annotates transcript-specific methylation calls and computes relative distances between sites on transcripts (cheui_to_bed.sh, R2_annotate, R2_lift), enabling the 1–15 nt pair selection) — https://github.com/comprna/R2Dtool

## Examples

```
expected = [pair_coocc(*rng.choice(other_tx_modified_sites, 2, replace=False)) for _ in range(1000)]  # permuted inter-transcript pairs
```

## Evaluation signals

- Observed read-level co-occurrence must exceed the 1000-permutation expected background at 5–15 nt site-pair distances
- Two-tailed Fisher's exact test on the four contingency classes must be significant (paper reports p = 1.76E-06 for transcript-level enrichment)
- CCDC102A transcript pair co-occurrence should be near the reported ~78%
- Expected co-occurrence distribution must derive from 1000 random pairs drawn from different transcripts, not the same transcripts
- Contingency classes (both, only m6A, only m5C, neither) should partition all tested protein-coding mRNAs with >=2 sites at >=20 reads coverage

## Limitations

- CHEUI used very strict cut-offs on cell transcriptome sites to maintain a low expected FDR, which reduces sensitivity — some truly modified sites fall below threshold and never enter the pair set
- Signal interference at distances <5 nt would be exacerbated in highly and densely modified molecules such as rRNA, so very short-distance pairs are unreliable
- The mechanisms underlying the observed read-level and transcript-level co-occurrence remain to be elucidated, so significance does not imply a defined biological mechanism
- Modified sites detected only from nanopore reads require independent validation experiments

## Evidence

- [abstract] CHEUI's capability to identify two modification types in the same sample reveals a co-occurrence of m6A and m5C in individual mRNAs in cell line and tissue transcriptomes.: "CHEUI's capability to identify two modification types in the same sample reveals a co-occurrence of m6A and m5C in individual mRNAs in cell line and tissue transcriptomes."
- [methods] we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites: "we shuffled all read signals across all transcriptomic sites, maintaining the same number of transcriptomic sites"
- [discussion] The mechanisms underlying the identified co-occurrence of modifications in reads and across transcripts remain to be elucidated.: "The mechanisms underlying the identified co-occurrence of modifications in reads and across transcripts remain to be elucidated."
- [readme] CHEUI can be used to detect m6A and m5C modifications at single read resolution and to identified modified sites and their stoichiometry: "CHEUI can be used to detect m6A and m5C modifications at single read resolution and to identified modified sites and their stoichiometry"

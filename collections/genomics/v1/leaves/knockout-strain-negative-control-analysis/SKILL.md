---
name: knockout-strain-negative-control-analysis
description: Use when validating m6A site predictions from ONT direct RNA reads by
  checking that base-calling error signals observed in wild-type are absent in a modification-deficient
  (ime4Δ) knockout/control strain.
when_to_use_negative:
- Input is a single sample with no matched knockout/knockdown or demethylase-treated
  control — EpiNano-Error pairwise mode and the negative-control logic require WT-vs-KO
  comparison
- The analysis target is single-molecule (per-read) modification calling — EpiNano
  aggregates all reads per site and does not predict modifications in individual RNA
  molecules
- The modification of interest is not expected to alter base-calling features, or
  a pretrained SVM model does not exist for it and cannot be trained from matched
  controls
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EpiNano
  role: Extracts per-k-mer base-calling error features (mismatch, deletion, insertion,
    per-base quality) from WT and knockout direct RNA reads and scores candidate m6A
    sites with pretrained or trained SVM models; pairwise (EpiNano-Error) mode uses
    WT-vs-KO error differences directly
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Platform used to generate the direct RNA sequencing reads for the SAy841 wild-type
    and SAy966 ime4Δ strains; the base-called features (qualities and errors) that
    form the modification signal come from ONT base-callers
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:358ff1da8b5a7d8c9b4c1a930e971dae38368917b34252a8b7707a76c6de481d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# knockout-strain-negative-control-analysis

## Summary

Validates nanopore direct RNA sequencing m6A calls by comparing base-calling 'error' features between a wild-type yeast strain (SAy841) and a modification-deficient ime4Δ knockout (SAy966). Because m6A introduces systematic errors (altered per-base qualities, mismatch and deletion frequencies), true modification signals should be present in WT but typically absent in the knockout strain.

## When to use

Use when validating m6A site predictions from ONT direct RNA reads by checking that base-calling error signals observed in wild-type are absent in a modification-deficient (ime4Δ) knockout/control strain.

## When NOT to use

- Input is a single sample with no matched knockout/knockdown or demethylase-treated control — EpiNano-Error pairwise mode and the negative-control logic require WT-vs-KO comparison
- The analysis target is single-molecule (per-read) modification calling — EpiNano aggregates all reads per site and does not predict modifications in individual RNA molecules
- The modification of interest is not expected to alter base-calling features, or a pretrained SVM model does not exist for it and cannot be trained from matched controls

## Inputs

- ONT direct RNA sequencing reads (FASTQ/FAST5) for S. cerevisiae wild-type strain SAy841, base-called on the ONT platform
- Matched ONT direct RNA sequencing reads for ime4Δ knockout strain SAy966 (m6A-deficient negative control)
- Annotated RRACH motif adenosine site list (61,163 candidate sites in yeast)
- m6A-Seq reference set of 363 known m6A-modified sites for recovery assessment

## Outputs

- Per-site base-calling error feature table (per-base qualities, mismatch, deletion, insertion frequencies) for WT and knockout
- Delta-feature comparisons (WT minus knockout error differences) at candidate sites
- Confidence-scored list of m6A-modified RRACH sites filtered by absence of signal in the ime4Δ control
- In vivo accuracy/specificity/recovery metrics (e.g., 87.8% accuracy, 89% specificity, 32% recovery = 117/363 sites)

## How to apply

Sequence matched wild-type and modification-deficient samples on the same ONT platform (S. cerevisiae SAy841 WT and ime4Δ SAy966), and base-call both with the same base-caller version, since EpiNano-SVM predictions are base-calling-algorithm dependent (pretrained models target Albacore 2.1.7 or Guppy 3.1.5). Run EpiNano feature extraction on both datasets to compute per-site per-base qualities, mismatch frequencies, deletion frequencies, and related features, aggregating over all reads mapping to each candidate site. Restrict analysis to RRACH motif adenosine sites (e.g., the 61,163 RRACH sites in yeast, of which 363 are m6A-annotated in the m6A-Seq reference). Optionally compute delta-features (differences in error metrics between WT and knockout), which the README reports have improved performance, since direct RNA base-calling is inherently error-rich even on unmodified transcripts. Confirm that the error-based signals at candidate sites are typically absent in the ime4Δ strain, which lacks m6A modifications, and use the WT-vs-KO difference as the negative-control filter before scoring sites with the EpiNano SVM.

## Related tools

- **EpiNano** (Extracts per-k-mer base-calling error features (mismatch, deletion, insertion, per-base quality) from WT and knockout direct RNA reads and scores candidate m6A sites with pretrained or trained SVM models; pairwise (EpiNano-Error) mode uses WT-vs-KO error differences directly) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Platform used to generate the direct RNA sequencing reads for the SAy841 wild-type and SAy966 ime4Δ strains; the base-called features (qualities and errors) that form the modification signal come from ONT base-callers)

## Evaluation signals

- Reproduce the reported in vivo benchmark: overall accuracy ~87.8% (paper cites ~87%), 89% specificity, and recovery of 117/363 (32%) of known m6A-annotated RRACH sites
- Confirm that the base-calling error signatures at called sites in wild-type are typically absent in the ime4Δ strain, which lacks m6A modifications
- Verify candidate sites are restricted to RRACH motif adenosines (61,163 sites in yeast, 363 annotated as modified)
- Check that WT and control samples were base-called with the same, model-compatible base-caller (Albacore 2.1.7 or Guppy 3.1.5) since SVM predictions are base-calling-algorithm dependent
- Delta-features (difference in mismatch/error metrics between samples) should be prioritized over absolute error frequencies, as the README reports improved performance

## Limitations

- Only 32% of known modification sites are recovered by the method, so negative-control filtering trades sensitivity for confidence
- The algorithm aggregates information across all reads mapping to a site and cannot call modifications in individual RNA molecules; single-read detection is left to future work
- The algorithm does not distinguish between different types of RNA modifications (e.g., m1A vs m6A)
- Direct RNA sequencing base-calling produces substantial 'errors' even in unmodified sequences, so without a matched control the false-positive rate is high; SVM model accuracy is also dependent on the base-caller version and modification stoichiometry

## Evidence

- [abstract] errors absent in knockout: "these ‘errors’ are typically not observed in yeast ime4 -knockout strains, which lack m 6 A modifications"
- [readme] WT-vs-KO control recommendation: [evidence span withheld — anchor status: external_doc]
- [discussion] accuracy and recovery benchmark: "We find that the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites."
- [discussion] no per-read resolution: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [discussion] error forms detected: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."

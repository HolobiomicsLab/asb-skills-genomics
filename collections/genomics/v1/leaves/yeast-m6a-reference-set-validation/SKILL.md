---
name: yeast-m6a-reference-set-validation
description: Use when predicted RNA modification sites from ONT direct RNA reads in
  S. cerevisiae need benchmarking against the m6A-Seq annotation set (363 modified
  RRACH sites) and an ime4Δ negative control.
when_to_use_negative:
- The organism or modification is not yeast m6A — the 363-site m6A-Seq reference set
  and ime4Δ knockout are yeast-specific benchmarks
- Per-read (single-molecule) modification calling is required — EpiNano aggregates
  information from all reads mapping to a site and has no per-read resolution
- The goal is to distinguish between modification types (e.g., m1A vs m6A) — the algorithm
  cannot discriminate between different modifications
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: EpiNano
  role: Extracts per-site base-calling error features (per-base qualities, mismatch,
    deletion frequencies) and applies the trained SVM to score RRACH candidate sites
    for m6A
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Direct RNA sequencing platform producing the native RNA reads base-called
    for feature extraction
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:dffa1c989d0faca2f5386ecba758c2663304f6553d7070db20367b75d6dd0663
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# yeast-m6a-reference-set-validation

## Summary

Validates computationally predicted m6A sites in yeast direct RNA sequencing data by restricting scoring to RRACH motif adenosines and comparing predictions against the m6A-Seq reference set and ime4Δ knockout controls, yielding accuracy, recovery, and specificity metrics.

## When to use

Use when predicted RNA modification sites from ONT direct RNA reads in S. cerevisiae need benchmarking against the m6A-Seq annotation set (363 modified RRACH sites) and an ime4Δ negative control.

## When NOT to use

- The organism or modification is not yeast m6A — the 363-site m6A-Seq reference set and ime4Δ knockout are yeast-specific benchmarks
- Per-read (single-molecule) modification calling is required — EpiNano aggregates information from all reads mapping to a site and has no per-read resolution
- The goal is to distinguish between modification types (e.g., m1A vs m6A) — the algorithm cannot discriminate between different modifications

## Inputs

- S. cerevisiae wild-type (SAy841) ONT direct RNA base-called reads (FASTQ/FAST5)
- S. cerevisiae ime4Δ knockout (SAy966) ONT direct RNA base-called reads
- Per-site EpiNano feature table (per-base qualities, mismatch frequencies, deletion frequencies) aggregated over all reads per site
- m6A-Seq reference set of 363 annotated m6A-modified RRACH sites in yeast

## Outputs

- Classified list of m6A-modified candidate RRACH sites with modification scores
- Benchmark metrics: overall in vivo accuracy (~87.8%), recovery of known sites (117/363, ~32%), and specificity (~89%)
- Confirmation that error signals are typically absent in the ime4Δ strain

## How to apply

Restrict candidate scoring to the 61,163 RRACH motif adenosine sites in the yeast transcriptome, of which 363 are annotated as m6A-modified in the m6A-Seq reference set. Compare per-site EpiNano SVM scores for the wild-type strain (SAy841) against the ime4Δ knockout (SAy966), since error-based signals (altered per-base qualities, mismatch and deletion frequencies) should be absent in the knockout that lacks the m6A methyltransferase. Compute the fraction of correctly classified sites to obtain overall accuracy (~87.8% expected), the fraction of the 363 known sites recovered (117/363, ~32%), and specificity (~89%). Accept the validation only when reproduced accuracy reaches ~87% and recovery ~32%, matching the discussion-section metrics.

## Related tools

- **EpiNano** (Extracts per-site base-calling error features (per-base qualities, mismatch, deletion frequencies) and applies the trained SVM to score RRACH candidate sites for m6A) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Direct RNA sequencing platform producing the native RNA reads base-called for feature extraction)

## Examples

```
python EpiNano_SVM.py --input yeast_say841.features.csv --model m6A_pretrained --motif RRACH --reference m6Aseq_363_sites.bed
```

## Evaluation signals

- Reproduced in vivo accuracy reaches ~87% (reported 87.8%) against the m6A-Seq reference set
- Overall recovery of the 363 known m6A-modified RRACH sites reaches ~32% (117/363)
- Specificity of classified sites is ~89%
- Error-based signals (altered qualities, mismatch/deletion frequencies) are typically absent at scored sites in the ime4Δ knockout strain
- Scoring is confined to the 61,163 RRACH motif adenosine sites rather than all adenosines

## Limitations

- Only 32% of known modification sites are recovered by the method, so many true m6A sites go undetected
- The current algorithm does not predict modifications in individual RNA molecules but aggregates information from all reads mapping to a specific site
- The algorithm does not distinguish between different types of RNA modifications (e.g., m1A vs m6A)
- Feature contributions vary across k-mers, so k-mer-specific training is needed to improve accuracy and reduce false positives

## Evidence

- [discussion] Method accuracy: 90% in vitro, 87% in vivo, with overall recovery of 32% of known sites: "We find that the method accurately detects modifications both in vitro (90% accuracy) and in vivo (87% accuracy), with an overall recovery of 32% of known sites."
- [abstract] In vivo yeast validation against ime4 knockout: "validate our method by showing that these ‘errors’ are typically not observed in yeast ime4 -knockout strains"
- [discussion] Feature aggregation rather than per-read prediction: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [discussion] Errors detected as altered base-called features at the modified site: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] EpiNano is a site-level, error-based tool; recommend paired WT/KO mode: [evidence span withheld — anchor status: external_doc]

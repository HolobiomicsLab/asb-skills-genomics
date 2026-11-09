---
name: base-calling-error-feature-extraction
description: Use when analyzing ONT direct RNA FASTQ/BAM data to build modification-detection
  features from base-called errors (quality, mismatch, deletion, insertion) at candidate
  motif sites, rather than from raw current signals.
when_to_use_negative:
- You already possess an extracted feature table (quality/error frequencies) — proceed
  directly to site scoring/mixture analysis instead of re-extracting.
- You need per-read (single-molecule) modification calls — EpiNano explicitly does
  not have per-read resolution and aggregates across all reads mapping to a site.
- Data are raw squiggle/current-intensity signals without base-calling — this skill
  works on base-called features only and avoids squiggle realignment.
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3524
tools:
- name: EpiNano
  role: Feature extractor producing per-base/per-kmer base-calling error tables; also
    provides pre-trained m6A SVM models
  repo: https://github.com/novoalab/EpiNano
- name: ONT platform
  role: Direct RNA sequencing platform whose base-called reads supply the quality
    and error features
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:0c3d96ca54310bcb307466fe6d4fbee8be5514b17a88f78062639d5f2bbd686a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - ONT platform
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# base-calling-error-feature-extraction

## Summary

Extract per-base/per-kmer base-calling 'error' features (per-base quality scores, mismatch frequency, deletion frequency, insertion frequency, sumErr) from Oxford Nanopore direct RNA sequencing base-called reads (FASTQ/FAST5) to serve as input features for RNA modification prediction with EpiNano, without squiggle realignment or raw current-intensity manipulation.

## When to use

Use when analyzing ONT direct RNA FASTQ/BAM data to build modification-detection features from base-called errors (quality, mismatch, deletion, insertion) at candidate motif sites, rather than from raw current signals.

## When NOT to use

- You already possess an extracted feature table (quality/error frequencies) — proceed directly to site scoring/mixture analysis instead of re-extracting.
- You need per-read (single-molecule) modification calls — EpiNano explicitly does not have per-read resolution and aggregates across all reads mapping to a site.
- Data are raw squiggle/current-intensity signals without base-calling — this skill works on base-called features only and avoids squiggle realignment.

## Inputs

- Base-called ONT direct RNA reads (FASTQ) from modified and unmodified/control samples
- Optionally Albacore FAST5 files (only for EpiNano 1.0 current-intensity extraction)
- Reference transcriptome/alignment for assigning reads to candidate m6A DRACH motif sites

## Outputs

- Per-base and per-kmer feature tables (quality, mismatch, deletion, insertion frequencies, sumErr)
- Per-site aggregated feature summaries for candidate m6A motif sites
- SVM-ready feature matrices for EpiNano-Error or EpiNano-SVM scoring

## How to apply

Run EpiNano's feature-extraction modules on base-called direct RNA reads to obtain per-base and per-kmer tables of base quality scores, mismatch frequency, deletion frequency, insertion frequency and sumErr; note that EpiNano 1.1+ drops current intensity since it was not used in the final trained models. Aggregate read-level features into per-site summaries for every candidate m6A motif site, because the algorithm scores sites using information from all reads mapping to the site rather than individual molecules. For pairwise analysis prefer EpiNano-Error mode (base-caller independent), computing differences in error patterns between matched samples (e.g. WT vs KO); for single-sample scoring use EpiNano-SVM with models matched to the base-caller version (Albacore 2.1.7 for EpiNano 1.0/1.1, Guppy 3.1.5 for EpiNano 1.2). Because direct RNA base-calling is error-rich even on unmodified transcripts, always couple with a control (demethylase-treated, KO or KD) and select modified sites by the largest between-sample differences in error features.

## Related tools

- **EpiNano** (Feature extractor producing per-base/per-kmer base-calling error tables; also provides pre-trained m6A SVM models) — https://github.com/novoalab/EpiNano
- **ONT platform** (Direct RNA sequencing platform whose base-called reads supply the quality and error features)

## Examples

```
git clone https://github.com/novoalab/EpiNano; python EpiNano/Epinano_per_base_error_rate.py -i aln.bam -r ref.fa -o features/per_base.csv
```

## Evaluation signals

- Feature table contains the expected columns: per-base quality, mismatch frequency, deletion frequency, insertion frequency, and (v1.2) sumErr, in both per-base and per-kmer formats.
- Mismatches/deletions with reduced quality are systematically concentrated at known modified sites and are absent in ime4-knockout (m6A-lacking) control strains.
- Downstream SVM trained on these features reaches ~90% accuracy on synthetic modified/unmodified sequences and 87% accuracy in vivo (yeast).
- Each candidate site's features are aggregated across all reads mapping to it (no per-read calls emitted).
- Control-vs-treated pairing (KO/KD or demethylase) is used to filter high-confidence sites by largest error-feature differences.

## Limitations

- Direct RNA base-calling is error-rich even for unmodified RNA, so a matched control (KO/KD or demethylase-treated) is strongly recommended.
- The algorithm does not predict modifications in individual molecules and does not distinguish modification types (e.g., m1A vs m6A).
- EpiNano-SVM predictions are base-caller dependent — only accurate if data are base-called with the version matching the pre-trained model (Albacore 2.1.7 for v1.0/1.1; Guppy 3.1.5 for v1.2).
- Feature contributions vary across k-mers, so k-mer-specific training is needed to reduce false positives; overall recovery of known sites is only ~32%.

## Evidence

- [discussion] error features from base-called reads: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [other] site-level aggregation: "our current algorithm does not predict RNA modifications in individual RNA molecules, but rather employs information from all the reads mapping to a specific site"
- [discussion] no squiggle manipulation: "here we provide a novel strategy to identify RNA modifications from base-called features, without the need of squiggling realignments or manipulation of raw current intensity data sets."
- [readme] feature list: [evidence span withheld — anchor status: external_doc]
- [readme] base-caller dependence: [evidence span withheld — anchor status: external_doc]
- [readme] control recommendation: [evidence span withheld — anchor status: external_doc]

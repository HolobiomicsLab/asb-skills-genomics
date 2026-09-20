---
name: cigar-operation-parsing-for-indel-frequencies
description: Use when starting from a sorted, indexed BAM of ONT direct RNA reads
  aligned to a transcriptome and you need per-position base-calling error features
  (mismatch, insertion, deletion, quality) to feed EpiNano SVM classification for
  m6A detection.
when_to_use_negative:
- When you already have an EpiNano per-site or per-kmer feature table — go straight
  to SVM classification instead of re-parsing CIGARs
- When the question is modification detection from raw current-intensity squiggle
  data — EpiNano's feature pipeline intentionally avoids squiggle realignment or manipulation
  of raw current intensity data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3511
- http://edamontology.org/topic_3295
tools:
- name: EpiNano
  role: Consumes the per-site/per-kmer error and quality features for SVM-based m6A
    prediction
  repo: https://github.com/novoalab/EpiNano
- name: Oxford Nanopore Technologies (ONT)
  role: Source of the direct RNA sequencing reads whose base-called error features
    are extracted
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-11713-9
    title: Accurate detection of m6A RNA modifications in native RNA sequences
  source_content_hash: sha256:bedc967a7909752db2a15954bb97d234d6cf01c9a65e4b5a256256b53d310a25
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/novoalab/EpiNano
  tools:
  - EpiNano
  - Oxford Nanopore Technologies (ONT)
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3511
  - http://edamontology.org/topic_3295
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# CIGAR operation parsing for per-site mismatch/insertion/deletion frequencies

## Summary

Walks the CIGAR strings of ONT direct RNA reads aligned to a reference transcriptome (BAM via samtools/pysam) and aggregates per-base records into a per-site feature table of mismatch, insertion, and deletion frequencies plus mean per-base quality, as required for EpiNano m6A classification.

## When to use

Use when starting from a sorted, indexed BAM of ONT direct RNA reads aligned to a transcriptome and you need per-position base-calling error features (mismatch, insertion, deletion, quality) to feed EpiNano SVM classification for m6A detection.

## When NOT to use

- When you already have an EpiNano per-site or per-kmer feature table — go straight to SVM classification instead of re-parsing CIGARs
- When the question is modification detection from raw current-intensity squiggle data — EpiNano's feature pipeline intentionally avoids squiggle realignment or manipulation of raw current intensity data

## Inputs

- Sorted, indexed BAM file of ONT direct RNA reads aligned to the reference transcriptome
- Reference transcriptome used for alignment

## Outputs

- epinano_features_per_site.tsv — one row per transcriptomic position with coverage depth, mean per-base qualities (A/C/G/T), and mismatch/insertion/deletion frequencies
- Per-base and per-kmer EpiNano feature tables ready for SVM classification

## How to apply

Load the sorted, indexed BAM with samtools/pysam and, for each aligned read, walk the CIGAR string sam2tsv-style to emit one record per reference position carrying the read base identity, base-calling quality, and match/mismatch/insertion/deletion status. Aggregate these records at each reference position across all reads mapping to that site, computing the mean per-base quality for each of the four bases (A, C, G, T) and the mismatch, insertion, and deletion frequencies. Restrict the table to positions with sufficient read coverage so the per-site error statistics are estimable, and retain the per-site depth alongside the error features. The rationale is that m6A manifests in direct RNA data as systematic errors — altered per-base qualities and mismatch/deletion frequencies — at the modified site, and these aggregated features are the inputs EpiNano's SVM uses to reach ~90% prediction accuracy. Features are organized in per-base and per-kmer formats over the 5-mer window (−2, −1, 0, +1, +2) centered on the candidate site.

## Related tools

- **EpiNano** (Consumes the per-site/per-kmer error and quality features for SVM-based m6A prediction) — https://github.com/novoalab/EpiNano
- **Oxford Nanopore Technologies (ONT)** (Source of the direct RNA sequencing reads whose base-called error features are extracted)

## Examples

```
python -c "import pysam; a=pysam.AlignmentFile('aln.sorted.bam','rb'); [print(r.reference_id, r.reference_start, r.cigarstring) for r in a.fetch()]"
```

## Evaluation signals

- The output TSV contains per-site mean per-base quality columns (A, C, G, T) and mismatch/insertion/deletion frequency columns for every covered position
- Per-site values are aggregated across reads and include coverage depth, consistent with site-level (not per-read) features
- Downstream EpiNano SVM trained on these features attains the reported ~90% in vitro accuracy (87% in vivo on yeast)
- Insertion frequency is present as a feature column even though only mismatch and deletion frequencies are the discriminative signals reported

## Limitations

- Only positions with sufficient read coverage yield estimable per-site error statistics; low-coverage sites must be dropped
- The feature extraction is base-caller-dependent for pre-trained models: EpiNano 1.2 SVM models require Guppy 3.1.5 base-called data; Albacore 2.1.7 for versions 1.0/1.1
- Direct RNA base-calling is error-rich even on unmodified sequences, so paired WT/KO (or demethylase-treated) control data is strongly recommended to distinguish modification signals from background errors

## Evidence

- [abstract] Altered qualities and deletion rates at modified sites: "using direct RNA sequencing, N 6 -methyladenosine (m 6 A) RNA modifications can be detected with high accuracy, in the form of systematic errors and decreased base-calling qualities"
- [discussion] Error types used as features: "These “errors” can be detected in the form of altered per-base qualities, mismatch frequencies and deletion frequencies at the modified site."
- [readme] Feature set extracted from direct RNA reads: [evidence span withheld — anchor status: not_found]
- [readme] Per-base and per-kmer organization: [evidence span withheld — anchor status: external_doc]
